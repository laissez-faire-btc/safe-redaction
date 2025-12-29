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
  Requires: 3, 143, 340
```

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

Note that the force of these words is modified by the requirement level of the document in which they are used. In this case, that requirement level is a soft fork, meaning that node implementations MAY ignore this BIP entirely, including all of its requirements.

## Abstract

This Specification BIP defines a new type of transaction output to enable nodes to safely and robustly redact objectionable content from the blockchain (within reason).

Any participant MAY write a _Redaction Statement_ to the blockchain. A Redaction Statement specifies which bytes of data will be redacted from the blockchain, and exactly how to safely redact those bytes. Once this is committed to the blockchain, any participant MAY apply the Redaction Statement to safely redact the specified content from their node.

Where two participants wish to redact the same content, redacted data MAY be shared between nodes, in redacted form; for example, as part of an initial block download.

The elements of the Redaction Statement workflow (including writing, mining, confirming, applying and sharing redaction statements) are each verifiable, trustless operations, that do not rely on any third party or authority.

## Motivation

**What problem does Safe Redaction address?**

Bitcoin is money for everybody, but the current rules require that to be a full participant, you must be willing to hold any arbitrary data that is mined, no matter how objectionable. There are, no doubt, people and businesses who would like to join the network, but will not do so under these rules, because they cannot currently do so safely.

**How does Safe Redaction improve things?**

This BIP provides a means to remove objectionable content from a node, with minimal impact, in line with the following design goals:

* optional - each node gets to decide what to redact, if anything 
* safe - provably no harm is done to those not choosing to use it, and any cost or risk to those using it is well understood, minimal, and mitigated
* full node functionality - a node that does redact content can still do everything it could have done otherwise, without relying on anyone else
* retrospective - content that exists on the blockchain today (pre-implementation) can be redacted later (post-implementation)
* trustless, verifiable, permissionless - Redaction Statements enabling data to be redacted are simple verifiable statements of fact that can be written by anybody
* lightweight - minimal changes and impact to policy, consensus, implementation, usage, the economy
* granularity, ~associativity~, commutativity, idempotence - the least possible data is redacted, and the ordering of Redaction Statements is inconsequential
* transferable - nodes that choose to redact objectionable content can share those blocks (with content redacted) with others who hold the same objection, so that the receiver may never even momentarily hold the objectionable content

## Specification

**How does Safe Redaction work?**

The proposal includes the following elements:

* A basic concept of how signed data can be modified, without modifying the signature, but while maintaining the verification and integrity benefits of the signature.
* A new data format for a *Redaction Statement*, which can tell a node how to safely redact some data out of an earlier transaction, while still allowing the node to validate the now-redacted transaction.
* A description of the *Redaction Operation*, which is where a node applies a Redaction Statement to redact data out of the node's local copy of the blockchain.
* A new *Consensus Rule*, to disallow a Redaction Statement if the statement does not correctly express a safe Redaction Operation, with a description of how a Redaction Statement can be written into a transaction, signed, broadcast, validated,  mined, and confirmed in line with other transaction types.
* A description of the process by which nodes can share redacted data with other nodes, for example as part of initial block download.

**Modifying Signed Data**

This section starts with a recap of how digital signatures work in Bitcoin, and then describes how we can change this process to work for redacted data.

How data is signed, in general:

* the data to be signed is first hashed
* the hash of the data is then input to the signing algorithm, along with a private key, to produce the signature data

How a signature is verified, in general: 

* the signed data is first hashed 
* the hash of the data is then input to the signature verification algorithm, along with a public key, to assess whether the signature is valid for the given hash, and given key 

If the signature verifies, it tells you some facts about exactly what happened in the original signing operation that created the signature data: 

* the corresponding private key was used to create the signature
* the given hash was used to create the signature

Crucially, note that if you hold the hash and the public key, you can verify the signature without the original data. The reason we normally need the original data, is because it's normally the only way to verify that the statements made in the original data were committed to by the holder of the private key. You hash the data, and use the hash to verify the signature.

The Redaction Statements we will create will attest to both the original hash of the unmodified data, and the updated hash of the redacted data. Then, with only the Redaction Statement and the redacted data, it is possible to verify the correctness of the redacted data, while also verifying that the original unmodified data was properly signed. This is performed as follows: 

* hash the current (redacted) data, and confirm it matches the expected value provided in the Redaction Statement
* retrieve the original hash value (for the unredacted data) from the Redaction Statement 
* use this original hash value to verify the signature, confirming that the original (unredacted) data was properly signed (even though we may not see it)

**The Redaction Statement**

```
<redaction-statement> ::= <uuid> <transaction-hash> <data-segment-list> <transaction-hash-update> <signature-hash-update-list>
```

Where loosely speaking:
* `<redaction-statement>` = the Redaction Statement, which is all the information needed to safely apply a redaction to a transaction, or to later validate that redacted transaction
* `<uuid>` = a specific 16 byte value used (only and always) to signify that this data is a redaction statement
* `<transaction-hash>` = the id of the transaction being modified
* `<data-segment-list>` = a sequence of pairs of numbers, each pair being first the index of the first byte to redact, and second the number of bytes to redact - with the entire list prepended with the length of the list (two bytes, little endian)
* `<transaction-hash-updated>` = the new transaction hash (transaction id)
* `<signature-hash-update-list>` = a list of pairs of sighashes, `<sighash-original> <sighash-updated>`, each being first the sighash of the original (unredacted) data, then the sighash of the updated (redacted) data, to be used for one signature in the transaction

The `<redaction-statement>` does not explicitly include the length of the `<signature-hash-update-list>`. The length of this list is implied. The list includes every signature in the transaction that is altered by the specified redaction, and none of the signatures that do not change.

**The Redaction Operation**

The semantics of the Redaction Statement are as follows: 

* The redaction is to be applied to the transaction specified by `<transaction-hash>`.
* To apply the redaction, change all bytes specified by `<data-segment-list>` to 0x00.
* After applying the redaction, the new hash of the redacted transaction will be `<transaction-hash-updated>`. Meanwhile, the original `<>` can still be used to refer to this transaction (for example, in future transaction inputs), search for this transaction, and in this block's Merkel tree.
* After applying the the redaction, when `<sighash-updated>` is found to be the input for signature verification in this transaction, it can safely be replaced by `<sighash-original>` for signature verification purposes. If the signature is a valid signature for `<sighash-original>`, this confirms that the only changes to the parts of this transaction that are committed to by this sighash, since signing, are the redaction specified in `<data-segment-list>`.

**Further Redaction For A Redacted Transaction**

It is not supported to apply a redaction to an already-redacted transaction. However, it is supported to have two Redaction Statements that specify the same transaction. In this way, a heavier redaction can entirely replace a lighter one. The node operator has the option to apply either, or neither, but not both.

**The Consensus Rule**

Redaction Statements are stored in OP_RETURN data. This BIP disallows some specific OP_RETURN output scripts, where they represent unsafe redactions.

This BIP changes the interpretation of OP_RETURN outputs, separating them into three categories with different handling:

Category 1: a valid Redaction Statement. If the data embedded in OP_RETURN starts with the well known magic number `<uuid>`, and if the Redaction Statement is well formed and represents true facts about a safe redaction, then this is a valid output. 

Category 2: an invalid Redaction Statement. If the data embedded in OP_RETURN starts with the well known magic number `<uuid>`, and if the Redaction Statement either is not well formed, or represents untrue statements about a redaction, then this is a invalid transaction. 

Category 3: not a Redaction Statement. If the data embedded in OP_RETURN does not start with the well known magic number `<uuid>`, then this BIP does not apply. 

Precisely, Categories 1 and 2 cover only outputs with scripts that start with the following data. Anything else is considered to be in Category 3.

`OP_RETURN OP_PUSHDATA2 <byte> <byte> <uuid> ...`

To be a consensus-valid Redaction Statement, the output MUST also be a valid output under general consensus rules.

When considering Category 1 (valid Redaction Statements), the following requirements are applied: 

1. `<transaction-hash>` MUST be a valid transaction id.
2. `<uuid>` MUST be the specific 16 byte value representing Redaction Statements, as defined in this BIP.
3. `<data-segment-list>` MUST NOT include any invalid byte indices or segment lengths. 
4. each segment in `<data-segment-list>` MUST be either wholly contained within a single input, or wholly contained within a single output.
5. each segment in `<data-segment-list>` MUST redact at least one non-zero byte.
6. `<transaction-hash-updated>` MUST be equal to the hash of the transaction after all of the bytes specified by `<data-segment-list>` are changed to 0x00, with no other changes.
7. `<signature-hash-update-list>` MUST include before-and-after (unredacted and redacted) sighashes for each signature that is invalidated by the redaction, and MUST NOT include any other sighashes.
8. The Redaction Statement MUST NOT include any further data after the `<signature-hash-update-list>`

**Sharing Redacted Data**

TODO

## Security Implications ##

**What's the impact on security? What new attacks are possible, and can they be mitigated?**

After the soft fork adopting this BIP, a new *Redaction Attack* becomes possible, targeting unsuspecting nodes. It requires a victim node that is willing to adopt a specific redaction, without seeing the original data, during initial block download (IBD), and the attacker must have 51% of the hashing power of the network.

* Step 1: Maliciously modify an existing transaction (*T1*).
* Step 2: Create an unsafe Redaction Statement, falsely claiming that T1 can safely be redacted, and that the updated hash matches the maliciously modified T1.
* Step 3: Using a 51% attack, mine and confirm this invalid transaction (*T2*). Continue the 51% attack until this Redaction Attack is successful.
* Step 4: Using separate channels, encourage the victim to accept this Redaction Statement. For example, claim that the original unredacted transaction has illegal content in it.
* Step 5: Offer the redacted blockchain to the victim for IBD, including the maliciously modified T1, and the unsafe Redaction Statement.
* Step 6: The victim accepts this blockchain, with this redaction applied. The victim sees that T1 is redacted, but the Redaction Statement in T2 tells them that the transaction hash they see is valid for the changes made to the transaction (which is not true), and that only insignificant non-financial data was changed (which is not true).
* Conclusion: The victim has accepted a maliciously modified transaction, T1.

## Rationale

**Why is this the right solution to the problem?**

This proposal doesn't impose any definition of objectionable content on anyone, or any process for determining what should count as objectionable. This is entirely up to each individual to decide for themself.

**Why not support further redaction of a redacted transaction?**

This would introduce additional complexity, and put additional burden on validators. Such support is also unnecessary, because it's possible to replace a redacted transaction with a different redacted transaction, that applies a different Redaction Statement.

**Why use OP_RETURN to store the Redaction Statement?**

It works, it's simple to implement, and it's a soft fork.

**Why not use a new opcode like OP_SUCCESSx, that's intended for future soft forks like this?**

OP_SUCCESSx and similar upgradable opcodes work by being committed to in the output of one transaction, and then revealed in the input of another transaction. That requires two transactions to publish the Redaction Statement to the blockchain. Whereas OP_RETURN can be put into the output of any transaction, as a one off.

**Why not use Witness Version 2 script, that's intended for future soft forks?**

That would be overkill.

**Concerns Raised**

The rest of this section is reserved for sharing, discussing and addressing concerns about this BIP. You can help by sharing, discussing or addressing concerns - on [the Discord server](https://discord.gg/DPXKfd9K3s), on the mailing list, or directly.

TODO

## Backward Compatibility

**How will activation work, and how will we know it has happened?**

TODO

**How will existing nodes and existing blocks handle this?**

TODO

## Reference Implementation

**Has this been implemented, in any way, shape or form?**

No.

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