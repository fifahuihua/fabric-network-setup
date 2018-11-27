# fabric-network-setup
A demo to setup hyperledger fabric network with mulitple machines/servers.

# Assumption
* Installed Docker (version >= 17.06.2)
* Installed Docker Composer (version >= 1.14.0)
* Installed Golang (version >= 1.11.0)
* Installed git
* Installed python (Only Support 2.7.*)
* Installed Node.js (version >= 8.9.0)
* IP address of host 1: 10.222.48.146
* IP address of host 2: 10.222.48.162
* There are 2 peers in org1, and 2 peers in org2.
* There orderer is in host 1.

And You have pulled following docker images:

| REPOSITORY                    | TAG                | IMAGE ID             | CREATED              | SIZE | 
|-------------------------------|--------------------|----------------------|----------------------|-------|
| hyperledger/fabric-javaenv    | 1.3.0              | 2476cefaf833         | 5 weeks ago          | 1.7GB | 
| hyperledger/fabric-javaenv    | latest             | 2476cefaf833         | 5 weeks ago          | 1.7GB | 
| hyperledger/fabric-ca         | 1.3.0              | 5c6b20ba944f         | 5 weeks ago          | 244MB | 
| hyperledger/fabric-ca         | latest             | 5c6b20ba944f         | 5 weeks ago          | 244MB | 
| hyperledger/fabric-tools      | 1.3.0              | c056cd9890e7         | 5 weeks ago          | 1.5GB | 
| hyperledger/fabric-tools      | latest             | c056cd9890e7         | 5 weeks ago          | 1.5GB | 
| hyperledger/fabric-ccenv      | 1.3.0              | 953124d80237         | 5 weeks ago          | 1.38GB | 
| hyperledger/fabric-ccenv      | latest             | 953124d80237         | 5 weeks ago          | 1.38GB | 
| hyperledger/fabric-orderer    | 1.3.0              | f430f581b46b         | 5 weeks ago          | 145MB | 
| hyperledger/fabric-orderer    | latest             | f430f581b46b         | 5 weeks ago          | 145MB | 
| hyperledger/fabric-peer       | 1.3.0              | f3ea63abddaa         | 5 weeks ago          | 151MB | 
| hyperledger/fabric-peer       | latest             | f3ea63abddaa         | 5 weeks ago          | 151MB | 
| hyperledger/fabric-zookeeper  | 0.4.13             | e62e0af39193         | 6 weeks ago          | 1.39GB | 
| hyperledger/fabric-zookeeper  | latest             | e62e0af39193         | 6 weeks ago          | 1.39GB | 
| hyperledger/fabric-kafka      | 0.4.13             | 4121ea662c47         | 6 weeks ago          | 1.4GB | 
| hyperledger/fabric-kafka      | latest             | 4121ea662c47         | 6 weeks ago          | 1.4GB | 
| hyperledger/fabric-couchdb    | 0.4.13             | 1d3266e01e64         | 6 weeks ago          | 1.45GB | 
| hyperledger/fabric-couchdb    | latest             | 1d3266e01e64         | 6 weeks ago          | 1.45GB | 
| hyperledger/fabric-baseos     | amd64-0.4.13       | f0fe49196c40         | 6 weeks ago          | 124MB | 

> **Please replace the IP address in the docker-compose-org1.yaml and docker-compose-org1.yaml with the IP addresses of your hosts.**

# Commands

## Step1, Run following commands in Machine 2
1. git clone https://github.com/fifahuihua/fabric-network-setup.git
1. cd fabric-network-setup
1. ./byfn.sh down 
1. docker volume rm net_peer0.org2.example.com
1. docker volume rm net_peer1.org2.example.com
1. docker image rm $(docker image ls -q dev-*)

## Step2, Run following commands in Machine 1
1. git clone https://github.com/fifahuihua/fabric-network-setup.git
1. cd fabric-network-setup
1. ./byfn.sh down 
1. docker volume rm net_orderer.example.com
1. docker volume rm net_peer0.org1.example.com
1. docker volume rm net_peer1.org1.example.com
1. docker image rm $(docker image ls -q dev-*)
1. ./byfn.sh generate
1. scp -r ./crypto-config antony@10.222.48.162:/home/antony/projects/fabric-network-setup
1. docker-compose -f docker-compose-org1.yaml up -d

## Step3, Run following commands in Machine 2
1. docker-compose -f docker-compose-org2.yaml up -d

## Step4, Run following commands in Machine 1 
1. docker exec -it cli bash
1. export CHANNEL_NAME=mychannel 
1. peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
1. peer channel join -b mychannel.block
1. CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block
1. peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
1. CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
1. peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
1. CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
1. peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
1. peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
1. CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
1. peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
1. peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
1. CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
