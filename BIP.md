```
  BIP: ?
  Layer: Consensus (soft fork)
  Title: Safe Redaction
  Authors: Laissez Faire BTC <laissez.faire.btc@gmail.com>
  Status: Draft
  Assigned: ?
  Licence: CC0-1.0 OR MIT-0
  Discussion:
    2025-12-06: https://gnusha.org/pi/bitcoindev/CABHzxrjfvyBRD7sG9rngvDhr9cfzLEQibn4bup_J8pz7UHQpqA@mail.gmail.com/T/
    2025-11-20: https://gnusha.org/pi/bitcoindev/aTl8Y7p4qtYAsHbP@petertodd.org/T/
  Version: 0.0.0
  Requires: 3
```

## Abstract

This **Specification BIP** defines a new type of transaction output to enable nodes to safely and robustly redact objectionable content from the blockchain (within reason).

Any participant MAY write a _Redaction Statement_ to the blockchain. A Redaction Statement specifies which bytes of data will be redacted from the blockchain, and exactly how to safely redact those bytes. Once this transaction is committed to the blockchain, any participant MAY apply the Redaction Statement to safely redact the specified content from their node.

Where two participants wish to redact the same content, redacted data can be shared between nodes, for example as part of an initial block download.

The elements of the Redaction Statement workflow (including writing, mining, confirming, applying and sharing redaction statements) are each verifiable, trustless operations, that do not rely on any third party.

## Motivation

## Specification

## Rationale

## Backward Compatibility

## Reference Implementation

## Changelog

* __v0.0.0__ (2025-12-25):
  * initial draft

## Copyright

This BIP is in the public domain. No Rights Reserved.

## Related Work

- ["Redactable Blockchain in the Permissionless Setting", Deuber et al, 2019](https://arxiv.org/abs/1901.03206)
- ["Redactable Blockchain: Comprehensive Review, Mechanisms, Challenges, Open Issues and Future Research Directions", Abd Ali et al, 2023](https://www.mdpi.com/1999-5903/15/1/35)