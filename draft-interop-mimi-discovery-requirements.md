---
title: "MIMI Discovery Requirements"
abbrev: "Discovery Requirements"
category: info

docname: draft-interop-mimi-discovery-requirements-latest
submissiontype: IETF
number:
date:
consensus: false
v: 3
area: ART
workgroup: "More Instant Messaging Interoperability (mimi)"
keyword:
 - user discovery
venue:
  group: mimi
  type: Working Group
  mail: mimi@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/mimi/
  github: femigolu/user-discovery
  latest: https://datatracker.ietf.org/doc/draft-interop-mimi-discovery-requirements/

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    fullname: Giles Hogben
    email: gih@google.com
    organization: Google
    country: United States of America
 -
    fullname: Femi Olumofin
    email: fgolu@google.com
    organization: Google
    country: United States of America
 -
    fullname: Jon Peterson
    email: jon.peterson@transunion.com
    organization: TransUnion
    country: United States of America
 -
    fullname: Jonathan Rosenberg
    email: jdrosen@jdrosen.net
    organization: Five9
    country: United States of America

normative:
  RFC2119:

informative:


--- abstract

This document defines requirements for the discovery problem within the More Instant Messaging Interoperability (MIMI) working group. Discovery is essential for interoperability, allowing message senders to locate recipients across diverse platforms using globally unique, cross-service identifiers (e.g., email addresses, phone numbers). The core challenge involves reliably mapping these identifiers to messaging service providers and determining the reachability of a recipient's identifier across multiple providers.


--- middle

# Terminology

{::boilerplate bcp14-tagged}

Glossary of terms:

1. A **Service Specific Identifier (SSI)** is a unique identifier for a user within the context of a single messaging service provider. It is not globally unique across services. A fully-qualified SSI includes the messaging service provider's unique identifier, making it globally unique. Note that existence of an unqualified SSI on two or more services does not guarantee that the associated accounts belong to the same user. Thus, linking SSIs across services requires other verified data to establish a match. An example of a service specific identifier is a user's X/Twitter handle.

1. A **Cross-Service Identifier (CSI)** is a globally unique identifier for a user that can be used to identify the user across multiple services. For example, a user's E.164 phone number, email address, or other service independent identifiers are CSIs, since they can be used to identify the user across multiple different services.

1. A **Messaging Service Provider (MSP)** provides messaging, voice, video and other forms of real-time communications services to end users. Examples of messaging service providers are WhatsApp, Messages, iMessage, Wire, Matrix, Signal etc. A user is reachable using one or more CSIs and may have at least one SSI internal to the MSP platform.

1. A **Cross-Service Identifier Provider (CSIP)** is an entity that issues, manages, and verifies CSIs. Examples are traditional telecom providers, email providers, and other platforms capable of issuing user-controlled identifiers.

1. A **Discovery Provider (DP)** is an entity that facilitates the discovery reachability as outlined in this requirements document. This involves creating, managing, and leveraging authoritative mappings of CSIs to the MSPs where users can be reached. We expect that a DP may be operated by or affiliated with an MSP or a third party.

# Introduction

MIMI discovery seeks to enable users to easily connect and communicate across different messaging platforms, even if they don't know which specific platform the other person uses. Currently, users often need to ask contacts what service they are on out-of-band, or try multiple services, which creates friction. Specifically, discovery helps a user on one messaging service (Alice on Service A) to find another user on a potentially different or same service (Bob on Service B) in a provider-neutral manner.

Discovery is necessary because the identifiers we commonly have for contacts (phone numbers, email addresses, etc.) do not necessarily tell us which messaging service they are using. Someone with the email alice@gmail.com might use iMessage, Signal, or another service entirely. Thus, the core problem is how to take one of these cross-service identifiers and learn the messaging service provider that the user is using, and how to communicate with them on that service.

## Problem Statement

The discovery problem involves:

1. Asserting verifiable mappings between CSIs and MSPs, and

1. Looking up mappings to determine MSPs for which a CSI can be reached for messaging.

Discovered mappings must be verifiable to ensure they are accurate. Crucially, discovery must prioritize user privacy, allowing users to control their discoverability, and it must integrate well with end-to-end encryption and other MIMI protocols.

Note that mapping lookup is distinct from the retrieval of the user's SSI and cryptographic keys. Retrieval of the SSI and keys typically occurs as part of the messaging process and can easily be done after the MSP for the message recipient has been located through discovery. For example, an MSP might store a mapping of CSIs to SSIs and cryptographic keys. The client would then retrieve this data just before Alice sends a message to Bob, following the successful discovery that Bob is reachable through the MSP.

The rest of this document describes a series of requirements for the discovery problem.

# Prior Efforts

Discovery services are far from new on the Internet. The whois protocol {{!RFC3912}}, largely focused on mapping domain names to associated services. It was one of the earliest discovery services deployed on the Internet. DNS SRV records, specified in {{!RFC2782}}, allow a similar process - given a domain name, a user can discover available services, such as VoIP based on the Session Initiation Protocol (SIP) {{!RFC3261}} {{!RFC3263}}. SRV records were adapted specifically for messaging in {{!RFC3861}}. However, both whois and DNS SRV records rely on domain names as lookup keys, making them unsuitable for identifiers like mobile phone numbers, which don't have inherent domain associations.

ENUM {{!RFC6117}} addressed this limitation. It used DNS to lookup phone numbers by reversing the digits and adding the "e164.arpa" suffix. This allowed delegation of portions of the namespace to telco providers who owned specific number prefixes. While technically straightforward, public deployment of ENUM was hampered by challenges in establishing authority for prefixes. However, private ENUM {{!RFC6116}} services have become relatively common, facilitating functions like MMS routing within messaging.

Another attempt was ViPR (Verification Involving PSTN Reachability) {{!I-D.rosenberg-dispatch-vipr-overview}} {{!I-D.petithuguenin-vipr-pvp}}. ViPR utilized a peer-to-peer network based on RELOAD (Resource Location and Discovery) {{!RFC6940}} to operate between enterprises. It addressed the authority problem by authorizing records based on proof of forward routability but faced the same network effects issue as ENUM. ViPR attempted to address incentive problems by focusing on enterprises seeking cost savings by bypassing the phone network. Ultimately, network effects challenges (among other protocol-unrelated issues) prevented widespread deployment.

Discovery and lookup services are now commonplace on the Internet but are largely scoped within large providers such as Facebook, Twitter, WhatsApp, and others.

The MIMI discovery service requires a solution that spans providers.

# Architectural Models

To ensure completeness and to address implementation considerations for MIMI DP, we present several potential architectural models. The working group observed these requirements are similar among these models and opted to maintain architectural neutrality for the discovery protocol. However, we will outline requirements for the roles of DPs, how they interact with each other, MSPs in a federated model, and how DPs accommodate queries from both MSPs and users.

## Centralized DP (Monolithic Service)

A globally accessible, authoritative database (potentially sharded/replicated) stores all authenticated CSI mappings. This monolithic service, implemented across synchronized global nodes, handles all CSI queries from messaging platforms and acts as the single source of truth for mapping data, even if certain mappings may be restricted in specific regions due to geopolitical considerations.

### Advantages
- Standardization and uniform control over mapping creation, updates, and data formats.
- Promotes fair and unbiased CSI mapping discovery.
- Single source of truth for mapping simplifies data management and ensures consistency.
- Sharding/replication can address geographical distribution and performance needs.

### Drawbacks
- Centralization of sensitive user data raises privacy risks.
- May conflict with data localization regulations.
- Single point of failure vulnerability from outages affecting the entire system.
- Potential difficulty with immediate global updates for rapidly changing mappings.

## Hierarchical DPs (Discovery Resolvers)

A global root Discovery Provider (DP) directs mapping requests to authoritative DPs based on CSI structure (e.g., country codes for E.164 phone numbers) or sharding mechanisms. The root DP, similar to hierarchical DNS, acts as a directory service, holding pointers to authoritative DPs rather than mappings themselves. Alternatives to hierarchical resolution, like the LoST protocol {{!RFC5222}}, or distributed hash tables (DHTs), can achieve similar outcomes.

### Advantages
- Scalability from distributed load and mapping management across multiple DPs.
- Flexibility that allows different DPs to specialize in specific CSI ranges for regions.
- Better alignment with data localization requirements.

### Drawbacks
- Requires coordination and maintenance of hierarchy.
- Root DP failure could disrupt the entire system.
- Potential delays due to additional hops in the discovery process.

## Federated DPs (Distributed Peer Service)

In a federated model, multiple independent DPs collaborate to provide CSI mapping discovery. Each DP holds a subset of mappings and pointers to other DPs, with no central authority dictating mapping locations. Discovering all mappings for a CSI may involve querying multiple DPs. DPs can exchange CSI information or recursively query each other. The specifics of DP federation are determined by business agreements, not technical requirements. A variation of this model involves messaging platforms acting as their own DPs, managing mappings for their users.

### Advantages
- Decentralization ensures there is no single point of control or failure. The system can continue functioning even if some DPs are unavailable.
- Mappings can be distributed according to local regulations.

### Drawbacks
- Discovery process may involve querying multiple DPs, increasing network load.
- Potential for bias as DPs may prioritize their own mappings, leading to uneven results.
- Requires robust mechanisms to prevent CSI impersonation and ensure trust between DPs.
- Pairwise relationships in a federated DP model could create a barrier to entry for smaller MSP/DPs, similar to the trust requirements in the DKIM {{!RFC6376}} and DMARC {{!RFC8616}} ecosystems.

## Additional Considerations on Architectural Models

### Telephone Number CSIs
Telephone number portability is complex due to its reliance on real-time queries to proprietary and legacy systems. Overall, the authenticated mappings proposed for MIMI may necessitate additional platform measures to assess mapping freshness and ensure up-to-date reachability responses.

### Bias Mitigation
Bias occurs when a DP prioritizes mappings to its affiliated MSP without consideration of what is best for end users. Mitigating bias is essential to ensure fair and equitable discovery of authenticated mappings across different services. The working group has decided to defer such mitigation to policies and regulations, excluding it from the discovery protocol.


# Summary of Requirements

| # | Requirement | Mandatory | Optional |
|---|-------------|-----------|----------|
|   | **Authenticating Mappings** | | |
| 1 | DP MUST verify user's CSI possession through proof-of-possession challenges through a CSIP, certificate authority or designated parties | x | |
| 2 | MSP MUST confirm CSI reachability on its service | x | |
| 3 | Client, MSP, and DP must collaborate to generate a verifiable representation of the CSI-to-MSP mapping. This can then be shared with any DP and verifiable by clients | x | |
| 4 | DP MUST NOT be able to create a verifiable mapping without CSI holder and MSP involvement | x | |
| 5 | DP MUST NOT be able to falsely claim user completed proof-of-possession | x | |
| 6 | Other users MUST be able to verify CSI holder's participation in mapping creation | x | |
| 7 | Authenticated mappings MUST include a preference tag to enable recipients to control their preferred contact mapping | x | |
|   | **Discovery Protocol** | | |
| 8 | Discovery MUST support any globally unique CSI with backing source of truth (CISP for telephone), ownership proof, and cross-service usability | x | |
| 9 | DP MUST protect at least the querier's identity or the target CSI in requests | x | |
| 10 | Discovery requests MUST support federation, MSP filter, and DP list query parameters | x | |
| 11 | DP MUST disclose default behavior and comply with agreed-upon federation default | x | |
| 12 | DP MAY rate-limit non-default queries given their higher processing costs | | x |
| 13 | DP MAY rate-limit requests sent to low-throughput DP endpoints | | x |
| 14 | Discovery MUST accommodate zero, one, or multiple MSPs in results | x | |
| 15 | The protocol MUST define both verbose and compact response formats, where verbose responses include detailed mapping information and metadata, while compact responses provide a simple indication of CSI reachability on returned MSPs | | |
|    | **Operational** | | |
| 16 | Discovery service MUST remove mappings made outdated by CSI re-assignment to a new user within reasonable time | x | |
| 17 | Older mappings generally take precedence over newer ones for the same CSI unless explicitly invalidated by the original CSI holder or superseded by a stricter proof of possession verification | x | |
| 18 | DP MUST verify if a mapping is the first mapping for a given CSI and, if so, broadcast invalidation requests to other DPs to invalidate any existing mappings for that CSI | x | |
| 19 | Users SHOULD be provided with mechanisms to invalidate existing mappings or create a replacement mapping for their CSIs | | x |
| 20 | New CSI mappings SHOULD be discoverable within some standardized maximum time limit (e.g., 24 hours) | | x |
|    | **Security** | | |
| 21 | Discovery service MUST leverage contractual and technical means prevent malicious MSPs from falsely claiming CSI association | x | |
| 22 | Discovery service MUST incorporate anti-DDoS, anti-enumeration and anti-spam mechanisms | x | |
| 23 | All communication between clients, DPs, and MSPs MUST be encrypted in transit and authenticated | x | |


# Authenticating Mappings Requirements

A Discovery Provider aggregates mappings between CSIs and platforms from trusted sources. To prevent impersonation attacks where platforms or users simply claim to own a CSI, the user must solve a proof of possession challenge for the CSI before a DP can establish reachability mapping involving the CSI. Suitable approaches include SMS one-time-code or voice call verification to prove possession of phone number, email link verification for emails, OAuth sign-in for Twitter and YouTube identifiers. A potential architecture for generating credentials from proof-of-possession checks is shown in {{!I-D.draft-peterson-mimi-idprover}}. A platform performing discovery should be able to verify that the target user established the reachability mapping. Note that once a mapping is created, it can be distributed by any DP, and it should include metadata for clients that receive it to verify its authenticity.

## Functional Requirements

At least, a user's client, MSP, and DP MUST participate in a consensus protocol to establish a CSI-to-MSP mapping with the following requirements:

1. The DP MUST verify the user's possession of the CSI through some proof-of-possession challenge.
1. The MSP MUST confirm that the CSI is reachable on its service.
1. The client, MSP, and DP must collaborate to generate a verifiable representation of the CSI-to-MSP mapping (e.g., using a threshold signature). The can then be shared with any DP and verifiable by clients.

## Privacy and Security Requirements

1. A DP MUST NOT be able to create a verifiable mapping without the involvement of the user holding the CSI and MSP.
2. The DP MUST NOT be able to falsely claim that a user completed the proof-of-possession challenge.
3. Other users MUST be able to verify that the CSI holder (and not an imposter) participated in creating the mapping.


# Preferences

The discovery process involves the preferences of multiple stakeholders:

1. the querier seeking reachability information,
2. the user with the mapped identity, and
3. DPs (and by extension, the collaborating MSPs).

The authors suggest the following:

- Implementations shouldn't dictate a one-size-fits-all approach for expressing and meeting these preferences, but should rather implement basic building blocks for each of these parties to express their preferences.
- DPs should clearly communicate their preference handling practices to promote transparency and trust.
- The discovery requirements consider detailed preferences and capabilities out of scope, leaving them to individual implementations.
- Given that the sender initiates discovery requests and already has options on which app, MSP, and DP to query, we will only provide a basic recipient preference specification as a requirement below.

## Basic Recipient's Preference Requirement

Requirement: Authenticated mappings MUST include a preference tag to enable recipients to control their preferred contact mapping.

The preference tag can be either a string (e.g., "Business", "Personal", "BasketballFriends", "WhatsApp") or a list of strings (e.g., "Business, WhatsApp"). For example, a recipient might want senders from WhatsApp to utilize a specific non-default "WhatsApp" mapping.

1. The default mapping must be designated as "Default" and cannot be a list.
2. Non-default mappings can have one or more tags to signify the recipient's intended purpose for that contact mapping.
3. The total length of tags should be limited within the protocol.
4. If a CSI's set of mappings lacks a default mapping or multiple mappings have the "Default" tag, the recipient can choose any of the mappings for communication.
5. Tie-breaking should occur only once and be re-evaluated solely through explicit user action. This prevents messages from a sender from being scattered across multiple mappings for the recipient.

## Recipient's Critical User Journeys (implementations)

Here are some Critical User Journeys (CUJs) that are the most important to discovery recipients.

In the CUJs below, Bob is the recipient, and Alice is the sender or user performing discovery:

1. Sender mapping preferences: Bob only wants to be found by Alice and other users on WhatsApp, not his other messaging apps.
2. Same-app preferences: Bob prefers that Alice can find him on the same messaging service that she is using. In other words, Bob does not want cross-app discovery and messaging.
3. No-random mapping preferences: Bob does not want to go through multiple apps to find a message from Alice when discovery returns one of 10 mappings that Bob has established with discovery providers.
4. No-duplication preferences: Bob does not want Alice's messages to be broadcasted to all or a subset of his apps based on the result of discovery.
5. Per-sender preferences: Bob wants to control which app messages from Alice go to and do the same for other users (e.g., Carol's messages may go to a different app than Alice's).
6. Closed group preferences: Bob only wants his soccer parents to discover and contact him on WhatsApp, not his Wire app. That is, a group of senders has the same mapping results based on Bob's preferences.
7. Open-ended group preferences: Bob wants his business contacts to discover and reach him on Wire, not WhatsApp. That is, an open-ended list of senders (i.e., including leads) are provided with a designated mapping.

# Discovery Protocol Requirements

## Identifier Types

Discovery MUST support any globally unique cross-service identifier with the following characteristics:

1. Backing source of truth: Authoritative issuing entities exist for issuing the CSI to users (e.g., CSIP for telephone numbers).
2. Ownership proof mechanism: The user issued a CSI must be able to demonstrate or pass a proof of possession challenge from a remote prover.
3. Versatility: CSIs must be deployable and usable across multiple services

Phone numbers and email addresses are examples of suitable and supported identifiers.

## Discovery Response Requirements

### Cardinalities
The discovery protocol must accommodate scenarios with varying numbers of MSPs in the discovery results:

1. Zero MSPs: The system should indicate a no-match condition if users and their associated Identifiers (e.g., CSIs) are not discoverable on any MSP. This enables the originating user to recognize that the CSI is not reachable via the discovery system.
2. One MSP: The system should function efficiently when a CSI is associated with a single MSP.
3. Multiple MSPs: The system should accommodate users with multiple MSPs.

### Response format
An MSP or client app may request responses that are verbose or compact. A verbose response may include all unique lists of mappings discovered with metadata for the client to verify each mapping, and metadata about the list or count of DPs where that mapping was found. A compact response may be as simple as a bit string with each set bit representing that the CSI is found in the MSP assigned to that bit position. The protocol MUST define specific formats for both response types.

## Discovery Request Requirement

### Requests
Discovery requests must include the CSI and may include additional query parameters to guide the search process. The query parameters below MUST be supported.

#### Federation
Indicate if the DP should answer queries using its own database or federate the request to other DPs and aggregate their answers in a fair and transparent manner. In a federated model, a DP that chooses not to federate may be limited in the queries it can answer. Certainly, a DP can incorporate mappings that either reference another DP's mapping or materialize those mappings into its own local database.

#### MSP filter
Useful for scoping the response of interest to one or more MSPs (e.g., query for CSI mapping to WhatsApp only).

#### DP list
A list of DPs may accompany each query to guide a DP on query federation decisions. These sub-options for DP list must be supported:

- DP-preferred federation: signals for the DP to forward the query to its preferred subset of DPs.
- Client-selected federation: instructs the DP to forward the query to a specific list of DPs provided by the client.
- All DPs: mandates the DP to forward the query to all DPs within the ecosystem, utilizing a publicly accessible registry (described below).

### Default behavior disclosure
A DP must externally disclose its default behavior which should be consistent with local regulatory requirements (e.g., do not federate by default for performance, privacy or regulatory reasons). For example, in the absence of a context parameter, the DP disclosed it will utilize its local database to fulfill requests.

A DP must follow some preferred default behavior that is agreed at the federation level.

### Client rate limit
Again, forking discovery requests consume ecosystem resources, and could facilitate attacks. Hence a DP may rate-limit non-default queries. Specifically, a client may be limited to a few queries daily when federated to the entire ecosystem DPs.

### Server rate limit
DPs may optionally rate-limit requests directed at DP endpoints with low query processing throughput for discovery responsiveness.

### Rate-limiting context
A DP must provide sufficient context to the federation of DPs on a fork tree to assist them in rate-limiting requests from specific clients in a privacy-friendly manner.

## Privacy Requirements

### Requirement
A DP must protect at least one end of the social graph during a request. In other words, the DP must protect either the querier's identity or the CSI of interest in requests.

Possible implementation approaches when Alice is discovering Bob's reachability:

1. Querier identity protection: IP blinding (e.g., Private Relay) can help to conceal Alice's identity or IP address so the DP may learn the query target only.
2. Query content protection: Techniques like Private Information Retrieval (PIR) or Private Set Membership (PSM) can conceal the target CSI, so the DP may learn the querier only.

The messaging social graph of a user shows all the other users that the user has communicated with over time.The discovery social graph of a user shows all the users for whom discovery has been attempted. The discovery social graph is significantly larger than the messaging social graph, even though there may be some overlap between the two. To clarify, a user's address book defines their potential social network. MIMI discovery, when applied comprehensively using the address book, reveals that network on various services. In contrast, the messaging social graph only includes contacts actively messaged within a specific period, which is inherently a subset of the user's wider contact network. Since discovery can query for users the initiator hasn't yet messaged, the discovery social graph is naturally larger.

# Operational Requirements
The discovery service must support a decentralized architecture with multiple DPs, enabling federation based on user preferences, geopolitical boundaries, and DP-specific policies.

Some additional considerations:

1. Federation mechanisms: Protocols or standards for DPs to communicate and exchange mapping data must be defined. This includes how requests are routed and how DPs locate potentially relevant mappings stored elsewhere.
2. Data sovereignty: Regulations such as GDPR have a direct impact on where user data can reside. Solutions must be designed to respect data locality and follow jurisdictional laws.

## Registry

It is likely that reachability mappings on some services will not be shared publicly for privacy reasons. Thus DPs may federate: for example, a discovery client may send a request to one DP, which will then need to consult a second DP in order to complete the request. Some sort of policies will need to govern the relationships between DPs and the terms under which they federate. Such a policy is outside MIMI's purview.

Metadata and service configurations about federation membership of interoperable DPs may be hosted on a registry managed by the federation. Each DP's record may include a unique identifier, discovery endpoints, and other configuration metadata.

## CSI Release Timeliness

The discovery service must strive to remove outdated mappings (resulting from users ending service with a CSIP) within a reasonable timeframe, but this timeframe must acknowledge the limitations of existing legacy systems and the potential for identifier reuse. For example, in scenarios where a number is disconnected and later reassigned, the new user must designate the very first mapping as such. This triggers the DP to broadcast invalidation requests to other DPs, effectively clearing any lingering mappings associated with the reassigned number.

### Mapping Prioritization Requirement

Whenever a new mapping is attempted for an existing CSI, the one established earlier should generally take precedence. This precedence SHOULD be overridden ONLY in the following cases:

1. The original CSI holder explicitly signals the mapping is no longer valid.
2. A new user of the CSI establishes a mapping and successfully completes a stricter proof of possession verification process.

### Conflict Resolution Protocol Requirement

1. DPs MUST implement a conflict resolution protocol when a new mapping creation attempt is made for a CSI that already has mappings within the DP's service.
2. This protocol SHOULD include a privacy-preserving notification to the holder of any existing mappings (without revealing the new user's identity).
3. If the conflict cannot be automatically resolved, the DP MAY escalate to a manual review process that involves additional verification steps for the new user.

### Invalidate Capability Requirement

1. Mechanisms SHOULD be provided to enable users to signal that a CSI they previously used is no longer under their control. This could include:
   - Collaborations with CSIP to receive re-assignment notifications.
   - User-initiated invalidation requests.
2. Invalidate signals SHOULD trigger updates to discovery mappings to minimize conflicts.

## CSI Claim Timeliness

Upon a user obtaining service for a new CSI (e.g., phone number or email address) from a CSIP, the discovery service must enable immediate discoverability when the user associates the CSI with a MSP. The MSP MUST implement a mechanism to validate the user's control over the claimed CSI (as usual).

# Security Considerations

## Blackhole Prevention

The discovery service must be designed to prevent malicious MSPs from falsely claiming association with a CSI to prevent messages intended for the legitimate user from being delivered. This includes:

- Preventing message interception: Malicious MSPs MUST NOT be able to redirect messages by associating themselves with a CSI they do not control.
- Ensuring accurate discoverability: Malicious MSPs MUST NOT be able to make a non-user appear discoverable, leading to misdirected messages or false impressions of service adoption.

## DDoS, Enumeration and Spam Prevention and Mitigation

The discovery service MUST put in place robust mechanisms to prevent and mitigate DDoS, large-scale CSI scraping and spamming by malicious providers (MSPs, DPs). This requirement includes:

- Anti-DDoS, anti-enumeration and anti-spam defenses: The system design must effectively thwart attempts to DDoS attack, enumerate CSIs (especially phone numbers), and prevent the creation of spam target lists. Techniques may include restrictions on bulk queries, obfuscation, or differential access levels based on MSP reputation and relationships.
- Flexible rate limiting: DPs must be able to enforce rate limits on discovery requests, with mechanisms to determine appropriate limits based on individual MSP behavior, business relationships, and potential risks.

## Encryption and Authentication

All information exchanged between clients, DPs, and MSPs MUST be encrypted in transit and authenticated.


## Notes

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}
We would like to acknowledge and express our appreciation for the thoughtful feedback and constructive discussions that took place during the MIMI interim meetings focused on the discovery problem.
