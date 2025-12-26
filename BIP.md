```
  BIP: ? (unassigned)
  Layer: Consensus (soft fork)
  Title: Safe Redaction
  Authors: Laissez Faire BTC <laissez.faire.btc@gmail.com>
  Status: Draft
  Type: Specification
  Assigned: ? (unassigned)
  Licence: CC0-1.0 OR MIT-0
  Discussion:
    2025-12-26: https://discord.gg/DPXKfd9K3s
    2025-12-06: https://gnusha.org/pi/bitcoindev/CABHzxrjfvyBRD7sG9rngvDhr9cfzLEQibn4bup_J8pz7UHQpqA@mail.gmail.com/T/
    2025-11-20: https://gnusha.org/pi/bitcoindev/aTl8Y7p4qtYAsHbP@petertodd.org/T/
  Version: 0.0.0
  Requires: 3
```

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

Note that the force of these words is modified by the requirement level of the document in which they are used. In this case, that requirement level is a soft fork, meaning that node implementations MAY ignore this BIP entirely, including all of its requirements.

## Abstract

This Specification BIP defines a new type of transaction output to enable nodes to safely and robustly redact objectionable content from the blockchain (within reason).

Any participant MAY write a _Redaction Statement_ to the blockchain. A Redaction Statement specifies which bytes of data will be redacted from the blockchain, and exactly how to safely redact those bytes. Once this is committed to the blockchain, any participant MAY apply the Redaction Statement to safely redact the specified content from their node.

Where two participants wish to redact the same content, redacted data MAY be shared between nodes, in redacted form; for example, as part of an initial block download.

The elements of the Redaction Statement workflow (including writing, mining, confirming, applying and sharing redaction statements) are each verifiable, trustless operations, that do not rely on any third party or authority.

## Motivation

**What issue does Safe Redaction address?**

Bitcoin is money for everybody, but the current rules require that to be a full participant, to run an archival node, to contribute to the resilience of the network, to be entirely self sufficient, you must be willing to hold any arbitrary data that is mined, no matter how objectionable. There are, no doubt, people and businesses who would like to join the network, but will not do so under these current rules, because they cannot currently do so safely.

One of Bitcoin's great strengths is being permissionless and uncensorable. But that can also be a bit of a problem.

The status quo is this: if you want to get some data into the blockchain, there is no restriction. You just follow the rules, and you pay by the byte, at market rates. But if you want to get some data removed, that's a hard fork. Now you need 51% of the miners to agree to a new rule: that this specific data is not allowed in the blockchain. You'll need to roll back to the block before it was mined, too.

In practice, this means that *absolutely any data* can be written to the blockchain, and *absolutely no data* can be removed. 

An archival node can only be run by a person who is comfortable with this, and also comfortable with holding the data that is currently embedded in the blockchain - according to their conscience, and according to their local laws and law enforcement.

This BIP deals with the following issues:

* There is content in the blockchain today, that in some jurisdictions makes it illegal to run a node. 
* The practical options for Bitcoin users today are all-or-nothing. If you're not comfortable with arbitrary data on your system, you don't run a node. In that case, if you do want to use Bitcoin anyway, then you rely on others to run nodes and keep the network running. 
* Even for people who are comfortable with the data that is in the blockchain today, finding none of it objectionable, something they do find objectionable may get mined tomorrow.
* There is a vast diversity of views on what individuals consider objectionable. For example, while a majority of Europeans might care about the legality of storing personal information without the subject's consent (vis. the GDPR), the majority globally most likely do not. The same may well be true for most regions, most countries, most categories of potentially objectionable content, and even most individuals.
* The status quo may be more appropriate for individuals, but it is less appropriate for businesses. In many jurisdictions, businesses are held to a higher standard, and have greater restrictions placed on them by regulation. They may also have more to lose then many individuals.

## Specification

**How does Safe Redaction work?**

## Security Implications ##

**What's the impact on security? What new attacks are possible, and can they be mitigated?**

## Rationale

**Why is this the right solution to the problem?**

This proposal doesn't impose any definition of objectionable content on anyone, or any process in determining what should count as objectionable. This is entirely up to each individual to decide for themself.


* even miners can apply redactions, which will help with global diversification and miner decentralisation

## Backward Compatibility

**How will activation work, and how will we know it has happened?**

**How will existing nodes and existing blocks handle this?**

## Reference Implementation

**Has this been implemented, in any way, shape or form?**

## Changelog

* __v0.0.0__ (2025-12-25):
  * initial draft

## Copyright

This BIP is in the public domain. No Rights Reserved.

This work is available under [Creative Commons Zero v1.0 Universal](https://spdx.org/licenses/CC0-1.0.html).

This work is available under the [MIT No Attribution](https://spdx.org/licenses/MIT-0.html) licence.

## Related Work

**"Redactable Blockchain in the Permissionless Setting", Deuber et al, 2019,** [https://arxiv.org/abs/1901.03206](https://arxiv.org/abs/1901.03206)

An academic proposal to enable redaction through a hard fork. It would change Bitcoin by adding an additional Merkel tree of redacted data into the block header. It relied on a central authority to decide what to redact, and then redactions would be applied to all nodes.

**"Redactable Blockchain: Comprehensive Review, Mechanisms, Challenges, Open Issues and Future Research Directions", Abd Ali et al, 2023,** [https://www.mdpi.com/1999-5903/15/1/35](https://www.mdpi.com/1999-5903/15/1/35)

This is a recent literature review that considers various methods of redacting blockchains (including Bitcoin). It goes into depth about the various reasons that a participant may want to redact content. It also demonstrates that (as of 2023) there were no realistic and practical solutions to this problem.

Chameleon hashes:

Redactable signatures: https://people.eecs.berkeley.edu/~daw/papers/hom-rsa02.pdf

Sanitizable signatures: https://link.springer.com/chapter/10.1007/11555827_10