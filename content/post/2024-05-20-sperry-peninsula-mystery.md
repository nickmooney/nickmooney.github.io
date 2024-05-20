---
date: "2024-05-20T00:00:00Z"
title: The Sperry Peninsula Mystery
slug: sperry-peninsula-mystery
---

I tried and failed to track down the owner of a Lopez Island property.

### Context

The Sperry Peninsula (1469 Sperry Rd) is a 387-acre property on Lopez Island that the Paul G. Allen Trust owned from 1996 to 2022. Prior to 1996 the property was occupied by Camp Nor’wester, a summer camp for children that now operates from nearby Johns Island. Although I have no personal relationship with the Sperry property, I do have a relationship with Nor’wester, so when the Sperry property changed hands in 2022 I became curious who the new owners were.

This is a post about how I have tried and failed to determine who purchased the Sperry Peninsula property - or framed another way, a story of reverse engineering the process by which someone might anonymously purchase property in Washington State. Read on if you’re curious about the mechanisms by which wealthy people can hide their identities when purchasing property. I didn’t know any of this when I started trying to figure out who bought the property, but I had a fun time reverse engineering it.

If you have further suggestions about how I might figure out who bought this property, please get in touch. There’s so much I don’t know about public records sleuthing.

### Public Records

San Juan County maintains an accessible repository of property ownership information. Via the map interface or by looking up the address of the property, you can find the [record for property 1002](https://parcel.sanjuancountywa.gov/PropertyAccess/Property.aspx?cid=0&year=2023&prop_id=1002) with further links to associated documents. The Sperry Peninsula actually comprises 12 properties, but 1002 seems to be the main one, and the IDs of the other can be found in the deed of sale.

The deed of sale is San Juan County [document 2022-0506015](https://apps2.sanjuanco.com/Auditor/DigitalResearchRoom/Document/Details?year=2022&document=506015), which indicates that the property was sold by the Paul G. Allen Trust to an entity called Lopez Peninsula LLC, “a Delaware limited liability company.”

Delaware has a unique reputation in the world of anonymous shell companies. We’ll take a detour to explain why.

An LLC is a corporate entity made of “members” (the people who own the LLC) and an operating agreement. If you’d like to form an LLC in the state of Delaware you can fill out [this form](https://corpfiles.delaware.gov/LLC_Forms/LLC%20Formation.pdf). The only information required is the name of the desired LLC and contact information for a “registered agent” of the company (which must be in Delaware). A registered agent receives legal process on behalf of the LLC, so must have a valid Delaware address, but there’s no requirement that the registered agent have any specific relationship to the member(s) of the LLC.

As a result of the above, Delaware doesn’t know and doesn’t care who the actual members of the LLC are. Delaware further [does not require](https://corp.delaware.gov/alt-entitytaxinstructions/) annual reports from LLCs – as long as someone pays the $300/year tax on behalf of the company, Delaware is happy. Compare that to Washington state, where the [executor’s identity must be disclosed](https://www.sos.wa.gov/sites/default/files/2023-10/LLC_COF.pdf) at the time of formation of the LLC, and annual reports must be filed.

If we look up Lopez Peninsula LLC on the [Delaware DOC website](https://icis.corp.delaware.gov/eCorp/EntitySearch/NameSearch.aspx), we can find the name of its registered agent and little else. This doesn’t help us, as the registered agent is a business that exists for the purpose of being registered agent of other businesses.

Per [RCW 23.95.520](https://app.leg.wa.gov/RCW/default.aspx?cite=23.95.520), a foreign LLC may own property in Washington state without having to register[^1] with the Washington Secretary of State.

### Hitting a Dead End

So the new owner is an LLC, but someone still has to sign the documents.

Looking through the documents made available by San Juan County, the name “Jenifer Jewkes” shows up constantly. Jenifer is a lawyer with Lane Powell PC; she specializes in estate planning for high net worth individuals. In public documents she’s the face of Lopez Peninsula LLC, but this doesn’t get us any closer to determining who really bought the property.

There is a single document I could find that references the names of the members of the LLC: a [notice of continuance](https://apps.sanjuancountywa.gov/Auditor/DigitalResearchRoom/Document/Details?year=2022&document=506016) of Open Space classification (essentially an affirmation that the new owners will continue to leave certain parts of the property undeveloped). Attached to this notice is an “affidavit of manager” - this document asserts that Jenifer Jewkes is legally permitted to act on behalf of Lopez Peninsula LLC. This document **contains the names of the LLC members**, but the copy recorded with the San Juan County auditor’s office is redacted. I gave them a quick call several months ago and they indicated that the way it appears online is the way it was recorded.

I’m not sure whether this document is part of the public record anywhere else, but I wasn’t able to find an unredacted copy.

### Miscellanea

The mailing address that San Juan County has on file for the property owner leads us to another corporation, Chinook Management LLC, but we can trace this back to the same Jenifer Jewkes of the Lane Powell PC law firm. There is a link to another Paul Allen property, but the connection here has nothing to do with the new owner of the property, other than indicating that Jewkes facilitated the legal structure for the transaction.

[^1]: Although the purchase of the Sperry property was completed in 2022, and Lopez Peninsula LLC is a Delaware company, Lopez Peninsula LLC was actually registered with the WA SOS as a foreign LLC for a period of just 16 days in 2021 under its former name, Sperry Holdings LLC. I’m not sure why it was registered and dissolved so quick – maybe some early part of the purchase process required a Washington corporate entity. Its Washington UBI number was 604 795 975, and you can find the name change history at the Delaware Division of Corporations website. There is no record of the current name, Lopez Peninsula LLC, with the WA SOS. The name change from “Sperry Holdings LLC” to “Lopez Peninsula LLC” was filed with Delaware on September 16, 2021, the same day the LLC officially withdrew from the Washington SOS.