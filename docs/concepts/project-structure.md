# Project structure

## apps

This folder contains the code for the different microservices.

It has the following applications:

| Application | Description |
| ------- | -------- |
| http-gateway | Gateway specific application to communicate externally. |
| http-observer | Observer specific application to communicate externally. |
| http-validator | Validator specific application to communicate externally. |
| network-gateway | Gateway specific application to communicate within the blockchain network. |
| network-observer | Observer specific application to communicate within the blockchain network. |
| network-validator | Validator specific application to communicate within the blockchain network. |
| parse | Application to handle and manage the keys. |
| persist | Application to write transactions to the blockchain. |
| shared | Not a standalone application but containing the code that several to all applications use. |
| wallet | Application to handle and manage the keys. |

## build

The 'build' folder contains the necessary components to build the project (in docker).

## libs

The 'libs' directory contains the libaries that the nodes are using. The libaries are not specific for one node or service but are used by several to all of them. 

There are the following libaries:

| Libary | Description |
| -------- | -------- |
| blockchain | Has the shared code used to work with the blockchain (e.g. Block, Transaction, Signature...)| 
| clients | Contains the code for the clients, that are used to interact with the microservices. |
| config | Contains the code for configuration of the applications. |
| invite | This libary is for inviting the different nodes or clients to the network. |
| p2-p | Has the shared code for connecting, sending and receiving information in the network. |
| transactions | Contains the code for handling the different transaction types. |

### transactions

This libary contains the different transactions types. In the source folder is a folder for each transaction type (e.g. did-schema, did-id...) and a 'transactions' directory, which stores the code that every transaction type needs. 

Each transaction type has the following folders:

| Folder title | Description |
| ----------- | ----------- |
| cached | Contains the components for reading existing data. |
| db | Contains the module to create the database connection for the corresponding transaction type. |
| dto | Contains the data transfer objects that are used to send and receive transactions. |
| gateway | Contains the components that are used for handling transactions by a gateway. |
| observer | Contains the components that are used for handling transactions by an observer. |
| parse | Contains the components for parsing transactions, so they can be stored in the database. |
| schemas | Contains the schemas for storing the transactions in the database |
| validation | Contains the components to validate the transactions on the blockchain. |

## test

This folder contains additional code for testing. (Additional to the spec files that are always where the coresponding service is.)

## tools

This folder contains several scripts to simplify some steps in the project.