# Project structure

## libs

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
