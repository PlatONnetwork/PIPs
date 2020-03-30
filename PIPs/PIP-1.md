---
pip: 1
topic: PIP Purpose and Guidelines
status: Active
type: Meta
author: 
created: 2020/02/26
updated: 2020/02/26
---

## What is a PIP?

PIP stands for PlatON Improvement Proposal. An PIP is a design document providing information to the PlatON community, or describing a new feature for PlatON or its processes or environment. The PIP should provide a concise technical specification of the feature and a rationale for the feature. The PIP author is responsible for building consensus within the community and documenting dissenting opinions.

## PIP Rationale

We intend PIPs to be the primary mechanisms for proposing new features, for collecting community technical input on an issue, and for documenting the design decisions that have gone into PlatON. Because the PIPs are maintained as text files in a versioned repository, their revision history is the historical record of the feature proposal.

For PlatON implementers, PIPs are a convenient way to track the progress of their implementation. Ideally each implementation maintainer would list the PIPs that they have implemented. This will give end users a convenient way to know the current status of a given implementation or library.

## PIP Types

There are some types of PIP:

Need to vote on the chain：
- **Parameter**： Proposals for Parameter track must modify governable parameters on the chain. 
- **Upgrade**： Proposals for Upgrade track must initiate a voting upgrade on the chain, a complete installation package needs to be prepared before initiating the proposal.
- **Cancellation**：Proposals of Cancellation track are used to cancel the upgrading and parameter proposals in voting in the chain. We will only initiate this proposal in exceptional circumstances.

Text proposals include the following two types, and in general, no voting is required. Votes can be initiated through text proposals in important or divergent situations.

- **Meta** : Proposals made to the Meta track must affect changes to PIP-1. 
- **Requirement**:  Proposals made to the Meta track must be clear about new functions, optimization functions, etc.

## PIP Work Flow

### PIP Process 

Following is the process that a successful voting PIP will move along:
[ DRAFT ] -> [ FINAL ] -> [ ACCEPTED ] -> [  VOTE ] -> [ PASS ]

Following is the process that a successful non-voting PIP  will move along:
[ DRAFT ] -> [ FINAL ] -> [ ACCEPTED ] 

####  Draft
Once the champion has asked the PlatON community whether an idea has any chance of support, they will write a draft PIP as a [pull request].The title of the draft is: Draft: XXXX. Use a template from the Templates section to ensure you are including all the necessary information. 

:arrow_right: ​Final：At this stage you need to collect their feedback by sharing PR in the PlatON community, and their feedback will help you improve your draft. You can submit changes to the draft through multiple commit until you think the content is perfect, then you can modify the title of the proposal as "Final: XXXX"  and notify PIP Editor through comment.

A request for Final status will be denied if material changes are still expected to be made to the draft. We hope that PIPs only enter Final once, so as to avoid unnecessary noise on the RSS feed.

#### Final

PIP Editor will assign the PIP a number (generally the  PR number related to the PIP) and merge your pull request. The PIP Editor will not unreasonably deny a PIP.

If the PIP Editor deny a PIP., PR will be closed. Reasons for denying draft status include being too unfocused, too broad, duplication of effort, being technically unsound, not providing proper motivation or addressing backwards compatibility, or not in keeping with the PlatON philosophy.

The merged PIP will be reviewed by the Core Devs from a more professional and comprehensive perspective. The audit content includes the evaluation of the feasibility and integrity of the proposal.

- :arrow_right: Accepted: The review will last for two weeks. If the review is passed, PIP Editor will modify the proposal status to "Accepted".

- :arrow_right: Draft: The Core Devs can decide to move this PIP back to the Draft status at their discretion. E.g. a major, but correctable, flaw was found in the PIP.

- :arrow_right: Rejected: The Core Devs can decide to mark this PIP as Rejected at their discretion. E.g. a major, but uncorrectable, flaw was found in the PIP.

#### Accepted 

Any further changes are unlikely and PIP client developers should consider this PIP for inclusion. Once a PIP is accepted, a reference implementation needs to be created.

- :arrow_right: Vote​ ：Parameter proposal, upgrade proposal and cancellation proposal all need to vote on the chain.

#### Vote

Once the parameters, upgrade and cancellation proposals are accepted, voting can be initiated on the chain.

**Note**：Only alternative nodes can initiate on chain proposals. If the PIP author is not an alternative node, it can be initiated by any alternative node instead.

- :arrow_right: Pass: after voting on the chain, PIP editor needs to modify the proposal status to pass.
- :arrow_right: Fail: when the voting on the chain fails, PIP editor needs to modify the proposal status to fail.

Other exceptional statuses include:

- **Active** -- Some PIPs may also have a status of “Active” if they are never meant to be completed. E.g. PIP 1 (this PIP).

- **Abandoned** -- This PIP is no longer pursued by the original authors or it may not be a (technically) preferred option anymore.

    :arrow_right: ​Draft -- Authors or new champions wishing to pursue this PIP can ask for changing it to Draft status.

- **Rejected** -- A PIP that is fundamentally broken or a Core PIP that was rejected by the Core Devs and will not be implemented. A PIP cannot move on from this state.

- **Superseded** -- A PIP which was previously Accpeted but is no longer considered state-of-the-art. Another PIP will be in Accpeted status and reference the Superseded PIP. An PIP cannot move on from this state.

- **Cancelled** --The PIP's status will change to "Cancelled" which means the PIP was cancelled by a passed Cancellation Proposal.

## What belongs in a successful PIP?

Each PIP should have the following parts:

- Preamble - RFC 822 style headers containing metadata about the PIP, including the PIP number, a short descriptive title (limited to a maximum of 44 characters), and the author details. See [below](#pip-header-preamble) for details.
- Abstract - A short (~200 word) description of the technical issue being addressed.
- Motivation (*optional) - The motivation is critical for PIPs that want to change the PlatON protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the PIP solves. PIP submissions without sufficient motivation may be rejected outright.
- Specification - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current PlatON platforms .
- Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.
- Backwards Compatibility - All PIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The PIP must explain how the author proposes to deal with these incompatibilities. PIP submissions without a sufficient backwards compatibility treatise may be rejected outright.
- Test Cases - Test cases for an implementation are mandatory for PIPs that are affecting consensus changes. Other PIPs can choose to include links to test cases if applicable.
- Implementations - The implementations must be completed before any PIP is given status “Accepted”, but it need not be completed before the PIP is merged as draft. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of “rough consensus and running code” is still useful when it comes to resolving many discussions of API details.
- Copyright Waiver - All PIPs must be in the public domain. See the bottom of this PIP for an example copyright waiver.

## PIP Formats and Templates

PIPs should be written in [markdown] format.
Image files should be included in a subdirectory of the `assets` folder for that PIP as follows: `assets/pip-N` (where **N** is to be replaced with the PIP number). When linking to an image in the PIP, use relative links such as `../assets/pip-1/image.png`.

Below is a list of PIP templates for each type. Copy the template for the your PIP, fill it out, and submit the pull request with your PIP for review.  Note that all proposals must be licensed CC-0.

- [Cancellation](../templates/Cancellation-template.md)
- [Parameter](../templates/Parameter-template.md)
- [Upgrade](../templates/Upgrade-template.md)
- [Meta](../templates/Meta_template.md)
- [Requirement ](../templates/Requirement_template.md)

## PIP Header Preamble

Each PIP must begin with an [RFC 822](https://www.ietf.org/rfc/rfc822.txt) style header preamble, preceded and followed by three hyphens (`---`). This header is also termed ["front matter" by Jekyll](https://jekyllrb.com/docs/front-matter/). The headers must appear in the following order. Headers marked with "*" are optional and are described below. All other headers are required.

` pip:` *PIP number* (this is determined by the PIP Editor)

` topic:` *PIP topic*

` author:` *a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). Details are below.*

` * discussions-to:` *a url pointing to the official discussion thread*

` status:` *Draft | Final | Accepted | Vote| Pass | Fail | Active| Abandoned | Rejected | Superseded*

` type:` Parameter| Upgrade| Cancellation | Meta |Reqirement

description: brief summary

` created:` *date created on*

` * updated:` *comma separated list of dates*

` * requires:` *PIP number(s)*

` * replaces:` *PIP number(s)*

` * superseded-by:` *PIP number(s)*

` * Cancelled-by:` *PIP number(s)*

` * resolution:` *a url pointing to the resolution of this PIP*

Headers that permit lists must separate elements with commas.

Headers requiring dates will always do so in the format of ISO 8601 (yyyy-mm-dd).

## PIP Editor Responsibilities

For each new PIP that comes in, an Editor does the following:

- Read the PIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the PIP for language (spelling, grammar, sentence structure, etc.), markup (Github flavored Markdown), code style

If the PIP isn't ready, the Editor will send it back to the author for revision, with specific instructions.

Once the PIP is ready for the repository, the PIP Editor will:

- Assign a PIP number (generally the PR number)
- Merge the corresponding pull request
- Send a message back to the PIP author with the next step.
- Change the PIP status from Final to Accepted, if the PIP is approved by the core developer.

Many PIPs are written and maintained by developers with write access to the PlatON codebase. The PIP editors monitor PIP changes, and correct any structure, grammar, spelling, or markup mistakes we see.

The editors don't pass judgment on PIPs. We merely do the administrative & editorial part.

## History

This document borrows from [Ethereum’s EIP-1](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1.md) written by Martin Becze and Hudson Jameson, which itself was derived from [Bitcoin's BIP-0001](https://github.com/bitcoin/bips/blob/master/bip-0001.mediawiki) written by Amir Taaki, which in turn was derived from [Python's PEP-0001](https://www.python.org/dev/peps/pep-0001/). In many places text was simply copied and modified.  Although the PEP-0001 text was written by Barry Warsaw, Jeremy Hylton, and David Goodger, they are not responsible for its use in the PlatON Improvement Process, and should not be bothered with technical questions specific to PlatON or the PIP. Please direct all comments to the PIP editors.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
