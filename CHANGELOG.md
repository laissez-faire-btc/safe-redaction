1. Original design posted to the Bitcoin Developer Mailing List. This proposed to track the internal state of the hashing function.
2. Updated to switch from a deletion model to a redaction model, now simply recording "before and after" hash values for the transaction hash, and each affected signature.
3. Published to GitHub. 
4. Included possible hard fork and soft fork option. 
5. Removed the hard fork option, and provided additional details on the more realistic soft fork option, including data format. 
6. Included a hypothetical worked example, including a simple deletion, that is validated and accepted by the network.
7. Public domain licensed. No Rights Reserved.
8. [TODO] Changed terminology from "deletion" to "redaction".