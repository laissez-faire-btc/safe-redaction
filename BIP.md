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
    2025-12-06: https://gnusha.org/pi/bitcoindev/CABHzxrjfvyBRD7sG9rngvDhr9cfzLEQibn4bup_J8pz7UHQpqA@mail.gmail.com/T/
    2025-11-20: https://gnusha.org/pi/bitcoindev/aTl8Y7p4qtYAsHbP@petertodd.org/T/
  Version: 0.0.0
  Requires: 3
```

## Abstract

This **Specification BIP** defines a new type of transaction output to enable nodes to safely and robustly redact objectionable content from the blockchain (within reason).

Any participant MAY write a _Redaction Statement_ to the blockchain. A Redaction Statement specifies which bytes of data will be redacted from the blockchain, and exactly how to safely redact those bytes. Once this is committed to the blockchain, any participant MAY apply the Redaction Statement to safely redact the specified content from their node.

Where two participants wish to redact the same content, redacted data can be shared between nodes, in redacted form; for example, as part of an initial block download.

The elements of the Redaction Statement workflow (including writing, mining, confirming, applying and sharing redaction statements) are each verifiable, trustless operations, that do not rely on any third party or authority.

## Motivation

**What problem does Safe Redaction address?**

One of Bitcoin's greatest strengths is that it's permissionless, and it's uncensorable. But that can also be a bit of a problem.

The status quo is this: if you want to get some data into the blockchain, there is no restriction. You just follow the rules, and you pay by the byte. But if you want to get some data removed, that's a hard fork. Now you need 51% of the miners to agree to a new rule: this specific data is not allowed in the blockchain; not now, and not ever. You'll need to roll back to the block before it was mined, too.

People don't generally want to maximise the amount of arbitrary data they are holding on their nodes at other people's request, but (as of 2025) this is the only way to run a node. Take it or leave it. Bitcoin users are okay with this, because the benefits greatly outweigh the costs. 

* illegal content
* immoral and abhorrent content
* information hazards

* vast disparity of views

* support for businesses (who in many countries are held to a higher standard than individuals)

## Specification

**How does Safe Redaction work?**

## Rationale

**Why is this the right solution to the problem?**

## Backward Compatibility

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