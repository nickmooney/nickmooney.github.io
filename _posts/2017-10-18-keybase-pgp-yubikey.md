---
layout: post
title: Keybase + PGP + YubiKey
comments: true
---

I wanted to document the way I most recently generated PGP keys to live on my YubiKey. My previous PGP keys expired and my current YubiKey is affected by the [Infineon RSA bug](https://crocs.fi.muni.cz/public/papers/rsa_ccs17), so my goal was to have keys generated on my laptop that live entirely on my YubiKey and that play nicely with Keybase.

This is not the most secure way to do this, but it works adequately for my purposes (in which I am more likely to be using PGP to enable YubiKey-based SSH login, not communicating national secrets). Ideally you would generate a master key on an airgapped computer and generate your subkeys on the YubiKey itself. The fact that your secret key material will be on your hard drive temporarily may not be acceptable depending on your threat model.

## Generate your master key
Keybase has sane defaults (RSA 4096) and makes the PGP key generation process easy, so I just ran a `keybase pgp gen` and entered my information. I chose not to upload my private key to Keybase.

This generates a master key with Signing and Authentication capabilities, as well as a subkey with Encryption capabilities. To offload your keys to your YubiKey, you'll likely want subkeys for encryption, signing, and authentication, so you're already 1/3 of the way there with the Keybase defaults.

## Generate subkeys
Run a `gpg --list-keys` to get your key fingerprint, then run `gpg --expert --edit-key <yourfingerprint>`.

You'll need to issue two `addkey` commands. Generate one subkey as "RSA (sign only)", and another with only authentication privileges using the "RSA (set your own capabilities)" option. My session looked like the following. Note that I used `16y` as the key expiry parameter -- you can use whatever expiry timing you want. I just stuck to Keybase' default of 16 years because I'm likely going to be "revoking" my PGP keys by removing them from my Keybase account rather than by issuing proper GPG revocations. In other words, in the worst case, a key is compromised and doesn't expire for a long time, but my Keybase account shows that I no have asserted that I no longer control that key.

{% gist c207b7845517b95cddd1025c023c445a %}

## Move your subkeys to your YubiKey
In the same key editing session, we're going to select each key and move it to the YubiKey.

1. Select the first subkey with `key 1`
2. `keytocard` to move the key to the YubiKey
3. Deselect the subkey with `key 1` again (as you can only select one subkey at a time when issuing a `keytocard`)
4. Repeat the process with `key 2` and `key 3`

Hooray! You've moved your subkeys to the YubiKey.

## Back up your master secret key
Issue a `gpg -a --export-secret-key <yourkeyid> > backup_secret.asc`. Ideally `backup_secret.asc` lives on a thumb drive you keep in a hole in your back yard or something.

Now delete your secret key from your computer, as you don't need it any more -- the keys you'll be using live on the YubiKey. You can do this with `gpg --delete-secret-key <yourkeyid>`.

## Update Keybase with your new subkeys
Run a `keybase pgp update` to push your updated public key to Keybase. Your master key hasn't changed, but your public key now reflects that you've authorized two additional subkeys.

## You're done!
That's it! Note that you will still be able to run `gpg --export-secret-key` and get some output. This confused me for a little while until I learned that only secret key _stubs_ are being exported. You can inspect the output of the export command by running `gpg -a --export-secret-key <yourkeyid> | gpg --list-packets --verbose`. You should see "pkey" entries, but no "skey" entries.

From now on, when you attempt to perform an operation that requires the use of your PGP keys, you should be prompted to insert the YubiKey and then enter the PIN.
