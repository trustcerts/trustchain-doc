# Getting Started

## Installation

### System requirement
The software is running inside docker containers so each operating system that is able to run docker should be fine.

Each node requires about 2 GB RAM and 2 CPUs. More detailed values will follow in the future. A dedicated machine is not required, but since there are multiple timeouts implemented in the algorithms, a machine will low performance will not be able to participate in e.g. the consensus.

### Docker and Docker-Compose
A node is a construct of microservices that are packed up in containers so docker and docker compose have to be installed.

### Adding a reverse proxy (optional)
You can add a reverse proxy if you prefer to address your node by https with a domain. The reverse proxy will handle any incoming requests and route them to the correct docker container.
<!-- TODO: insert image how the proxy works -->

There are two different possibilities to expose the services: by hostname or by port. Each node of the network can decide witch way it prefers. Changing the method is possible during production mode.

#### Expose by port
We do not recommend to run a node by port in production mode without tls. Right now most of the traffic is only signed with public keys, but it is not encrypted.

<!-- TODO: How do you do this?-->

#### Expose by hostname
Make sure to open Port 80 and 443 so the reverse proxy can redirect incoming traffic on port 80. Check if 
[CAA](https://letsencrypt.org/docs/caa/) could cause some problems. If you run multiple nodes on one machine only one 
reverse proxy is required.

Let's encrypt will also make sure that the traffic over HTTP ans WS is encrypted. To set up 
the proxy git has to be installed. We recommend to use this [project](https://github.com/evertramos/nginx-proxy-automation) with the network called proxy.

<!-- TODO: how is this done? -->
By using the hostname method you have to configure your DNS. You can either route by a wildcard or add every hostname
manually. No service is listening on the fully qualified domain name so only `*.example.com` is required. If you can not
use wildcard, you have to define these routes:
| Service | Description | Default |
| :--- | ----------- | ---------- |
| api | to interact with the node | _api_ |
| node | only required if you run a validator | _node_ |
| prometheus | only required when you want to expose your prometheus instance to the public | _prometheus_ |
| loki | only required when you want to expose your loki to the public | _loki_ |
| grafana | only required if you want to monitor your node directly on the machine with public access | _grafana_ |

Nodes in one network do not have to choose the same names for the services. Each node will tell the other nodes how it can be reached.



<!-- ----------------------CONFIGURATION CHAPTER----------------------  -->
## Configuration

### Configuration of the .env file 
The .env file includes all variables that are required to run the node. An example .env file can be found in the
`configs` directory. Keep in mind that it includes some secrets so be 
careful where you store it! In the future trustchain nodes will be able to run in kubernetes or docker swarm so secrets
can be securely mounted into the containers.

Some variables inform about including a yaml file. These files have to be included in the docker-compose call that is defined in the bash script.

<!-- TODO add a column with the default value that is defined in the docker compose files -->
| Variable | Description |
| :--- | ----------- |
| COMPOSE_PROJECT_NAME      | give your containers a better prefix instead of the folder where you are right now. Be careful since a project name has to be unique so a node will not be overwritten.       |
| NODE_TYPE   | there are three different types of nodes in the system:<br /><br />  **validator**: is able to be part in the consensus algorithm to generate blocks. Can generate or revoke certificates for gateways or observers. Running a validator means you have to expose your network service to the public so other nodes can establish a connection with it. <br /><br />**gateway**: is able to generate or revoke certificates for clients and will send new transactions from clients to the validators. <br /><br />**observer**: will return hash signatures and certificates. Not able to write to the chain. |
| RESTART      | Choose the docker restart policy. See Docker [restart policy](https://docs.docker.com/config/containers/start-containers-automatically/#use-a-restart-policy)       |
| IDENTIFIER      | The unique identifier of a node in the network. Nodes with the same identifier will not be able to establish a connection. It's included in the signed certificate for your keypair so pick a name everyone can understand.       |
| IMAGE_TAG      | Image tag that should be used for all containers. latest is the current one to use.|
| DOMAIN      | Qualified domain name of the node. (Services will be attached with a port or a subdomain to be accessible. You can use an IP adress and expose the services by port, but then the traffic will not be secured with TLS.)       |
| NODE_SECRET      | Secret to protect admin endpoints of the node. It will be loaded when the node starts, so it can be changed in the future. This code is required when you want to communicate with endpoints that are exlusive for administration.       |
| LOG_LEVEL | Defines the log level in the docker log, default value is info. debug should be used with caution since it produces a lot of data. |
| DID_NETWORK | the subnamespace of the did trust method for this network like `tc:dev` with will resolve all dids beginning with `did:trust:tc:dev` |
| HTTP_PORT | External port where the http service is exposed. <br /> Include `http/docker-compose.port.yml` |
| HTTP_HOSTNAME | External hostname where the http service is exposed. <br />Include `http/docker-compose.port.yml` |
| DATABASE_PORT | External port where the database service is exposed. <br /> Include `database/docker-compose.port.yml` |
| PROMETHEUS_PORT | External port where the prometheus service is exposed.<br />Include `prometheus/docker-compose.port.yml` |
| PROMETHEUS_HOSTNAME | External hostname where the prometheus service is exposed.<br />Include `prometheus/docker-compose.proxy.yml` |
| GRAFANA_PORT | External port where the grafana service is exposed.<br />Include `grafana/docker-compose.port.yml` |
| GRAFANA_HOSTNAME | External hostname where the grafana service is exposed.<br /> Include `grafana/docker-compose.proxy.yml` |

<!-- ----------------------Validator specific----------------------  -->
### Validator Configuration
Only required if you run a validator.

Include `validator/docker-compose.yml`

| Variable | Description |
| :--- | ----------- |
| VALIDATOR_MIN | Amount of validators that are required to run the consensus. (Minimum is 2 so there is one proposer and one validator.) |
| NETWORK_PORT | External port where the network service is exposed.<br />Include `network/docker-compose.port.yml` |
| NETWORK_HOSTNAME | External hostname where the network service is exposed.<br /> Include `network/docker-compose.port.yml` |


<!-- ----------------------Development specific----------------------  -->
### Development Configuration
Only required if you work with the source code of the nodes.

Include `docker-compose.dev.yml`

| Variable | Description |
| :--- | ----------- |
|COMPILED_PATH|Path to the compiled source code, so the nodes can restart themselves if they detect new code changes.|
|RESET|If set to true the node can be reset. Never use it in production!|

<!-- ----------------------Configration of the shell script ----------------------  -->
### Configuration of the shell script

#### Shell script for validators
After setting up the .env files the shell scripts have to be configured. The normal input for a validator looks like this:
```shell script
#!/bin/bash
NODE_PATH=../nodes
ENV=validator

docker-compose \
  -f $NODE_PATH/docker-compose.yml \
  -f $NODE_PATH/http/docker-compose.proxy.yml \
  -f $NODE_PATH/validator/docker-compose.yml \
  -f $NODE_PATH/validator/docker-compose.proxy.yml \
  --env-file .$ENV.env \
  $@
```
`NODE_PATH` is pointing to the folder of the yml files from the repository. You don't have to change it when the shell
script is stored in the recommended `config` folder. The `ENV` is the name of the `.env` file that should follow the 
`.$ENV.env` syntax. 

The minimum setup for a validator requires 4 yml files:
- `docker-compose.yml`: defining the base structure of a node
- `http/docker-compose.proxy.yml`: exposing the api of the node by hostname
- `validator/docker-compose.yml`: configuration of the services to run as a validator
- `validator/docker-compose.proxy.yml`: exposing the validator node service by hostname

In the [.env documentation](getting_started.md#Configuration) each variable explains which yml file has to be appended. 
The order of the yml files are important because old values will be overwritten.

If you want to add another tool e.g. grafana, you have to include the corresponding yml files. For grafana it would
look like this:

```shell script
#!/bin/bash
NODE_PATH=../nodes
ENV=validator

docker-compose \
  -f $NODE_PATH/docker-compose.yml \
  -f $NODE_PATH/grafana/docker-compose.yml \
  -f $NODE_PATH/grafana/docker-compose.proxy.yml \
  -f $NODE_PATH/http/docker-compose.proxy.yml \
  -f $NODE_PATH/validator/docker-compose.yml \
  -f $NODE_PATH/validator/docker-compose.proxy.yml \
  --env-file .$ENV.env \
  $@
```

You can make the shell script executable or call it with `bash`.

#### Shell script for gateways
After setting up the .env the shell script has to be configured. The normal input for a gateway looks like this:

**Gateway**
```shell script
#!/bin/bash
NODE_PATH=../nodes
ENV=gateway

docker-compose \
  -f $NODE_PATH/docker-compose.yml \
  -f $NODE_PATH/http/docker-compose.proxy.yml \
  -f $NODE_PATH/gateway/docker-compose.yml \
  --env-file .$ENV.env \
  $@
```
`NODE_PATH` is pointing to the folder of the yml files from the repository. You don't have to change it when the shell
script is stored in the recommended `config` folder. The `ENV` is the name of the `.env` file that should follow the 
`.$ENV.env` syntax.

The minimum setup for a gateway requires 3 yml files:
- `docker-compose.yml`: defining the base structure of a node
- `http/docker-compose.proxy.yml`: exposing the api of the node by hostname
- `gateway/docker-compose.yml`: configuration of the services to run as an gateway

In the  [configuration of the env file](getting_started.md#configuration-of-the-env-file) each variable explains which yml file has to be appended. The order of the yml files are
important because old values will be overwritten.

You can make the shell script executable or call it with `bash` in the next steps.


#### Shell script for observers

After setting up the .env the shell script has to be configured. The normal input for an observer looks like this: 
```shell script
#!/bin/bash
NODE_PATH=../nodes
ENV=observer

docker-compose \
  -f $NODE_PATH/docker-compose.yml \
  -f $NODE_PATH/http/docker-compose.proxy.yml \
  -f $NODE_PATH/observer/docker-compose.yml \
  --env-file .$ENV.env \
  $@
```
`NODE_PATH` is pointing to the folder of the yml files from the repository. You don't have to change it when the shell
script is stored in the recommended `config` folder. The `ENV` is the name of the `.env` file that should follow the 
`.$ENV.env` syntax.

The minimum setup for an observer requires 3 yml files:
- `docker-compose.yml`: defining the base structure of a node
- `http/docker-compose.proxy.yml`: exposing the api of the node by hostname
- `observer/docker-compose.yml`: configuration of the services to run as an observer

In the  [configuration of the env file](getting_started.md#configuration-of-the-env-file)  each variable explains which yml file has to be appended. The order of the yml files is
important because old values will be overwritten.

You can make the shell script executable or call it with `bash` in the next steps.


<!-- ----------------------SETUP CHAPTER----------------------  -->
## Setup
This chapter describes, how to set up the layers with the corresponding nodes. 

### Layer one

Layer one is the base layer of the network. It contains the validators. The validators are the nodes responsible for the
consensus. 

#### Requirements
**1. Correct configuration**  
The first step is always the configuration of the nodes. See chapter ["Configuration"](getting_started.md#configuration-of-the-env-file) 
for the explanation on how to use the .env file and how to configure the shell scripts. As layer one is the layer of
the consensus, make sure to configure everything correctly. 

**2. Pulling the latest image**  
Make sure you are using the latest images:
```shell script
bash validator.sh pull
```

#### Start node
Start the node in the detached mode:
```shell script
bash validator.sh up -d
```
On the first run docker-compose will also create the required networks and the docker volumes for persisted data.

#### Validate start
<!--
TODO: HOW TO VALIDATE?
To validate that all services are running you can check the logs of the running containers. Depending on the system's
resources the startup can take some seconds.

```shell script
bash validator.sh logs -f network
```
The last line should look similar to this one:
```
network_1     | 2020-11-04 09:38:19.446  (Connection) node.validator1.example.com seems to be available
network_1     | 2020-11-04 09:38:19.452  (P2PService) key isn't signed yet
```

The first line shows that the node can access itself by its hostname. This can take some seconds since the proxy has to
generate a tls certificate and update the router settings. The second line is the result of a missing root certificate
that will be generated in the next steps.

All other services (`http`, `parse`, `persist`, `wallet`) should generate a log ending like this:
```
http_1        | 2020-11-04 09:38:20.960  (undefined) Nest application successfully started
```
-->
#### Generate root certificate
When all validators are online, one of them has to start the generation process of a [root certificate](). 
A curl request to the http endpoint will inform the node which validators should be included:
- The request has to be fired against the http endpoint, not the node service
- The node secret has to be passed as an authorization token
- All node endpoints have to be passed as a JSON string-array

```shell script
curl -X POST "https://api.validator1.example.com/root-cert/create" \
-H "accept: */*" -H "Authorization: Bearer ${NODE_SECRET}" -H "Content-Type: application/json" \
-d "[\"node.validator1.example.com\", \"node.validator2.example.com\", \"node.validator3.example.com\", \"node.validator4.example.com\"]"
```

The http endpoint will coordinate the public key exchanges to generate a root certificate. After this the nodes will connect to
each other to build up a mashed p2p network. The genesis block will be built including a transaction that defines the
root certificate. The response from the curl request is the transaction's hash.

After this step the definition of layer one is complete the network is ready to build [layer two](getting_started.md#layer-two).

#### Updating layer one
There are several reasons to update the first layer:
- a new validator should join the network
- an existing validator should be removed
- the public key of a validator changes

This can be done by sending the curl request to generate a new root certificate that will replace the current one.




### Layer two
In this step the second layer for a trustchain network is set up. It contains the gateways and observers, which allow 
the communication with the blockchain from the outside world. 

#### Requirements
**1. Correct configuration**  
To start the layer properly, you have to configure the nodes correctly, see [the configuration chapter](getting_started.md#Configuration)
for information on how this is done. 

**2. Pulling the latest image**  
Before you start the node make sure you are using the latest images (where `node.sh` is the shell script of your node):
```shell script
bash node.sh pull
```

#### Start nodes
Start the node in the detached mode:
```shell script
bash node.sh up -d
```

On the first run docker-compose will also create the required networks and the docker volumes for persisted data.

#### Validate startup
A get request to the http endpoint like `curl api.example.com` with a 200 http code response will tell you if your node is ready. Since the http service is the last service that will initialized it can take some seconds. Using the proxy will increase the start procedure since let's encrypt has to generate the certificates first.

#### Invitations
A member of the second layer needs an invitation from the first layer. 

##### Create invitation
To add a new member to layer two, it needs an invitation token. This token acts like a secret. A node of the layer above 
(i.e. a validator from layer one) can generate an invitation. 

An invitation token can be generated by calling the http endpoint of a validator:
```shell script
curl -X POST "https://api.validator1.example.com/invite" \
-H "accept: */*" -H "Authorization: Bearer ${NODE_TOKEN}" -H "Content-Type: application/json" \
-d "{\"id\":\"${id}\",\"secret\":\"${SECRET}\",\"name\":\"Gateway-Trust\",\"role\":\"gateway\",\"force\":false}"
```
The endpoint is protected by the node secret so only the admin of a node can create or delete invite tokens for the node.
The body has to be json formatted.
| Key | Description | Required |
| --- | ----------- | :--------: |
| id | id of the new member's did. If not passed the validator will generate a the id | |
| secret | it's possible to define a secret that is stored in the invitation object. If nothing is passed the node will generate one. | |
| name | a human readable name that will be stored only on this node, not inside the blockchain. The validator will add a service endpoint to the new did that will response with the name | x |
| role | the privileges that a did should have. A validator can generate gateways or observers | x |
| force | if set to true a new secret can be set for an existing did | |
##### Re-Invite a user

A member of the second or third layer can lose access to the private key connected with his identity.
To get access back, a member from the next layer can generate an invitation token for the identifier and `force` it. This will
allow the member to sign a new keypair and bind it to the existing identifier. The old key will be revoked automatically
when the the key got registered on the blockchain. It cannot be used for new signatures but all old signature will be
valid until they are revoked by the new keypair.


##### Receive invitation token
The invitation token has to be passed to the http service:

```shell script
curl -X POST "https://api.gateway1.example.com/init" \
-H "accept: */*" -H "Authorization: Bearer ${NODE_SECRET}" -H "Content-Type: application/json" \
-d "{\"id\":${ID},\"secret\":\"${INVITE}\",\"url\":\"${HTTP_VALIDATOR_URL}\"}"
```
- the `id` of the did from the invite step
- the `node secret` protects the endpoint and that was set in the .env file
- the generated `invite` code has to be send in the body
- `url` needs the address of the node which generated the invite token

The node will receive its certificate including its signed key and connect to the network knowing it is a valid member
with a singed key. After syncing up with the network the node is ready to interact with [layer three](getting_started.md#layer-three).



### Layer three

Layer three members are no blockchain system nodes, but interact with the blockchain via layer two. The layer three nodes
are the clients. 
Every client can send read requests to one of the observers to get parsed information from the blockchain,
but only members with a signed key can send write requests to a gateway, so the transaction will be added to the blockchain.


#### Invitation
A client that wants to write to the blockchain needs a signed key. To get this key it requires an invitation token from
a gateway. This token will be sent to the gateway http endpoint to get the own key signed.

##### Create invitation
To add a new member to layer three, it needs an invitation token. This token acts like a secret. A node of the layer above 
(i.e. a gateway from layer two) can generate an invitation. 
An invitation token can be generated by calling the http endpoint of a gateway:
```shell script
curl -X POST "https://api.gateway1.example.com/invite" \
-H "accept: */*" -H "Authorization: Bearer ${NODE_TOKEN}" -H "Content-Type: application/json" \
-d "{\"id\":\"${id}\",\"secret\":\"${SECRET}\",\"name\":\"Gateway-Trust\",\"role\":\"gateway\",\"force\":false}"
```
The endpoint is protected by the node secret so only the admin of a node can create or delete invite tokens for the node.
The body has to be json formatted.
| Key | Description | Required |
| --- | ----------- | :--------: |
| id | id of the new member's did. If not passed the gateway will generate a the id | |
| secret | it's possible to define a secret that is stored in the invitation object. If nothing is passed the node will generate one. | |
| name | a human readable name that will be stored only on this node, not inside the blockchain. The gateway will add a service endpoint to the new did that will response with the name | x |
| role | the privileges that a did should have. A gateway can generate clients | x |
| force | if set to true a new secret can be set for an existing did | |

##### Re-Invite a user

A member of the second or third layer can lose access to the private key connected with his identity.
To get access back, a member from the next layer can generate an invitation token for the identifier and `force` it. This will
allow the member to sign a new keypair and bind it to the existing identifier. The old key will be revoked automatically
when the the key got registered on the blockchain. It cannot be used for new signatures but all old signature will be
valid until they are revoked by the new keypair.


##### Using the invitation token
The invitation token has to be passed to the http service to get a valid did:

```shell script
curl -X 'POST' \
  'https://api.gateway1.example.com/did/create' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "identifier": "did:trust:tc:prod:id:D3fc9Yq1djFwQuwV4M3vTx",
  "secret": "cryptoSecretKey",
  "publicKey": {
    "key_ops": [
      "verify"
    ],
    "kty": [
      "RSA"
    ],
    "n": "zvbICKrRLlnDWuTXRwWV9nsaiYCaLCNiNF1WmbsWFXHbT9AhyYDbIh_KLI0y5vpYTIfdneRYeNWjkldzZ_J3xZDJ9zUdxHZGXUa9j-NHInmKsYVPDhTYTbTEmDQ2COGKv26klNkyNFKS1Sap8Q7y3jyZQvV4fVd4KynpkJirpDRoDS4jeqPrZKjXQdLxmLBnBiUuD7V2phy5PFBxTsnX6wkZiWJKRRzq6CnavlgeieLgCUrsD6fmmV7B5MtJJ-fdrLxXFXXDaD9d82ZFmM24dqaMkwLvMt22xEaz27WoYftUJJIbYGNec4qTTzacEv_YcYgR8YIXQSpnviXsZ0mqPw",
    "e": "AQAB",
    "alg": "RS256"
  }
}'
```
- the `identifier` of the did document from the invite request
- the `secret` from the invite request
- a `publicKey` that is registered to update the did document