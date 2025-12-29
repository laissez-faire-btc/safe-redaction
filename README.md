# a safe way to remove objectionable content from the blockchain

**Abstract**

This Specification BIP defines a new type of transaction output to enable nodes to safely and robustly redact objectionable content from the blockchain (within reason).

Any participant MAY write a _Redaction Statement_ to the blockchain. A Redaction Statement specifies which bytes of data will be redacted from the blockchain, and exactly how to safely redact those bytes. Once this is committed to the blockchain, any participant MAY apply the Redaction Statement to safely redact the specified content from their node.

Where two participants wish to redact the same content, redacted data MAY be shared between nodes, in redacted form; for example, as part of an initial block download.

The elements of the Redaction Statement workflow (including writing, mining, confirming, applying and sharing redaction statements) are each verifiable, trustless operations, that do not rely on any third party or authority.

[Read the full draft BIP](./bip-safe-redaction.md)

**FAQs**

**Q1. Isn't this basically just Simplified Payment Verification / BIP 157 / Neutrino?**

The similarity is the reliance on proof of work - but actually all node types do that. Regular full nodes rely on PoW to order transactions and avoid double spending - which is what gives bitcoin value as money. This solution additionally uses PoW to verify that some data can be safely and securely removed (optionally, of course). SPV nodes rely on PoW to check that a transaction has been verified and accepted by the rest of the network, so that the SPV node doesn't have to store and verify it.

**Q2. Doesn't this lower security, remove autonomy, and introduce trust?**

It certainly increases the attack surface - it's another feature, which is another target for vulnerabilities. So it's not zero cost, for security, but the cost is very low. It doesn't introduce any more trust in the system. That's what the consensus rule does. Only safe redactions of arbitrary data are allowed in, and even then node operators decide which (if any) they want to apply.

**Q3. Doesn't this allow an attacker to modify the hash of a transaction, and modify the contents of the transaction, thereby bypassing the transaction signature? Why sign a transaction if anyone can just modify it later?**

Yes, it does allow an attacker to modify the hash of a transaction, and to modify the contents of the transaction. But the new consensus rule ensures that we know what the hash was originally, and we know exactly what data was changed, and what wasn't. That gives the node operator enough information to decide if it's safe to accept the change. For example, in an OP_RETURN with a single data push, the contents of that data push are safe to modify. But if the attacker wants to modify the transaction's op codes or signature, the change isn't going to pass consensus, and it isn't going to make it into the blockchain.

**Q4. Why not extend this to a full data hiding scheme like ZKPs, Monero, ZeroSync? Do it right, once and for all?**

What I've presented here is a simple, opt-in solution, with low overheads. Implementation could be quite lightweight in terms of code changes, and quite fast if needed. For the problem this design solves, those other solutions are severe overkill.

**Q5. In the event of miner and node collusion, such that the blockchain is effectively altered: what harm can occur, and how might it be mitigated or limited?**

This is a 51% attack, and the changes proposed here do increase the blast radius of that attack. The usual 51% scenario lets the attacker double spend their funds. This new 51% attack lets the attacker alter a transaction and bypass the transaction signature, allowing them to spend other people's money. But only for anyone who wants to accept the change to the transaction. Certainly, we can expect the regular 51% attack to remain more profitable for attackers, because it can affect anyone, not just those who opt in.

**Q6. What data is safe to redact from the blockchain?**

Data pushes that contain arbitrary data, that's not relevant to future transactions. At the very least, data in an OP_RETURN output data push. There will be other examples, but there will also be an unclear boundary between the safe and the unsafe. Node operators will continue to have the final say on whether to apply a change.

**Q7. What data is unsafe to remove from the blockchain?**

Parts of the transaction that includes op codes or signatures. Changing these would change the meaning of the transaction (in relation to past and future transactions on the blockchain). Therefore, these kinds of changes should not be accepted into the blockchain.

**Q8. Why do miners need to enforce the safety of redactions at the consensus level? Why not have miners accept all redactions, and leave it to nodes to decide what to accept?**

If we do it at the consensus level, then future nodes may be able to accept redactions from peers (e.g. in initial block download) without ever having to hold the objectionable content - because they know that each redaction is only a data push, and the op code semantics of transactions haven't changed. If it's left to nodes to decide what's safe, then a node can't really accept a change from another node without seeing the original transaction first - because that change may have altered the semantics and the financial effect of the transaction.

**Q9. To redact a 100,000 byte OP_RETURN, do you need a 100,000 byte redaction transaction?**

No, redaction costs about 64 bytes *per signature*, regardless of how much data is changed in that signature's input data. The solution simply shows the signature hash value, both before and after applying the redaction. Then it's about 8 bytes of extra data for each non-contiguous chunk of data. For example, if a transaction had 100,000 bytes of objectionable data split across 200 data pushes, that would require about 1,600 bytes to precisely specify. Alternatively, you may be able to redact all of the data after OP_RETURN, including the PUSHDATA op codes. In that case, it costs only 8 bytes to specify. 

**Q10. Can a node that has made redactions then share redacted blocks via initial block download? I.e. do they operate as a full archival node, capable of re-seeding the entire Bitcoin, if all other nodes fail?**

Yes, redacted transactions are shareable, but only if the receiving node is happy to receive these redacted transactions, not the original. That's good for sharing with people who don't ever want the objectionable content landing on their device, and who share the same view of what's objectionable - eg for social, legal, moral, religious, economic or political reasons. And if there ever becomes a unanimous consensus about a bit of content, where 100% of nodes have redacted it (whether through armageddon taking out most nodes, or by everyone unanimously agreeing to redact something) then that content will be permanently redacted for everyone, and it will never be available to anyone ever again. (Of course, in practice, you might expect the node it originated from to maintain it unredacted, at least in most cases.)

**Q11. If a node accepts and applies a redaction, what happens in the case of a block reorg? Eg. that might be a few blocks as can happen from time to time by chance, or arbitrarily many blocks in the case of a 51% attack.**

Much like a transaction where you receive bitcoin, you may want to wait a few blocks for confirmation before you act on a redaction. If there is a reorg affecting the redaction statement after you apply the redaction, and if you permanently lose the redaction statement, then you no longer have proof that this redaction statement was correctly applied. In that case, you may want to reverse the redaction, by downloading the original unredacted block from peers.

**Q12. Exactly which data envelopes for arbitrary data will this support? And if there's one it doesn't support, can this be extended later?**

It's too soon to say, because there is a spectrum of complexity. At least OP_RETURN will be supported, and most likely some similar simple methods like inscriptions. On the other hand, it's possible to steganographically embed arbitrary data in public keys that are not revealed when used for outputs, but are later revealed when used in input signatures. Or those very public keys might include XOR-masked (simple encryption) arbitrary data to hide the objectionable content from view, while still making it extractable by those who know the scheme. At this end of the spectrum, these elements will probably never be supported for redaction. However, we don't need to decide this once and for all time. We can easily extend this solution later with a second soft fork, e.g. by using a new magic number (UUID) for the new redaction options.