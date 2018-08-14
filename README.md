# DTN-docs
Distributed Trade Network

### How to verify transactions

1. Transparent and credible permissioned blockchain ledger.

    TigerWit togther with our partners, regulators, users will set up a permissioned blockchain Ledger (At first, it will be a beta version) by open source software [Hyperledger Fabric](https://github.com/hyperledger/fabric). Only by collect  enough endorsement that signed by member of  permissioned blockchain, the transaction can be record on the chain. Once it record, it cannot be modified anymore. We support that: our users can apply to become `ordinary member` of the  permissioned blockchain. And some users can be a `permissioned member` atfter they finish the high level KYC audit.


2. Security of privacy protection
    For privacy thinking, We only record the `digest` of the transaction on the blockchain ledger.  `Digest` generated by hash algorithm. Any change will lead to differet value of the `digest`, thus,  ensure the consistency of original transaction data and digest. And Also ensure simplification of data.

3. Two-way authentication transaction data
  We offer the following way to enable users to verify transaction data:

   1. Original transaction data to the Digest: users can calculate the digest value `V1` by the common hash algorithm we published on the website.  And then they can visit any node of blockchain query service that deployed in the blockchain node to verify if `V1` exist. 


   2. Digest to original transaction data: users can visit any node of blockchain query service that deployed in the blockchain node to query their transactions history data. selcet digest `V2` one of the history data, go to offical website query  the  original transaction data of `V2` to verify that if data has been modified.



### How to become an ordinary node of the permissioned blockchain:

Prerequisites：
  Before you start, you should get `Docker` installed(see `https://docs.docker.com/install/`) in your Linux server。

Steps:

1. Use the following command to set up evironment：

  ```bash
  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0
  cd fabric-samples
  ```

2. Create a new file called `crypto-config.yaml`. Edit it with the following content:

   ```yaml
   PeerOrgs:
     - Name: YourOrgName
       Domain: YourOrgName.yourdomain.com
       Template:
         Count: 1  
       Users:
         Count: 1
   ```

  You should relace `YorOrgName` with your Org name.
3.  Generate the Your Org Crypto Material.

  ```
     ./bin/cryptogen extend --config=./crypto-config.yaml 
  ```

  Excute `tree crypto-config -L 4`，you will see something like the following information: 

   ```bash
   crypto-config
   └── peerOrganizations
    └── YourOrgName.yourdomain.com
        ├── ca
        │   ├── c13cbe59063d173a4bf4aa5222a6d08880c7863ef09db1901c671936481dd1be_sk
        │   └── ca.YourOrgName.yourdomain.com-cert.pem
        ├── msp
        │   ├── admincerts
        │   ├── cacerts
        │   └── tlscacerts
        ├── peers
        │   └── peer0.YourOrgName.yourdomain.com
        │   
        ├── tlsca
        │   ├── 94a6de4c811c100f9c8a685a06ee1dfbaa27ef5d4ca10a6fa8032342b0155426_sk
        │   └── tlsca.YourOrgName.yourdomain.com-cert.pem
        └── users
            ├── Admin@YourOrgName.yourdomain.com
            └── User1@YourOrgName.yourdomain.com
   ```

4. Create a new file called `peer-base.yaml`. Edit it with the following content:

  ```yaml
  version: '2'
  services:
    peer-base:
      image: hyperledger/fabric-peer:latest
      restart: always
      environment:
        - GODEBUG=netdns=go
        - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
        - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=host
        - CORE_LOGGING_LEVEL=INFO
        - CORE_LOGGING_FORMAT=%{color}[%{id:03x} %{time:01-02 15:04:05.00 MST}] [%{longpkg}] %{callpath} -> %{level:.4s}%{color:reset} %{message}
        - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=v120_default  # uncomment this to use specific network
        - CORE_PEER_TLS_ENABLED=true
        - CORE_PEER_GOSSIP_USELEADERELECTION=true
        - CORE_PEER_GOSSIP_ORGLEADER=false
        - CORE_PEER_PROFILE_ENABLED=false
        - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
        - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
        - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
        - CORE_CHIANCODE_LOGGING_LEVEL=DEBUG
        - CORE_CHIANCODE_LOGGING_FORMAT=%{color}[%{id:03x} %{time:01-02 15:04:05.00 MST}] [%{longpkg}] %{callpath} -> %{level:.4s}%{color:reset} %{message}
        - CORE_PEER_MSPCONFIGPATH = /
      expose:
        - "7051"  # Grpc
        - "7052"  # Peer CLI
        - "7053"  # Peer Event
      volumes:
        - $GOPATH/src/github.com/hyperledger/fabric:/go/src/github.com/hyperledger/fabric
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
      command: peer node start
  ```
5. Create a new file called `docker-compose-base.yaml`. Edit it with the following content:
  ```yaml
  version: '2'
  services:
    peer0.YourOrgName.yourdomain.com:
      container_name: peer0.YourOrgName.yourdomain.com
      extends:
        file: peer-base.yaml
        service: peer-base
      environment:
        - CORE_PEER_ID=peer0.YourOrgName.yourdomain.com
        - CORE_PEER_ADDRESS=peer0.YourOrgName.yourdomain.com:7051
        - CORE_PEER_CHAINCODEADDRESS=peer0.YourOrgName.yourdomain.com:7052
        - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
        - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.YourOrgName.yourdomain.com:7051
        - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.trade.tigerwit.com:7051
        - CORE_PEER_LOCALMSPID=YourOrgNameMSP
      volumes:
          - /var/run/:/host/var/run/
          - ../crypto-config/peerOrganizations/YourOrgName.yourdomain.com/peers/peer0.YourOrgName.yourdomain.com/msp:/etc/hyperledger/fabric/msp
          - ../crypto-config/peerOrganizations/YourOrgName.yourdomain.com/peers/peer0.YourOrgName.yourdomain.com/tls:/etc/hyperledger/fabric/tls
      ports:
        - 7051:7051
        - 7052:7052
        - 7053:7053
  ```

6.  Create a new file called`docker-compose-yourorgname.yaml`. Edit it with the following content

  ```yaml
  version: '2'
    services:
      peer0.YourOrgName.yourdomain.com:
        extends:
          file:   base/docker-compose-base.yaml
          service: peer0.YourOrgName.yourdomain.com
        container_name: peer0.YourOrgName.yourdomain.com
      peer1.YourOrgName.yourdomain.com:
        extends:
          file:   base/docker-compose-base.yaml
          service: peer1.YourOrgName.yourdomain.com
        container_name: peer1.YourOrgName.yourdomain.com  
  
  ```

 7. Start peers：

  ```bash 
    docker-compose -f docker-compose-yourorgname.yaml up -d
  ```

    8. Packaged `msp` folder. 

  ```bash
  tar -czvf msp.tar.gz crypto-config/peerOrganizations/YourOrgName.yourdomain.com/msp
  ```

    9. After you pass KYC verification successfully,  fill out the [Application Form](https://docs.google.com/forms/d/e/1FAIpQLSfEKn9Nd-KNC58xSykppZYxtdc_0qwIGjP9KhHZ0-5on3bsxQ/viewform?usp=sf_link` )

  10.After all organizations signature your application，you will receive the  email which will tell you the value of `YourOrgNameMSP`, `CHAINCODE_NAME`, `CHAINCODE_VERSION`, `CHAINCODE_SRC_PATH` and send you a file called: `tradechannel.block`.

  Download the  `tradechannel.block` to the directory `fabric-samples`

    11. Join  Channel 

  ```bash
     export FABRIC_CFG_PATH=`pwd`
     export CORE_PEER_TLS_ENABLED=true
     export CORE_PEER_TLS_CERT_FILE=YOUR_TLS_PATH/client.crt
     export CORE_PEER_TLS_KEY_FILE=YOUR_TLS_PATH/client.key
     export CORE_PEER_MSPCONFIGPATH=YOUR_MSP_PATH
     export CORE_PEER_ADDRESS=YOUR_PEER_ADDRESS
     export CORE_PEER_LOCALMSPID=YOUR_MSP_ID
     export CORE_PEER_TLS_ROOTCERT_FILE=YOUR_TLS_PATH/ca.crt
     export CORE_PEER_ID=cli
     ./bin/peer channel join -b tradechannel.block
  ```

  You should:
    repalce `YOUR_TLS_PATH`   with `./crypto-config/peerOrganizations/YourOrgName.yourdomain.com/users/Admin@YourOrgName.yourdomain.com/tls`
    repalce `YOUR_MSP_PATH`  with `./crypto-config/peerOrganizations/YourOrgName.yourdomain.com/users/Admin@YourOrgName.yourdomain.com/msp`
    repalce `YOUR_PEER_ADDRESS` with `peer0.YourOrgName.yourdomain.com:7051`
    repalce `YOUR_MSP_ID` with  `YourOrgNameMSP`  

 The same below. 


 12. Install the `chainchode`:
   ```bash
     export FABRIC_CFG_PATH=`pwd`
     export CORE_PEER_TLS_ENABLED=true
     export CORE_PEER_TLS_CERT_FILE=YOUR_TLS_PATH/client.crt
     export CORE_PEER_TLS_KEY_FILE=YOUR_TLS_PATH/client.key
     export CORE_PEER_MSPCONFIGPATH=YOUR_MSP_PATH
     export CORE_PEER_ADDRESS=YOUR_PEER_ADDRESS
     export CORE_PEER_LOCALMSPID=YOUR_MSP_ID
     export CORE_PEER_TLS_ROOTCERT_FILE=YOUR_TLS_PATH/ca.crt
     export CORE_PEER_ID=cli
     ./bin/peer chaincode install -n CHAINCODE_NAME -v CHAINCODE_VERSION -p CHAINCODE_SRC_PATH
   ```

After finish all steps，you become a node of `Permissioned Blockchain`.


In addition:
  If you want to become a `permissioned` node of blockchain, please contact us:support@tigerwit.co.uk.
