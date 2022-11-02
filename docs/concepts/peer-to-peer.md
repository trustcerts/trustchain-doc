# Peer-to-peer (P2P) protocol

## Summary

The peer-to-peer protocol describes how nodes connect to other nodes. It also defines how new nodes are added to the network or existing nodes are removed from the network.

<!-- Synch von KNoten fehlt noch-->

## Motivation

The blockchain network consists of multiple nodes that communicate with each other. We need to establish a procedure for how the nodes initially connect to each other and how new nodes can be added or existing nodes can be removed once the network is established, and to ensure that all nodes have a consistent view of the network.

## Validator connecting to another validator

### Validators connecting flow
![Validators connecting flow](https://trustcerts.github.io/trustchain-doc/diagrams/p2p_ablauf.drawio.png)

```mermaid
  stateDiagram 
    
    [*] --> Is_node_available 
    state if_state <<choice>>
    Is_node_available --> if_state
    if_state --> Request_the_nodes_identifier_and_send_own_identifier: yes
    if_state --> Is_maxRequest_reached?: no
    Is_maxRequest_reached? --> Increase_maxRequest: no
    Is_maxRequest_reached? --> [*]: yes
    Increase_maxRequest --> Is_node_available
    
    state Already_connected_to_validator2?
    state Already_connected <<choice>>

    Request_the_nodes_identifier_and_send_own_identifier --> Already_connected_to_validator2?
    Already_connected_to_validator2? --> Already_connected
    Already_connected --> [*]: yes
    Already_connected --> Is_validator2_a_validator?: no
    
    state Is_validator <<choice>>
    Is_validator2_a_validator? --> Is_validator
    Is_validator --> Is_validator1_compared_tovalidator2>0?: yes

    state Val1_compare_val2 <<choice>>
    Is_validator1_compared_tovalidator2>0? --> Val1_compare_val2
    
    Val1_compare_val2 --> Push_validator2_to_connection_set: yes
    Val1_compare_val2 --> [*]: no 
    
    Is_validator --> Push_validator2_to_connection_set: no
    Push_validator2_to_connection_set --> Open_websocket_connection_to_validator2: pushed
    Open_websocket_connection_to_validator2 --> Set_listener_for_challenge_from_validator2: connected
    Set_listener_for_challenge_from_validator2 --> Send_challenge_to_validator2:set
    Send_challenge_to_validator2 --> Wait_for_response
    Wait_for_response --> Is_response_valid?: received
    Is_response_valid? --> valid
    
    state valid <<choice>>
    valid --> Add_connection: yes
    valid --> Diconnect: no 
    
    Add_connection --> [*]
    Diconnect --> [*]
    
```


The peer-to-peer (P2P) protocol first checks whether the validator or node to which a connection is to be established is still available. If not, it will be checked if the number of requests has reached the maximum. If the number of request reached the maximum, the connection will be closed, if not the number of the request will be increased by 1 and and it is checked one more time if the node is available.

In case that the targeted node is available the node identifier will be requested and the requesting node will sent it's identifier. As a result the node identifier of the requested node is sent back. 

With the identifier of the requested node a validator can check if a connection to the requested node is already established. If there is already a connection, nothing more needs to be done. 

Otherwise, the next step is to check whether the node to which a connection is to be established is a validator. In case the node is a valid validator it is agreed which node starts the connection establishment by a simple string compare so that only one websocket connection is created and the listeners are set only once. 

If the requested node is not a validator it is pushed to the connection set and a websocket is open to the node and the connection will established. In the next step a listener is set and a challenge is sent to the requested node. Is the response is valid the connection is added, if not the connection will be disconnected. 

## Validators connecting sequence
![Validators connecting sequence](https://trustcerts.github.io/trustchain-doc/diagrams/sequence_p2p.drawio.png)

<!-- In case that the targeted node is available the node identifier will be requested and the node identifier is sent back. -->

```mermaid
  sequenceDiagram
    autonumber
    participant V1 as Validator1
    participant V2 as Validator2
    V1->>V2: available?
    V2->>V1: yes
    V1->>V2: identifier? mine: 'Validator1'
    V2->>V1: mine: 'Validator2'
    V1->>V1: Validate connection request
    V2->>V2: Validate connection request
    V1->>V2: open websocket
    V2->>V1: connect event
    V1->>V1: Start listening for challenge
    V1->>V2: challenge
    V2->>V1: challenge
    V1->>V2: response
    V1->>V1: Add connection
    V2->>V1: response
    V2->>V2: Add connection
```



## Synchronisation of Data

## Network adding new nodes

## Network removing existing nodes
<!-- How to remove existing nodes from the network-->
<!-- -> in Code gucken oder Mirko fragen-->

## What happens if one node fails

## Security

### Trustanchor
##### Node to node
-Genesis file with list of validators and their respective public keys
##### Client to node

## Open Questions
























