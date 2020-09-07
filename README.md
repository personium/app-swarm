# app-swarm  
This is working in progress notes on integrating with Swarm Bee to demonstrate how personal data store can use decentralized storage to create valued functions.  

## The Idea  
Storing and managing personal or organisation data can imply greater responsibilities:  

- On average, a user would not be able to technically sufficient running Personium for personal use  
- If need to consume the service, the data owner need to trust the hosting provider or a Personium data registry  
- The services provider need to ensure all operators would respect security procedures  
- The data owner need to find ways to migrate to another services provider with little efforts (competitiveness and interoperability)  
- The hosting proivider or Personium data registry has little to gain but taking more risks of being a provider  

Storing sensitive information on the Swarm ethereum storage network could potentially simplify these concerns:  

1. In personal data context, assume a publicly maintained network is available, the personal data will be replicated through the Swarm network without a centralised hosting provider. This potentially changes the role of Personium to a data processor and consent management role.  
2. On local/personal node initialisation, a network account and passphrase are required to be created. Both account and passphrase are unique, must be owned by the data owners. If they are lost, unencrypted data maybe recoverable if the owner has the hash reference, but encrypted data will be lost.  
3. Sensitive data can be encrypted via Swarm network, this means data encryption can be offloaded to Swarm storage layer   
4. With a high level view, encrypted data are chunked/distributed and only can be decrypted at a local node (aka, with the correct account and passphrase presents). Now this is interesting, because potentially--once data is replicated over the network-- the data owner can easily switch hosting providers (with their own account/passphrase).  
5. In the event of disaster of lost services or data, the network may provide data recovery capabilities (although not guaranteed).   
6. Hosting are designed to reward providing popular content, this means the hosting provider could potentially have a commercial reason to serve data or host local swarm nodes for either business or personal use.  
7.  Enterprise can maintain a private swarm network to limit the data replication boundaries  

## Swarm Details   
Swarm is a distributed & decentralised storage running on the ethereum network (although not the only one). There were two major versions of Swarm developed over time.   

Since the information is very sparse, if you are looking for up-to-date references, make sure you refer to the following:

- Swarm Bee: https://docs.ethswarm.org/  
- API References: https://docs.ethswarm.org/docs/api-reference/api-reference  
- Public Gateway: https://gateway.ethswarm.org/  


Bee is a local node, just like Personium node, but local node file system will replicate data to the public gateway. The public gateway is used for demonstration purposes, but it is enough to demonstrate how replication works, test assumption and integration mechanisms.  

I highly recommend run it with the binary `bee` to begin with, avoid using docker image at the start - because the volume mounting and network bridge can complicate things... The `bee` was built with golang, thus running the Swarm Bee node does not need to install any other dependencies. 

Download binary based on the platform: https://github.com/ethersphere/bee/releases/tag/v0.1.0


### Start Bee Node

Step 1: Create a `bee-config.yaml` file in project working directory (remember to fill a password for decryption key):

```yaml
api-addr: :8083
bootnode:
- /dnsaddr/bootnode.ethswarm.org
config: ~/.bee.yaml
cors-allowed-origins: []
data-dir: ~/.bee
db-capacity: "500000"
debug-api-addr: :6060
debug-api-enable: false
global-pinning-enable: false
help: false
nat-addr: ""
network-id: "1"
p2p-addr: :7070
p2p-quic-enable: false
p2p-ws-enable: false
password: ""
password-file: ""
payment-threshold: "100000"
payment-tolerance: "10000"
tracing-enable: false
tracing-endpoint: 127.0.0.1:6831
tracing-service-name: bee
verbosity: "5"
welcome-message: ""
```

Step 2: Start the Swarm Bee Node, and a service URL should be available at `http://localhost:8083`

```bash
bee start --config bee-config.yaml
```

Step 3: Upload file from local gateway
the gateway provides a standard http API compliance with OpenAPI 3 standard. After uploading the file, the API responses with a JSON containing Swarm file reference.

```bash
➜  curl -F file=@personium-logo.jpg  http://localhost:8083/files

{"reference":"e535b..."}
```

with file upload encryption:
```bash
curl -H "swarm-encrypt: true" -F file=@mydataorg.png https://localhost:8083/files

# returning references containing both hash and encryption key
# e.g. 276ee... is the hash
#          ece74... is the decryption key, ideally the data owner should keep the decryption key secured 
{"reference":"276ee...ece74..."}
``` 

The public gateway decryption is disabled, this means the data are replicated to the public gateway, but your data is won't allow decryption. 


The reference can be used generate a HTTP gateway link on localhost:  

- http://localhost:8083/files/e535b...

or once the file is replicated over public gateway, the same reference is accessible via:  

- https://gateway.ethswarm.org/files/e535b...


The `network-id` value `1` would by default peering to the swarm public gateway. A user can potentially create their own private cluster. this is interesting because a user can keep data privately in addition to encryption provided by Swarm, but also allow them to switch storage/hosting provider when presenting the right identity/ownership. The hosting provider should agree when full sync/replication is complete, then terminate the services to complete hosting switch.