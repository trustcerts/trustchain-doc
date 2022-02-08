# Chain of trust
When requesting information from an external system the client has to validate the information, normally by validating a digital signature. To increase the trust level the requested information needs a chain back to the root of trust. In the PKI world a root certificate is used, in the case of a blockchain the genesis file is the perfect candidate. This file never changes so it has to be requested only once.

## Problem with huge amount of transactions
Since all DIDs are mutable there can be a huge amount of transaction that have to be parsed to assemble the DIDDocument. Besides requesting multiple transactions the client has also to validate multiple signatures. Validating the chain of trust from a client back to the genesis file increases the amount of included DIDs and therefore the amount of transaction to handle.

Of course the client is able to cache already validated signatures to reduce to speed up the process. But in the most cases in the SSI world there are a lot of different clients and every validation process should be done in a short time. Therefore the amount of requested transactions as to be reduced so the amount of validation is reduced.

### Store DIDDocs in transactions
Instead of storing the changes the complete DIDDoc will be stored in a transaction on the ledger. This reduces the amount of requests to one but is a very inefficient way of storing the data. In case of one key rotation all other keys that haven't been touched in the update are already stored in the transaction. Since blockchain should be stored for a long time the size of a transaction should be as low as possible.

### Request assembled DIDDoc from a node
To outsource the resource intensive tasks the client requests the already assembled DIDdoc from a node. In this case the client has to trust the requested node that it response with the correct DIDdoc since there is no way to validate any signatures anymore. The signatures were bound to the transaction content but not the result based on the assembled transactions.

The client could request multiple nodes, e.g. `f+1` where f is the amount of faulty nodes. But this increases the network traffic and does not allow the verification of the chain of trust. A man in the middle could manipulate all responses when there is no kind of signature validation involved.

## DidDocSignature for latest proof
To reduce the amount of requests to one the node has to pass an assembled DIDDoc based on multiple transactions. A proof signature is added by the gateway so the observer is not able to include false information:
- the client creates a transaction to manipulate its did
- the gateway validates the signature and the checks if the transaction assembled a new did doc based on the changes. If so the gateway will add a signature to the transaction which that it signs that the changes are correct. The gateway has to sign the DIDDoc state and not the client by itself. The public key to validate a signature would be included in same data that way signed, so it self signed without any root of trust.
- the validators validate the transaction changes and also the added signature from the gateway. If both are valid, the transaction will be added to the block. The validators will not trust the one gateway since this would result in a single point of failure. The validators accept the transaction with the signing of the block. This prevents the validators from singing every transaction in the block individually

Since only observers are used to requests transaction from the blockchain they have no chance to append their signature to the DIDDoc to present a faulty response. The client is now able to request the DIDdocs of the chain of trust up to the genesis file to validate the chain of trust. The genesis file is stored locally and defined as the root of trust. Therefore the client is able to handle DIDdocs that are based on a huge amount of transactions, the observer can be optimized when caching different versions of a DIDdoc.