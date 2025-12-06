# a safe way to remove objectionable content from the blockchain

I propose two changes to Bitcoin, one at the consensus level, and one at the client level. The purpose of these changes is to support filtering of objectionable content after the content has been mined, allowing each node operator to maintain only that data they find agreeable. In so doing, my hope is that we can address all users' greatest concerns, for a true consensus resolution.

I do however acknowledge those people that want to stop miners from mining non-monetary transactions, because of the data storage and processing cost, and I recognised that this proposal does nothing to address those concerns.

**Motivation**

You can't just change or delete some data from the blockchain, because a hash of each transaction is in the transaction signature, and a hash of each block is in the next block. If you change the data, you change the hashes, and you break things.

The design presented here is an attempt to achieve a compromise, where a person can have all of the benefits of running a full archival node, including the integrity of the ledger, yet without storing the objectionable content - and importantly without even being able to recreate that objectionable content from what data they still have.

**Preliminary**

Objectionable content is defined here as whatever you decide it is, and two users don't have to share the same views. One person might object to copyrighted material used without permission, another a negative depiction of the prophet Muhammad, and another video of the sexual abuse of children. The design presented below lets each person decide what to remove for themself (if anything), while those who want everything still have it all.

**Solution**

If you take some arbitrary data that's in a transaction, written into the blockchain, and you change part of it (e.g. to all zeros) to remove some objectionable content, then you also change any hash that it's been used in. For example, that might be signatures in transaction inputs, or a transaction hash in a block's Merkel tree.

The solution presented below is based on two ideas, both aimed at maintaining data and signature integrity through hashing, while changing some of the hash's input data.

**First idea**

If you know that changing the data from D to D' changes the hash from H to H', and if you confirm that D' hashes to H', then you know that H is the correct hash of the original data D, even without seeing D. If you know all that, then you can safely update D to D', removing the objectionable content. You can still confirm that the only thing that changed was the change you knew about. With that knowledge you can continue to validate a signature, or you can use the original hash in construction of a Merkel tree.

Of course that's a big 'if', at the beginning of that paragraph. That 'if' is doing a lot of work.

The easiest way to obtain D' and H' securely is to generate them yourself: update D to D' to remove the objectionable content, hash D' into H' (while keeping a copy of H). But that has multiple problems. You have to hold the objectionable data, at least for a time. There's also no evidence that the original D ever hashed to H, beyond the fact that you hold D' and H'. You certainly don't have any evidence that could be shared and used by anyone else.

But what if instead, we could share H' across the network, and do it securely and verifiably, and in a way where up to 100% of nodes could choose to make this change to D, permanently, without breaking anything?

**Second idea**

It may seem like there is no one you can trust to tell you what H' is. There is only one source of data that a Bitcoin node can trust, and that is the blockchain, as mined by miners, with the most proof of work, and verified locally. Therefore, the second idea is that H' can be trusted if (and only if) it is written into the blockchain, and verified by the network.

For example, we write data to the semantic effect of "In Transaction X: changing data (to all zeros) between byte offset A and byte offset B has the effect of changing the transaction hash to H', and the transaction signatures' signed hash values to S1', S2' ... Sn'." Miners then validate and mine this statement into a block, and verifiers confirm that it is cryptographically accurate with respect to the data in Transaction X, as described - or else they drop the new block as invalid.

Once this statement is established in the blockchain, any node can choose to delete (change to all zeros) the data between A and B in Transaction X. This can now be done with confidence because they can double check the accuracy, and the impact on the ledger, before they delete the data. After that they may also be able to share (with the agreement of the receiving node) Transaction X as part of initial block downloads - along with H', S1', S2' ... Sn' - to any other nodes that don't want the objectionable content. The receiving nodes wouldn't immediately and necessarily be able to verify and rely on H' etc., but they would eventually, once they have the full blockchain.

**Putting it all together**

Here's some very loose pseudocode showing the steps to remove the objectionable content:
```
1. write the following to the blockchain:
1.a) the transaction (T) to be changed 
1.b) the exact data (D) to be changed, specified
by byte offset, and length (or multiple byte
offsets, and lengths)
1.c) the current hash (H) of T
1.d) the new hash (H') of T, after D is changed
1.e) for any signatures that sign D (for example, 
signatures in transaction inputs), the current 
hash (S) and the new hash (S') of that signed 
data (the signed data includes D, but the hash of 
the signed data is different to the hash of D, 
and different to the hash of T)
2. miners and validators check that this 
represents a true statement about T, D, H, H', S, 
S', and the change is safe (e.g. it is arbitrary 
data, not structural), and it is written into a 
block
3. a client sees this removal and chooses to use 
it, additionally confirming that the removal is 
valid and safe
4. a client may further share this transaction or 
this block in future, in its altered state, along 
with the information necessary to validate it in 
its altered state (which will be found and can be 
confirmed later in the blockchain)
```

**Hard fork, soft fork**

I can see a few ways to implement this consensus change. The most straightforward is a new opcode, where we push the facts we want to verify onto the stack, and then the new opcode verifies that they meet the rules for what can be deleted, per this proposal. Much as how OP_RETURN just accepts data and moves on, this new opcode would do the same - but with the exception that we require the data to be well formed, and we require the implied statement about hashes to be true, or else the transaction is rejected.

Slightly less obvious and slightly more interesting is the possibility of a soft fork. One way we might do that is to use an arbitrary data field and assign meaning to that arbitrary data. For example, we might define a data format to represent a deletion statement. Then, any data that is a well-formed deletion statement is not allowed to contain false statements about transaction hashes. True statements are still allowed, as is any arbitrary data that does not conform to the definition of a well-formed deletion statement.

Because this is purely a restriction on otherwise-acceptable transactions, this would be a soft fork.

Here is a concrete example of how we might differentiate well formed deletion statements from other arbitrary data.
```
<deletion-statement> ::= <uuid> <transaction-id> <data-segment-list> <transaction-hash-update> <signature-hash-update-list>
```

Where loosely speaking:

* `<deletion-statement>` = the entire contents of an op return output's data pushes
* `<uuid>` = a specific 16 byte value used (only and always) to signify that this data is a deletion statement
* `<transaction-id>` = the id of the transaction being modified
* `<data-segment-list>` = a sequence of pairs of numbers, each pair being first the index of the first byte to delete, and second the number of bytes to delete - with the entire list prepended with the length of the list 
* `<transaction-hash-update>` = the new transaction hash (transaction id)
* `<signature-hash-update-list>` = a list of hashes, each of which is the new hash to be used for one signature in the transaction - not prepended by the length of the list because the data changed (`<data-segment-list>`) is sufficient to uniquely describe exactly which hashes change

**FAQs**

**Q1. Isn't this basically just Simplified Payment Verification / BIP 157 / Neutrino?**

The similarity is the reliance on proof of work - but actually all nodes do that. Regular full nodes rely on PoW to order transactions and avoid double spending - which is what gives Bitcoin value as money. This solution additionally uses PoW to verify that some data can be safely and securely deleted (optionally, of course). SPV nodes rely on PoW to check that a transaction has been verified and accepted by the rest of the network, so that the SPV node doesn't have to store and verify it.

**Q2. Doesn't this lower security, remove autonomy, and introduce trust?**

It certainly increases the attack surface - it's another feature, which is another target for vulnerabilities. So it's not zero cost, for security, but the cost is very low. It doesn't introduce any more trust in the system. That's what the consensus rule does. Only safe deletions of arbitrary data are allowed in, and even then node operators decide which (if any) they want to apply.

**Q3. Doesn't this allow an attacker to modify the hash of a transaction, and modify the contents of the transaction, thereby bypassing the transaction signature? Why sign a transaction if anyone can just modify it later?**

Yes, it does allow an attacker to modify the hash of a transaction, and to modify the contents of the transaction. But the new consensus rule ensures that we know what the hash was originally, and we know exactly what data was changed and what wasn't. That gives the node operator enough information to decide if it's safe to accept the change. For example, in an OP_RETURN with a single data push, the contents of that data push are safe to modify. But if the attacker wants to modify the transaction's op codes or signature, the change isn't going to pass consensus, and it isn't going to make it into the blockchain.

**Q4. Why not extend this to a full data hiding scheme like ZKPs, Monero, ZeroSync? Do it right, once and for all?**

What I've presented here is a simple, opt-in solution, with low overheads. Implementation could be quite lightweight in terms of code changes, and quite fast if needed. For the problem this attempts to solve, those other solutions are severe overkill.

**Q5: In the event of miner and node collusion, such that the blockchain is effectively altered: what harm can occur, and how might it be mitigated or limited?**

This is a 51% attack, and the changes proposed here do increase the blast radius of that attack. The usual 51% scenario lets the attacker double spend their funds. This new 51% attack lets the attacker alter a transaction and bypass the transaction signature, allowing them to spend other people's money. But only for anyone who wants to accept the change to the transaction. Certainly, we can expect the regular 51% attack to remain more profitable for attackers, because it affects everyone.

**Q6: What data is safe to remove from the blockchain?**

Data pushes that contain arbitrary data, that's not relevant to future transactions. At the very least, data in an OP_RETURN output data push. There will be other examples, but there will also be an unclear boundary between the safe and the unsafe. Node operators will continue to have the final say on whether to apply a change.

**Q7: What data is unsafe to remove from the blockchain?**

Parts of the transaction that includes op codes or signatures. Changing these would change the meaning of the transaction (in relation to past and future transactions on the blockchain). Therefore, these kinds of changes should not be accepted into the blockchain.

**Q8: Why do miners need to enforce the safety of deletions at the consensus level? Why not have miners accept all deletions, and leave it to nodes to decide what to accept?**

If we do it at the consensus level, then future nodes may be able to accept deletions from peers (e.g. in initial block download) without ever having to hold the objectionable content - because they know that each deletion is only a data push, and the op code semantics of transactions haven't changed. If it's left to nodes to decide what's safe, then a node can't really accept a change from another node without seeing the original transaction first - because that change may have altered the semantics and the financial effect of the transaction.