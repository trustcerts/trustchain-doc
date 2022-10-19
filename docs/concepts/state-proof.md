# State Proof

- Abschnitt Terminology: Welche davon aufnehmen? ✅
- Abschnitte Verification process & Catchup müssen noch gemeinsam mit Mirko erarbeitet werden
- Bei unresolved questions: mismatching networks?


- Todos:
    - Anpassen Stellen wo Merkle Tree über Transaktionen erwähnt wird (weil wegfällt)
    - Wie damit umgehen, dass es Ledger State & Network State gibt, bzw. auf welche Art dieses Dokument hier anpassen? (extra Kapitel für je Ledger & Network State machen ja wenig Sinn)


13.10.2022 Call mit Mirko

https://static.miro-apps.com/gmeet/board/?mtt=2v64kl7qkiyp&meet_origin=&boardId=uXjVPOdrGOc%3D&boardAccessToken=8FWopcVpIJJOULbfzvvweN27j1KVerr7

Notizen:
- Ablauf Root of trust aka Genesis fetchen (z.B. GitHub, in der SDK)
    - ist jedenfalls "geregelt", da kein weiterer Klärungsbedarf
    - in Genesis sind halbwegs aktuelle Infos, mit dem Ziel, zum Network zu connecten (IPs von Gateways/Observer als auch PubKeys Validatoren)
        - ab dann kann der Rest des Pool Ledgers synchronisiert werden
- Pool Ledger:
    - aka "node resources"
    - Kein Merkle Tree, sondern Hash von allen DID Docs der Validatoren (ohne Gateway/Observer)
    - noch technisch umzusetzen: Kanonisierung Transaktionsreihenfolge für Hash
        
- Wer macht Validierung State Proof?
    - ✅ Szenario 1: Resolver fetcht, Resolver State Proof
    - ❌ Szenario 2: Resolver fetcht, Client State Proof
        - schlecht, weil Client sowieso mit BC kommunizieren muss, also kann Client auch direkt das DID Doc anfragen statt Umweg über Resolver
    - ✅ Szenario 3: Client fetcht, Client State Proof
        

## Summary

The state of the trustchain ledger is stored in a Patricia Merkle trie and represents the state of all parsed DID resources. By signing the state root along with a timestamp by multiple validators, it is possible to verify that a particular DID resource is included in the state trie and is therefore part of the ledger.

The signed state is stored periodically and allows to work with verified previous states of the ledger.


## Motivation

To validate verifiable credentials (VC), the verifier may not need to request the issuer's current DID documents, but rather those at the time the credential was issued, as the issuer's signing keys may have changed over time.

This problem occurs because DID documents are mutable objects that can change their state over time (e.g., due to key rotation). When the object is immutable, it is trivial for the ledger to provide a verifiable response. However, if the object is mutable, there is no trivial way for the ledger to provide a verifiable response of an object's state in the past.
State proof allows for a mechanism to validate every version in the history of an object.

Since not all verifiers operate their own observer node that could provide them with the information directly, they must instead rely on external observer nodes to provide them with valid DID documents. State proof is a mechanism that enables trust in the observer's response by cryptographically verifying that the returned DID document was included in the ledger state at the requested time.

One could use the BFT approach to verify the received information, but this approach requires querying multiple observer nodes and lacks cryptographic verification. With state proof, it is sufficient to query only one external observer node, since the returned state proof is signed by all validators of the network.


## Structure of the state proof

The structure of a state proof is similar to an ordinary block header of the trustchain. A client needs all necessary information to verify a state proof. Therefore the state proof consists of signatures of the validators, the ledgerStateRootHash to assemle the DID resource and the timestamp of the block creation.

A client needs the signatures to validate the authenticity of the information sent by the trustchain/node. The ledgerstateRootHash and the timestamp can be used to prove that a specific DID document exists at the given time. The timestamp would also proof the freshness of the timestamp.



A state proof consists of following elements:
* Signatures


* The ledgerStateRootHash
    * Which is signed together with a timestamp as part of the block header
* The hashes of sibling nodes leading to the root hash

<!--

A block consists of following elements:
* signatures
    * A list of signatures of all validators 
* index
    * The index of the block, incremental
* previousHash
    * The hash of the previous block, for chaining the blocks
* hash
    * The hash of the transaction list (no need for a merkle tree since there is no case to present just one transaction of a block)
* timestamp
    * The timestamp of the creation to prove the freshness
* transactions
    * A list of all transactions in the block
* version
    * The version number of the block
* ledgerStateRootHash
    * The root hash of the Merkle Patricia trie of all DID resources on the ledger
* networkStateHash
    * A hash of all the DID documents of the network nodes (gateways, observers, validators)
    
    
Example of a block:
```
{
    signatures: ["abc...def", "dhi...jkl", "5fa...2ef"],
    index: 42,
    previousHash: "4a5...12a",
    hash: "f54..627",
    timestamp: 599616123,
    transactions: [...],
    version: 1,
    ledgerStateRootHash:,
    networkStateHash:
}
```
-->


<!--
Kontext: Für Implementierung von State Proof in Blockchain wird Block-Header erweitert, dann wird der LedgerStateRootHash & NetworkStateHash im Teil des Block-Signatur-Prozesses mitsigniert (siehe MiroBoard)
-->

## Terminology
- State
    - The state of a system at a specific time
- State root
    - The root element of a Merkle tree, which is signed by the validators to be accepted
- State proof
    - A proof that a given element is part of a Merkle Patricia trie whose root has been signed by multiple validators
- Consensus
    - The consensus is the process in which the validator nodes of the network decide about adding a new block to the blockchain, see [consensus](https://trustcerts.github.io/trustchain-doc/#/./concepts/consens)
- ❓ Block signature
<!--
    - von was wird die Signatur gemacht, Hashes Inhalt keine Ahnung
    - überall die gleiche Signatur (manchmal DID docs, manchmal Transactions, manchmal Blöcke, manchmal State Root), was wird wo signiert?
        - das einmal auflisten vllt.?
    - Siehe Fragen Abschnitt Block Signature approach
-->
- Transactions
    - A DID document is assembled by transactions, where each transactions can modify the DID document such as by adding, rotating or removing a key. These transactions are combined in a block and signed during the consensus of the blockchain. Additionally, there is a signed Merkle tree over all transactions on the blockchain.
- Merkle tree
    - A Merkle tree / hash tree is a cryptographic data structure that allows efficient and secure verification of its contents, see [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree).
- Patricia Merkle Trie
    - A Patricia Merkle Trie provides a cryptographically authenticated data structure that can be used to store all (key, value) bindings, see [Patricia Merkle Trie](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/).
- BFT approach
    - The [Byzantine fault tolerance](https://en.wikipedia.org/wiki/Byzantine_fault) is used to accomplish resiliency in distributed networks, see [consensus](https://trustcerts.github.io/trustchain-doc/#/./concepts/consens).


## Freshness

State proofs are generated at regular intervals (e.g. every 5 minutes) and stored on the ledger. This allows older state proofs to be retrieved for any time in the past, and ensures that the current ledger state always has an up-to-date signature.

<!--
- "prove me that the resource is still the latest one"
    - latest version is two months old
    - prove that this is still the latest version by signing the root state with an up to date timestamp
-->

## Evaluation of using block signatures as an insufficient alternative to state proof

<!-- 
(Kontext: wie Bitcoin das macht: Beweisen, dass nur 1 gewisse TA in einem Block ist)
- proof das TA in MT des Blockes (also im Block) ist. Knoten zeigt Beweis, dass TA in MT ist -> Merkle Tree Proof (ich kenne Pfad der bis oben geht) 
Kommt Block wirklich von BC? Anderen Knoten fragen gucken ob der das gleiche sagt. Gewisse Wahrscheinlichkeit, aber nicht gg Man in the middle.
    - ❓ warum kann man nicht wissen ob Block von BC kommt, hat man nicht die Keys der Validatoren die das signiert haben?
        -man geht zum Block 0
    - ❓ welcher Trustanchor hat der Client?

Man hat Block 0 lokal. Blöcke sind verkettet. Beweis zeigt, dass TA aus Block n kommt. Kann ich dann mit den Blockheadern der Blöcke 1 bis n-1 errechnen.
    - ❓ Heißt nur, dass ich keine weiteren Blockheader als die vorherigen laden muss, oder was ist mit der Aussage gemeint? Man muss die Blockheader bis zu 0 laden

state ist interessant wenn Ressourcen von dezentralem System bereit gestellt werden. 

Block wurde von bestimmten Validatoren signiert. Was sind die Public Keys? Wie komme ich daran? Führen ja auch Key rotationen durch. Wie komm ich an Liste der Rotationen? Verschiedene Mechanismen: Indy hat Pool Ledger, der recht klein ist. Habe einen root of trust irgendwo verankert, dem ich vertraue, hole alle Pool Ledger TAs, kann die aktuellen ablesen. ❓ 
-->



All transactions on the ledger are protected by a Merkle tree, whose hash is signed during the consensus of the blockchain. It seems reasonable to use the block signature as a way to verify a requested DID document. This method however has several drawbacks compared to the state proof:

* A DID document is composed of several transactions. A verifier cannot verify a DID document without querying and analyzing all the transactions involved. These transactions are aggregated in blocks together with other - unrelated - transactions and these blocks are signed by the consensus. To verify a transaction, the entire block needs to be retrieved, which causes additional overhead.
* While verifying the validity of a DID document by querying the corresponding transactions from the ledger, this approach lacks the ability of temporal verification. E.g. if there are more/newer transactions on the ledger than those returned by the observer, the verifier has no way of verifying it has received **all** relevant transactions, as the incomplete transactions (sub-)set is still cryptographically valid on its own.

![](https://i.imgur.com/PNCKNfG.png)

<!--
req: all transaction for DIDDocA
 -> res: 5 transactions, each block is signed by 5 validators -> 10 different validators have signed all blocks
     -> req: get all transactions for all 10 validators

❓ Frage 21.09.: Warum steht das hier? Die Thematik dass man Validator-Keys verifizieren muss hat man ja generell, z.B. um den State Proof zu verifizieren, und nicht nur um die hier genannten Block-Signaturen zu verifizieren
```
-->

## Handling the state in the nodes

The state root is generated as part of the consensus.

1. After the [proposer](https://trustcerts.github.io/trustchain-doc/#/./concepts/consens) builds a new block, it calculates the new state root. 
2. Then the proposer's [network service](https://trustcerts.github.io/trustchain-doc/#/./concepts/architecture?id=trustchain-specific-services) sends the new block to the [parse service](https://trustcerts.github.io/trustchain-doc/#/./concepts/architecture?id=trustchain-specific-services) (which is handling the state module)
3. If the block contains new transactions that affect DID documents, the parse service assembles the resulting DID documents and updates the state.
4. Then it returns the (new) state root to the network service.
5. The proposer sends the new block to the [validators](https://trustcerts.github.io/trustchain-doc/#/./concepts/nodes?id=node-types).
6. The validators validate the transactions and the new state
7. The validators sign both the Merkle tree of the transactions and the state root, each including the current timestamp
8. The validators return the signed objects to the proposer
9. The proposer puts everything together into a new block and shares it with the network
10. All nodes (validators, [gateways & observers](https://trustcerts.github.io/trustchain-doc/#/./concepts/nodes?id=node-types)) parse the new block, updating their databases. In case of an empty block, only the state root database is updated, so it contains the newly signed state roots.

![](https://i.imgur.com/HZ8eIaZ.png)
```plantuml
Bob->Alice

```

## Verification process

The verification process describes how a verifier can validate the authenticity and integrity of a requested DID resource for a specific time using the state proof.  
To achieve this, the verifier also checks the integrity of the currently used keys of the validators.

The process can be described as follows:

1. The verifier sends a request to an observer for a DID resource for a specific time
2. The observer responds with    
    - the generated state proof (who signed the rootHash, Values to proof the element is part of the merkle pat. trie),
    - the assembled DID resource,
    - information(❓) about relevant validator transactions
4. If needed, the verifier syncs all missing transactions relevant to validator key rotations
    - All validator transactions are hashed and signed by the validators in the consensus to guarantee the freshness
    - Using this information, the verifier assembles the DID docs of the validators based on the time of the state proof
3. Using the keys of the validators at the time of the state proof, the verifier can validate the signatures of the state proof

<!--

In order to validate the state root hash, which is part of the block header, the whole block header has to be received

- Client hat nicht alle Pool-Ledger-Transaktionen parat (sondern 0-n)
- Bei jedem Request wird mitgeschickt, wie viele Transaktionen Client bereits synchronisiert hat
- Observer liefert in seiner Antwort die potenziell noch fehlenden Transaktionen mit
    - special case: falls z.B. zu viele (und Observer deswegen keine sendet), kann über eine Schnittstelle ein spezifizierter Bereich an Transaktionen separat chunkweise nachgeladen werden
        - falls Größe der Transaktionen 8MB überschreitet (konkreter Threshold wird noch definiert)

Situation: Client hat 20 Transaktionen, Netzwerk ist bei 25
a)
request: did:trust:ewhwrqlksadhökgnb

response: {
    metaData: {
        networkTransLength: 25
    }
}
Client request: give me the transaction from 21 to 25

b)
request: did:trust:ewhwrqlksadhökgnb?networkTransLength=20 // my last transaction    

response : {
    diddoc: {}
    metaData: {
    missingNetworkTrans[] //Elements 21-25 since 25 is the latest transaction manipulating the network nodes
    networkLedgerRootHash: "",
    timeStamp: "",
    networkStateSignatures: [] // The signature was made of the 
    }
}

-->

<!--
- [x] verifier requests a did resource for a specific time
- [x] observer node creates the requested state and generates the proof and the did document
- [x] verifier receives the did doc, the proof and the information about the validator transactions
- [x] syncs missing validator transactions to have all public keys of the validators available
    - [x] all validator transactions are hashed and signed by the validators in the consensus to guarantee the freshness
    - [x] assemble the did docs based on the time of the state proof
    - [✅❓] the integrity of the transaction is given by the signature in the blocks
- [x] validates the signatures of the state proof
-->



<!-- 
- kleinster Aufwand wäre: ich prüfe nur die Signaturen.
anders: ich verifiziere die Rotationen, da kann man sich aber eig die Eingiung über den Konsens sparen (wenn ich denen nicht vertraue brauche ich auch kein Konsens)
    - ❓ Was hat Rotation mit Konsens zu tun, und warum kann man sich Konsens-Einigung dann sparen?
- ich prüf nur Signatur und über state root prüfe ich ob ich wirklich auf selben proof komme. Frage ist, ob das ein deutlicher mehr Aufwand ist. wenn es 20% mehr Zeit brauch wäre es doof, gerade bei großen Catch up. Wenn BC 100k TAs hat...
-->

<!--
- Im Rahmen der State-Proof-Validierung holt der Verifier sich auch die aktuellen Validator-Keys, ausgehend von den Genesis-Validator-Keys (?)
- Warum Validator-Transaktionen hashen signen freshness? Gibt es die nicht schon in der Blockchain?
- "all validator transactions are hashed and signed by the validators in the consensus to guarantee the freshness" 
    - Ist das ein aktiver Schritt oder ist das so schon hinterlegt?
    - "are hashed" = einfaches Hashen oder wird auch ein state (proof) erstellt?
- "the integrity of the transaction is given by the signature in the blocks"
    - Warum hier wieder signature of the blocks?

[Grafik, die veranschaulicht, wie der passende State Proof geladen und mit dem geparsten Dokument abgeglichen wird?]

------------
-->

<!--
```
# Abschnitt: bzgl. Lokales Caching von Validator-Transaktionen 
# Gehört zu Observer. Gehört zu verification process
req: did doc from 1.5
    - getting all validator transactions until 1.9 to validate state proof
req: did doc from 1.3
    - validate state proof since all required validator transactions are locally available
req: did doc from 1.8
    - request missing validator transactions
```
-->

## Ledger state

The ledger state describes the state of all DID resources that are stored on the ledger and consists of the hashes of the DID resources.

❓ ist "state" hier äquivalent zu Patricia Merkle Trie?
<!--
PMT ist eine Methode/Technologie um den State zu repräsentieren
-> der gebaute PMT ist letztlich der State
-> PMT wird genutzt um den State Proof effizient durchzuführen
-->

<!--
Ledger-State: beinhaltet die States von allen Resourcen, die über den Ledger gemanaged werden

Unterschied zu Indy: Indy managed Ledger-State als Domain-Ledger (alle Resourcen, die extern abgerufen werden können) und Pool Ledger (Infos, die für den State Proof + die Interaktion mit dem Netzwerk benötigt werden)
-->

## Network state

The network state describes the state of all nodes that are part of the network and consists of all the hashes of the DID documents of the network nodes and is signed by the validators.
This has two goals:
* A client is able to know the IP addresses of available gateways and observers
* A client is able to verify the public keys of the validators, e.g. to validate the (ledger❓) state proof


<!--
Network-State: beinhaltet die States von allen Knoten, die über den Ledger gemeanaged werden. Ziel: der Client braucht alle IPs (um Gateways und Observer zu kennen) und alle Public Keys aus dem Konsens (um den Ledger State Proof zu validieren)

Network-Hash: Summe aus allen Hashes der Node-Did-Dokumente

Unterschied zu Indy: Indy managed Ledger-State als Domain-Ledger (alle Resourcen, die extern abgerufen werden können) und Pool Ledger (Infos, die für den State Proof + die Interaktion mit dem Netzwerk benötigt werden)


Subject of the client to node, node to node communication

# TODO: which information will the client receive to get more information about the network
- e.g. sync all node did transactions to receive information:
    - other observer endpoints (reduce single point of failure)
    - other gateway endpoint (reduce single point of failure)
    - validator public keys (state proof)
Open questions:
- does it make sense to add the ip addreess to the service endpoints of the nodes
- does it make sense to generate a file/transaction including the public keys of the validators for the consensu (to reduce the amount of requesting transactions. Same for observer and gateway endpoints

Indy Pool Ledger:
- all public keys. relevant for connection + state proof
- all IPs, relevant for p2p connections
-->

## Security 

<!--
Liste der bekannten nodes zu dem ein node eine verbindung hatte auf den ledger schreiben

damit Clients auf jedenfall die richtigen pub keys bekommen kann um zu

haben wir noch nicht geklärt wie der client an alle wirklichen public keys der validatoren kommt. Client müsste den node immer vertrauen dass er alle schickt
-->

## Drawbacks

- Keine? 
<!-- TODO: Validierung kann sehr rechenintensiv sein, wenn ich das auslager, muss ich diesem Dienst aber auch vertrauen -->

## Unresolved questions

* Cryptoagility: Since the state proof is persisted on the ledger, existing state proofs cannot be directly updated in case the underlying algorithm needs to be updated
* Mismatching networks: ?

<!--
- Cryptoagility: when the algorithm is outdated all state proofs are insecure. Since they are persisted in the blocks a complete rebuild has to be done
- Mismatching networks:
    - When using the universal resolver approach to get a did document it has to be guaranteed that the client and the universal resolver are using the same blockchain network to get the information
    - This can only happen when there is a namespace with two different networks
    - Can be solved when all namespaces are registered in a central place like a registry on github
-->