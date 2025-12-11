***Example 1 - valid deletion (a simple hypothetical example)***

**Stage 1 - a hypothetical transaction contains objectionable data**

In block 900,000, there is "Transaction 2" that spends a UTXO from "Transaction 1", in block 800,000. Transaction 2 includes an OP_RETURN output. The block is mined, and gets validated and committed to the blockchain.

Some people find the data in Transaction 2's OP_RETURN output objectionable.

**Stage 2 - a deletion statement is mined**

In block 1,000,000 there is "Transaction 3". Transaction 3 includes an OP_RETURN output. The data in the OP_RETURN output indicates that it is a deletion statement. This is because the data starts with the 16 byte magic number that represents deletion statements, as defined in the specification.

Next, after the magic number, Transaction 3's OP_RETURN data includes instructions on which transaction is being modified - in this case Transaction 2, specified by transaction id.

Next, Transaction 3's data specifies how many non-contiguous segments of data in Transaction 2 are going to be deleted. In this case, to keep things simple, let's assume it is one single contiguous blob of data to be deleted. After this, the Transaction 3 OP_RETURN data specifies a byte offset in Transaction 2 at which the deletion will start. It then goes on to specify how many bytes will be deleted.

At this point, this is already sufficient information to specify exactly what to delete. The rest of the data is what allows nodes to use this deletion statement safely, without a trusted third party. It can also be used after the deletion is applied by a node, to confirm and revalidate that only the specified alteration has been made.

All of the rest of the data from the deletion statement is a series of new hashes. The first one is the updated hash for Transaction 2. This is the hash of the transaction after the specified deletion has been applied.

The rest of the hashes are updated values for the hashes used in each of the affected signatures in the transaction. In our example, there is only one signature, which is in the single input in Transaction 2. The existing signature in this input covers the relevant data in this input of Transaction 2, but also commits to the OP_RETURN output of Transaction 2, including the data being modified by this deletion statement. Therefore, the hash for this signature is changing.

In this simple example, only two hashes are changing: the transaction hash, and a single input signature hash. For each hash, we include both the original hash value, and the updated hash value. The original value of the transaction hash is the transaction id, which was included already at the start of this deletion statement payload, so it does not need to be repeated here. The payload only includes the updated transaction hash, followed by the original signature hash, and then the updated signature hash.

That completes the deletion statement payload. In summary, we have the following data in the payload, all of which is written into Transaction 3's OP_RETURN output:

* the magic number, signfying that this OP_RETURN data is a deletion statement [16 bytes]
* the transaction id of Transaction 2, which is the transaction to be operated on [32 bytes] 
* 1, which is the number of data segments to be deleted [2 bytes] 
* the byte offset of the first byte to be deleted [4 bytes] 
* the number of bytes to be deleted [4 bytes] 
* the updated transaction hash for Transaction 2 [32 bytes] 
* original hash value for the single signature [32 bytes] 
* updated hash value for the single signature [32 bytes]

Validation of Transaction 3 works as follows. First, it must be a valid transaction under the general rules, considering the OP_RETURN data as arbitrary meaningless data. Note that this is all legacy nodes will see, and this is what allows this design to operate as a soft fork.

Then, in addition, because Transaction 3's OP_RETURN data payload starts with the magic number that indicates a deletion statement, the data payload must satisfy three further rules. First, it must be well formed, according to the format from the specification for deletion statements. Second, it must delete only safe data from Transaction 2. For example, it could not overwrite the number of satoshis sent to each UTXO. Third, and lastly, the three hash values at the end of the payload must precisely match what actually happens when the deletion is applied to the data in Transaction 2.

That is, when the specified bytes in Transaction 2 are all overwritten with 0's, the new hash for Transaction 2 must precisely match the hash written to the payload. 

Next, the second hash, which is the old signature hash value, must precisely match the unaltered Yransaction 2, as computed when verifying the signature.

Lastly, the third hash value is calculated as per the second hash value, but using the altered Transaction 2 data after the deletion is applied, instead of the original Transaction 2 data.

If any of the hash values written to the Transaction 3 data payload do not precisely match the valid values calculated, then the transaction is invalid, and must be rejected.

These hashes are written into the blockchain and committed with accumulated proof of work over time. While in theory any possible alteration *could* act on Transaction 2, most are trivially unsafe, and would never be accepted by nodes. This is, foundationally, the very purpose of signatures. Only this deletion, validated and committed with accumulated proof of work, is verifiably safe to apply. It brings the total number of valid possibilities for the content of Transaction 2 from one (only the original signed value) to two (this modified, validated value - plus the original signed value).

**Stage 3 - a node chooses to delete this content**

It is relatively straightforward for a node to delete this objectionable content. First, they come to the conclusion that this data in Transaction 2 is objectionable content. This first step is not in scope for this design, but this could be done either through word of mouth (or the online equivalents), or through a trusted subscription to a third party service, or simply by viewing the content directly and finding it to be objectionable. Next, they become aware that there is a deletion available for this objectionable content. This second step is somewhat trivial, because it is written explicitly to the blockchain, in Transaction 3.

The third step is to validate the deletion statement in Transaction 3, confirming the validity as detailed earlier. In this case, it means applying the deletion to a temporary copy of Transaction 2, then checking that the following conditions hold: 

* rule 1:  it is a well formed deletion statement
* rule 2: the bytes it specifies for deletion in Transaction 2 are safe to delete (eg arbitrary data) 
* rule 3:
* the first hash written must match the hash of the updated Transaction 2
* the second hash written must match the original unaltered transaction data used in the transaction signature
* the third hash written must match the altered transaction data used in the transaction signature, after the deletion is applied to Transaction 2

If all of these conditions are met, then the node proceeds to overwrite the specified bytes (from the specified byte offset, to the specified number of following bytes) with all 0's.

Now and in future, when the node wants to verify that this transaction is valid, it will need to remember that this deletion has been applied. When calculating the transaction hash (either as the transaction id, or for input into Block 900,000's Merkel tree), the node should calculate the hash of the updated block, and then compare that to the value for the updated block hash written into the deletion statement in Transaction 3. If (and only if) the hash result matches, then the original hash can be used, as if the block had in fact hashed to that value.

When verifying the signature, a similar process is used. Relevant data is hashed to create the input to signature verification. If (and only if) it matches the third hash from the deletion statement payload in Transaction 3, then the original hash value (the second hash from the payload) can be swapped in and used for signature verification.