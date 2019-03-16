# Setup Hyperledger Fabric in multiple physical machines.

[![N|Solid](http://www.1950labs.com/wiki/images/logo.png)](http://1950labs.com/)

This tutorial will show how to setup an Hyperledger Fabric starting from the **Basic-Network** example that is shared in the **Fabric-Samples** in the [Hyperledger Fabric](https://hyperledger-fabric.readthedocs.io/en/release-1.2/install.html) official page.
I will be using two droplets of [Digital Ocean](https://www.digitalocean.com/), those droples count with **Ubuntu 16.04** and **Docker 17.12**.
You have to be sure that your physical machines can comunicate with each other, if not, this solution will not work.

So we will:
* Setup a multi-host Hyperledger Fabric (two physical machines)
* Install a Business Network to our Hyperledger Fabric

## Architecture
Basically we will create a Hyperledger Fabric with one orderer organization and one peer organization. So we basically will have:
* Orderer
* Certificate Authority
* Peer 0
* Peer 1 (This will be located in other physical machine)
![Architecture](https://image.ibb.co/nEqOY9/Architecture_HLFV1.png)

## Prerequisites
* Install Docker (If you are not going to use Digital Ocean's droplets)
* Update apt (If necesary)
    * sudo apt update
* Install npm
    * apt install npm
* Install Composer
    * npm install -g composer-cli@0.20
    * npm install -g composer-rest-server@0.20
    * npm install -g generator-hyperledger-composer@0.20
	* npm install -g composer-playground@0.20
* Download Fabric-Samples
    * curl -sSL http://bit.ly/2ysbOFE | bash -s 1.4.0 1.4.0 0.4.14

## Let's start!
We will start en our first server or machine, so I will asume that you downloaded all the prerequisites in that machine
We will open the **basic-network** folder inside the **fabric-samples** folder, and we will see this folder structure.

![Folder Structure](https://image.ibb.co/hFfP6U/folder_structure.png)

If we open the **configtx.yaml** file, it will have the orderer organization and peer organization configuration.
(I removed the comments in the file in order to see a cleaner code)
```yaml
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/example.com/msp
    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp
        AnchorPeers:
            - Host: peer0.org1.example.com
              Port: 7051
Application: &ApplicationDefaults
    Organizations:

Orderer: &OrdererDefaults
    OrdererType: solo
    Addresses:
        - orderer.example.com:7050
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
    Kafka:
        Brokers:
            - 127.0.0.1:9092
    Organizations:

Profiles:
    OneOrgOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
    OneOrgChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
```
**We don't need to change anything in this file**

Now we open the **crypto-config.yaml** file, this also has configuration regarding the orderer organization and peer organization configuration.

The default configuration has configured one Peer. Since we need to create two peers, we will need to change some values in our file. So, when we create our certificates, those will be created correctly.  

```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    Template:
      Count: 1 
      #Change for 2
    Users:
      Count: 1 
      #Change for 0
```
We need to change the number of peers that we want to use, in our case, we will use '2' peers, and we will set the user count to '0', you can see the comments to know where you have to make your changes.

Before we run the **`generate.sh`** file, we have to add to the PATH enviroment variable, the location of our fabric-tools, these tools are located in the **bin** folder inside of the **fabric-samples** folder.
```sh
$ export PATH=<path to download location>/bin:$PATH
```

Now that we have our configuration files OK and we added the fabric-tools to our PATH enviroment variable, we will need to run the script **`generate.sh`** file.
```sh
$ ./generate.sh
```
This will show us this output if everything is OK.
```sh
org1.example.com
2018-08-11 21:05:07.967 UTC [common/tools/configtxgen] main -> WARN 001 Omitting the channel ID for configtxgen is deprecated.  Explicitly passing the channel ID will be required in the future, defaulting to 'testchainid'.
2018-08-11 21:05:07.968 UTC [common/tools/configtxgen] main -> INFO 002 Loading configuration
2018-08-11 21:05:07.974 UTC [common/tools/configtxgen/encoder] NewChannelGroup -> WARN 003 Default policy emission is deprecated, please include policy specificiations for the channel group in configtx.yaml
2018-08-11 21:05:07.974 UTC [common/tools/configtxgen/encoder] NewOrdererGroup -> WARN 004 Default policy emission is deprecated, please include policy specificiations for the orderer group in configtx.yaml
2018-08-11 21:05:07.974 UTC [common/tools/configtxgen/encoder] NewOrdererOrgGroup -> WARN 005 Default policy emission is deprecated, please include policy specificiations for the orderer org group OrdererOrg in configtx.yaml
2018-08-11 21:05:07.975 UTC [common/tools/configtxgen/encoder] NewOrdererOrgGroup -> WARN 006 Default policy emission is deprecated, please include policy specificiations for the orderer org group Org1MSP in configtx.yaml
2018-08-11 21:05:07.975 UTC [common/tools/configtxgen] doOutputBlock -> INFO 007 Generating genesis block
2018-08-11 21:05:07.975 UTC [common/tools/configtxgen] doOutputBlock -> INFO 008 Writing genesis block
2018-08-11 21:05:08.017 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-08-11 21:05:08.023 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-08-11 21:05:08.023 UTC [common/tools/configtxgen/encoder] NewApplicationGroup -> WARN 003 Default policy emission is deprecated, please include policy specificiations for the application group in configtx.yaml
2018-08-11 21:05:08.023 UTC [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 004 Default policy emission is deprecated, please include policy specificiations for the application org group Org1MSP in configtx.yaml
2018-08-11 21:05:08.024 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
2018-08-11 21:05:08.065 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-08-11 21:05:08.070 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-08-11 21:05:08.071 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

This will generate out crypto material (certificates, private keys, public keys), inside the **crypto-config** folder (We will copy this folder to our second server in the future).

### Configure Docker Services
We will open now the **docker-compose.yaml** file, this file now has the following services:
* `ca.example.com`
* `orderer.example.com`
* `peer0.org1.example.com`
* `couchdb`
* `cli`

In this file we will have to add/modify a couple of things.

We have to update our certificate for the **`ca.example.com`** for that we have to find this service in the file and update the key **FABRIC_CA_SERVER_CA_KEYFILE**. (You will find it in the code snippet as <New KeyFile>), the file is located in **crypto-config/peerOrganizations/org1.example.com/ca/**, you have to copy the filename of the file that ends with **_sk** and replace it for the <New KeyFile> tag
```yml
ca.example.com:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.example.com
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/<New KeyFile>
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:adminpw -d' 
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.example.com
    networks:
      - basic
```
Also in the **`ca.example.com`** service in the command section, we add other parameter a **-d**, so the section will be something like.
```yml
command: sh -c 'fabric-ca-server start -b admin:adminpw -d' 
```

Now you have to add an **extra_hosts** region after the volumes region in all the services but the **cli** and the **couchdb** services. In this section we will add the domain of our peer that will be located in the second machine (<Second machine IP address>)

This is an example.
```yml
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    extra_hosts:
    - "peer1.org1.example.com:<Second machine IP address>"
    container_name: ca.example.com
    networks:
      - basic
```
After that we will have to create a new file named **docker-compose-peer2.yaml**, this file will have the configuration of your services for your second physical machine.
```yml
version: '2'
networks:
  basic:
services:
  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer1.org1.example.com
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer1.org1.example.com:7051
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    ports:
      - 8051:7051
      - 8053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org1.example.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    extra_hosts:
    - "orderer.example.com:<First machine IP address>"
    - "peer0.org1.example.com:<First machine IP address>"
    - "ca.example.com:<First machine IP address>"
    depends_on:
      - couchdb1
    networks:
      - basic

  couchdb1:
    container_name: couchdb1
    image: hyperledger/fabric-couchdb
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 6984:5984
    networks:
      - basic
```
As you can see this just have two services **`peer1.org1.example.com`** and **`couchdb1`**, also you will need to setup you IP address of the first physical machine.
You can find it in the code snippet as <First machine IP address>.

### Start Hyperledger Fabric
Now we will start the Hyperledger Fabric in our first physical machine. We will run the **`start.sh`** file.
```sh
$ ./start.sh
```
We will see this output if everything is fine.
```sh
# Create the channel
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
2018-08-11 19:47:32.865 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-08-11 19:47:32.949 UTC [main] main -> INFO 002 Exiting.....
# Join peer0.org1.example.com to the channel.
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel join -b mychannel.block
2018-08-11 19:47:33.071 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-08-11 19:47:33.147 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
2018-08-11 19:47:33.148 UTC [main] main -> INFO 003 Exiting.....
```
And to validate that our Docker containers are up, we will run
```sh
$ docker ps
```
And we have to see our 4 services running.

```sh
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
39f709f2110a        hyperledger/fabric-peer      "peer node start"        13 seconds ago      Up 12 seconds       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
01b8d41a4620        hyperledger/fabric-orderer   "orderer"                15 seconds ago      Up 13 seconds       0.0.0.0:7050->7050/tcp                           orderer.example.com
692fbe8dafe9        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   15 seconds ago      Up 13 seconds       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
9616be4ea791        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   15 seconds ago      Up 14 seconds       0.0.0.0:7054->7054/tcp                           ca.example.com
```

So with that we have our Hyperledger Fabric running in our first server.

Now we go to the second server, we will copy our **crypto-config** folder created on our first server.
Also we will take the **`docker-composer-peer2.yml`** created some steps before, the **env** file and create a new file named **`start-peer2.sh`**.
The folder structure of out second server must be something like that.

![Server2 Folder Structure ](https://image.ibb.co/cawg39/Folder_Structure_S2.png)

The **`start-peer2.sh`** will start our second peer and join it to the Hyperledger Fabric started in our first server.
We will open the file and add this code.
```sh
set -ev
# don't rewrite paths for Windows Git Bash users
export MSYS_NO_PATHCONV=1

docker-compose -f docker-compose-peer2.yml down
docker-compose -f docker-compose-peer2.yml up -d peer1.org1.example.com couchdb1

export FABRIC_START_TIMEOUT=10
#echo ${FABRIC_START_TIMEOUT}
sleep ${FABRIC_START_TIMEOUT}

# Create the channel
docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer1.org1.example.com peer channel fetch config -o orderer.example.com:7050 -c mychannel

docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer1.org1.example.com peer channel join -b mychannel_config.block
```
Save and now we will run the **`start-peer2.sh`**.
```sh
$ ./start-peer2.sh
```
If everything its Ok we will see something like this.
```sh
# Create the channel
docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer1.org1.example.com peer channel fetch config -o orderer.example.com:7050 -c mychannel
2018-08-11 21:13:49.790 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-08-11 21:13:49.860 UTC [cli/common] readBlock -> INFO 002 Received block: 0
2018-08-11 21:13:49.930 UTC [cli/common] readBlock -> INFO 003 Received block: 0

docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer1.org1.example.com peer channel join -b mychannel_config.block
2018-08-11 21:13:50.094 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-08-11 21:13:50.173 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
```
To validate that our peer in the second service joined successfully to our Hyperledger Fabric run:
```sh
$ docker ps
```
You will something like this
```sh
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
5cec846146df        hyperledger/fabric-peer      "peer node start"        2 minutes ago       Up 2 minutes        0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp   peer1.org1.example.com
60413da0259a        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   2 minutes ago       Up 2 minutes        4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp       couchdb1
```
Now we will run:
```sh
#The Container Id of our peer
$ docker logs 5cec846146df
```
And we will see something like this at the end of the log
```sh
2018-08-11 21:13:50.116 UTC [kvledger] CommitWithPvtData -> INFO 029 Channel [mychannel]: Committed block [0] with 1 transaction(s)
2018-08-11 21:13:50.117 UTC [pvtdatastorage] func1 -> INFO 02a Purger started: Purging expired private data till block number [0]
2018-08-11 21:13:50.117 UTC [pvtdatastorage] func1 -> INFO 02b Purger finished
2018-08-11 21:13:50.149 UTC [ledgermgmt] CreateLedger -> INFO 02c Created ledger [mychannel] with genesis block
2018-08-11 21:13:50.170 UTC [cscc] Init -> INFO 02d Init CSCC
2018-08-11 21:13:50.171 UTC [sccapi] deploySysCC -> INFO 02e system chaincode cscc/mychannel(github.com/hyperledger/fabric/core/scc/cscc) deployed
2018-08-11 21:13:50.171 UTC [sccapi] deploySysCC -> INFO 02f system chaincode lscc/mychannel(github.com/hyperledger/fabric/core/scc/lscc) deployed
2018-08-11 21:13:50.171 UTC [qscc] Init -> INFO 030 Init QSCC
2018-08-11 21:13:50.172 UTC [sccapi] deploySysCC -> INFO 031 system chaincode qscc/mychannel(github.com/hyperledger/fabric/core/scc/qscc) deployed
2018-08-11 21:13:56.171 UTC [gossip/election] beLeader -> INFO 032 [134 251 205 127 134 154 103 139 61 25 239 14 36 136 229 124 78 89 60 237 29 37 27 83 96 123 4 22 167 24 205 52] : Becoming a leader
```
So now we have our Hyperledger Fabric running in two separate physical machines!

## Create a Peer Admin Card
In this step we will create a Peer Admin Card for our Hyperledger Fabric network, now we will create a file named **`createPeerAdminCard.sh`**. This file have the responsabilty to create our server peer admin, that we will use to manage our network.
So we will put this code inside the file:
```sh
Usage() {
	echo ""
	echo "Usage: ./createPeerAdminCard.sh [-h host] [-n]"
	echo ""
	echo "Options:"
	echo -e "\t-h or --host:\t\t(Optional) name of the host to specify in the connection profile"
	echo -e "\t-n or --noimport:\t(Optional) don't import into card store"
	echo ""
	echo "Example: ./createPeerAdminCard.sh"
	echo ""
	exit 1
}

Parse_Arguments() {
	while [ $# -gt 0 ]; do
		case $1 in
			--help)
				HELPINFO=true
				;;
			--host | -h)
                shift
				HOST="$1"
				;;
            --noimport | -n)
				NOIMPORT=true
				;;
		esac
		shift
	done
}

HOST=localhost
Parse_Arguments $@

if [ "${HELPINFO}" == "true" ]; then
    Usage
fi

# Grab the current directory
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

if [ -z "${HL_COMPOSER_CLI}" ]; then
  HL_COMPOSER_CLI=$(which composer)
fi

echo
# check that the composer command exists at a version >v0.16
COMPOSER_VERSION=$("${HL_COMPOSER_CLI}" --version 2>/dev/null)
COMPOSER_RC=$?

if [ $COMPOSER_RC -eq 0 ]; then
    AWKRET=$(echo $COMPOSER_VERSION | awk -F. '{if ($2<19) print "1"; else print "0";}')
    if [ $AWKRET -eq 1 ]; then
        echo Cannot use $COMPOSER_VERSION version of composer with fabric 1.1, v0.19 or higher is required
        exit 1
    else
        echo Using composer-cli at $COMPOSER_VERSION
    fi
else
    echo 'No version of composer-cli has been detected, you need to install composer-cli at v0.19 or higher'
    exit 1
fi

cat << EOF > DevServer_connection.json
<JSON CONNECTION PROFILE SECTION>
EOF

PRIVATE_KEY="${DIR}"/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/3c4f94b15fc3eef33c85dd640dbce88bb4e72df43599185d6acb81b96fefae98_sk
CERT="${DIR}"/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem

if [ "${NOIMPORT}" != "true" ]; then
    CARDOUTPUT=/tmp/PeerAdmin@hlfv1.card
else
    CARDOUTPUT=PeerAdmin@hlfv1.card
fi

"${HL_COMPOSER_CLI}"  card create -p DevServer_connection.json -u PeerAdmin -c "${CERT}" -k "${PRIVATE_KEY}" -r PeerAdmin -r ChannelAdmin --file $CARDOUTPUT

if [ "${NOIMPORT}" != "true" ]; then
    if "${HL_COMPOSER_CLI}"  card list -c PeerAdmin@hlfv1 > /dev/null; then
        "${HL_COMPOSER_CLI}"  card delete -c PeerAdmin@hlfv1
    fi

    "${HL_COMPOSER_CLI}"  card import --file /tmp/PeerAdmin@hlfv1.card 
    "${HL_COMPOSER_CLI}"  card list
    echo "Hyperledger Composer PeerAdmin card has been imported, host of fabric specified as '${HOST}'"
    rm /tmp/PeerAdmin@hlfv1.card
else
    echo "Hyperledger Composer PeerAdmin card has been created, host of fabric specified as '${HOST}'"
fi
```
Now we have to update the file with our **Connection Profile**, we will replace the **<JSON CONNECTION PROFILE SECTION>** tag with the following json.
```json
{
    "name": "hlfv1",
    "x-type": "hlfv1",
    "version": "1.0.0",
	"client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300",
                    "eventHub": "300",
                    "eventReg": "300"
                },
                "orderer": "300"
            }
        }
    },
    "channels": {
        "mychannel": {
            "orderers": [
                "orderer.example.com"
            ],
            "peers": {
                "peer0.org1.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                },
				"peer1.org1.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                }
            }
        }
    },
    "organizations": {
        "Org1": {
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com",
				"peer1.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.example.com"
            ]
        }
    },
    "orderers": {
        "orderer.example.com": {
            "url": "grpc://<IP First Server>:7050",
            "grpcOptions": {
                "ssl-target-name-override": "orderer.example.com"
            },
            "tlsCACerts": {
                "pem": "<Orderer CA Cert>"
            }
        }
    },
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpc://<IP First Server>:7051",
            "eventUrl": "grpc://<IP First Server>:7053",
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org1.example.com"
            },
            "tlsCACerts": {
                "pem": "<Peer CA Cert>"
            }
        },
		"peer1.org1.example.com": {
            "url": "grpc://<IP Second Server>:8051",
            "eventUrl": "grpc://<IP Second Server>:8053",
            "grpcOptions": {
                "ssl-target-name-override": "peer1.org1.example.com"
            },
            "tlsCACerts": {
                "pem": "<Peer CA Cert>"
            }
        }
    },
    "certificateAuthorities": {
        "ca.example.com": {
            "url": "http://<IP First Server>:7054",
            "caName": "ca.example.com",
            "httpOptions": {
                "verify": false
            }
        }
    }
}
```
We need to modify several things in this json, first we will put our servers IP in the tags **<IP First Server>** and **<IP Second Server>**.
After that we need to update our Certificates, we need to update this tags **<Orderer CA Cert>** and **<Peer CA Cert>**.
To get value of **<Orderer CA Cert>** we will execute this:
```sh
$ awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
This will return something like this
```sh
-----BEGIN CERTIFICATE-----\nMIICNTCCAdygAwIBAgIRAJWZVOq9L5YiIFegOAWS8mgwCgYIKoZIzj0EAwIwbDEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xFDASBgNVBAoTC2V4YW1wbGUuY29tMRowGAYDVQQDExF0bHNjYS5l\neGFtcGxlLmNvbTAeFw0xODA4MTEyMTAwMDdaFw0yODA4MDgyMTAwMDdaMGwxCzAJ\nBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJh\nbmNpc2NvMRQwEgYDVQQKEwtleGFtcGxlLmNvbTEaMBgGA1UEAxMRdGxzY2EuZXhh\nbXBsZS5jb20wWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAASn3ngfrIOXpC0LbYCx\nX/WQgE7J/RXxMG5wiPRpLUMU1kG/qyFtwO8PEr7ilBwwX9/CmymoDnsIeK5DxhBa\nuZqdo18wXTAOBgNVHQ8BAf8EBAMCAaYwDwYDVR0lBAgwBgYEVR0lADAPBgNVHRMB\nAf8EBTADAQH/MCkGA1UdDgQiBCDCBdqJVf5rJRR+GEInsIRvDgonmM2pjvNw0KUr\necCOGDAKBggqhkjOPQQDAgNHADBEAiBfAhEZ1g3UTfVY5U+0RySSeDuVM3YhFZxq\nhX5lCJwZGQIgLL8vbiMx+XTC5PLSLlqFDKLrrGTilV8r6pPH3Pao1FM=\n-----END CERTIFICATE-----\n
```
We will copy that output inside the **<Orderer CA Cert>** tag, we will see something like this.
```json
{
"orderer.example.com": {
            "url": "grpc://<IP First Server>:7050",
            "grpcOptions": {
                "ssl-target-name-override": "orderer.example.com"
            },
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICNjCCAdygAw.....ojmoLOGj\n-----END CERTIFICATE-----\n"
        }
    }
}
```
Now we will do the same with the **<Peer CA Cert>**, executing this command:
```sh
$ awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
This it will show this output.
```sh
-----BEGIN CERTIFICATE-----\nMIICSTCCAfCgAwIBAgIRAJkUpDVtZ0g1/N46SwkP5IgwCgYIKoZIzj0EAwIwdjEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHzAdBgNVBAMTFnRs\nc2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMTgwODExMjEwMDA3WhcNMjgwODA4MjEw\nMDA3WjB2MQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UE\nBxMNU2FuIEZyYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEfMB0G\nA1UEAxMWdGxzY2Eub3JnMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49\nAwEHA0IABJvn7Tgbeez1yZtCVdPYIro3Lk8pcvMU6jRYB7zOwD943DTPACfQbpOE\n6f0gJmCDtTLwsiFeL53we5MVda81Kq6jXzBdMA4GA1UdDwEB/wQEAwIBpjAPBgNV\nHSUECDAGBgRVHSUAMA8GA1UdEwEB/wQFMAMBAf8wKQYDVR0OBCIEINfi+mMVmuTH\noDlsV0YWLxIlK0zMQeF4wKwQnfsCGW5eMAoGCCqGSM49BAMCA0cAMEQCIFtE1LYR\nXRlqeH2ZwcbDj9SqLtC5/ADU1DXIQ/4x1o4aAiB+/C0TT70TT+6ZpEgK8P7EC/9u\nefJIEPDbPgbryw75hQ==\n-----END CERTIFICATE-----\n
```
And our json has to look like this
```json
{
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpc://<IP First Server>:7051",
            "eventUrl": "grpc://<IP First Server>:7053",
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org1.example.com"
            },
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICST...75hQ==\n-----END CERTIFICATE-----\n"
            }
        },
		"peer1.org1.example.com": {
            "url": "grpc://<IP Second Server>:8051",
            "eventUrl": "grpc://<IP Second Server>:8053",
            "grpcOptions": {
                "ssl-target-name-override": "peer1.org1.example.com"
            },
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICST...75hQ==\n-----END CERTIFICATE-----\n"
            }
        }
    }
}
```
Thats all regarding our connection profile for out Peer Admin Card, now we have to update **PRIVATE_KEY** variable, it's located after the connection profile.
```sh
PRIVATE_KEY="${DIR}"/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/<KeyStoreFile>
```
As you can see the **<KeyStoreFile>** is located in the path showed before, so the variable it will look like this.
```sh
PRIVATE_KEY="${DIR}"/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/3c4f94b15fc3eef33c85dd640dbce88bb4e72df43599185d6acb81b96fefae98_sk
```
Now we will execute the **`createPeerAdminCard.sh`**
```sh
$ ./createPeerAdminCard.sh
```
We will see this output if everything it's OK
```sh
Using composer-cli at v0.19.12

Successfully created business network card file to
        Output file: /tmp/PeerAdmin@hlfv1.card

Command succeeded
Successfully imported business network card
        Card file: /tmp/PeerAdmin@hlfv1.card
        Card name: PeerAdmin@hlfv1

Command succeeded

The following Business Network Cards are available:
Connection Profile: hlfv1
┌─────────────────┬───────────┬──────────────────┐
│ Card Name       │ UserId    │ Business Network │
├─────────────────┼───────────┼──────────────────┤
│ PeerAdmin@hlfv1 │ PeerAdmin │                  │
└─────────────────┴───────────┴──────────────────┘
Issue composer card list --card <Card Name> to get details a specific card
Command succeeded
Hyperledger Composer PeerAdmin card has been imported, host of fabric specified as 'localhost'
```
With that we have our Peer Admin Card created and ready for use, now we will install a Business Network using the card.

## Install the Business Network
For this of course we need a Business Network, we will take it from https://composer-playground.mybluemix.net/ here is an example Business Network, we need to export the .bna file and move it to our first server.
**Important:** You must remember the version of the Business Network that you are exporting, you will use the version number in the commands in the future.

![Basic Sample](https://i.ibb.co/YX9NnLK/Basic-Sample.png)

In the image we see that the version number is **0.2.6**

After that we will execute the following command to install the business network in out Hyperledger Fabric.
```sh
$ composer network install --card PeerAdmin@hlfv1 --archiveFile my-basic-sample.bna
```
And we will see this output if it's all Ok.
```sh
✔ Installing business network. This may take a minute...
Successfully installed business network my-basic-sample, version 0.2.6
Command succeeded
```
Now we will execute this command to start out Business Network.
```sh
$ composer network start --networkName my-basic-sample --networkVersion 0.2.6 --networkAdmin admin --networkAdminEnrollSecret adminpw --card PeerAdmin@hlfv1 --file networkadmin.card
 ```
 This has to be the output
 ```sh
Starting business network my-basic-sample at version 0.2.6 
Processing these Network Admins:
        userName: admin
✔ Starting business network definition. This may take a minute...
Successfully created business network card:
        Filename: networkadmin.card
Command succeeded
 ```
We have to import our Business Network card now.
```sh
 $ composer card import --file networkadmin.card
 ```
 Showing this output
 ```sh
 Successfully imported business network card
        Card file: networkadmin.card
        Card name: admin@my-basic-sample
Command succeeded
```
Now just we will test the connection to our Business Network running:
```sh
$ composer network ping --card admin@my-basic-sample
 ```
 And the output:
 ```sh
The connection to the network was successfully tested: my-basic-sample
        Business network version: 0.2.6
        Composer runtime version: 0.20.0
        participant: org.hyperledger.composer.system.NetworkAdmin#admin
        identity: org.hyperledger.composer.system.Identity#01959ec6ef322fe987223f002498b3a80b03dad2882a353ebfaf20be9370907f
Command succeeded
 ```
 
Well and that's all, now we have our Hyperledger Fabric running in two separate machines with one Business Network running on it!.
 Let's do some validation, using de **Composer Playground**
 
Run this command in your first server to start an instance of the Playground API to play around
  ```sh
 $ composer-playground -p 8081
  ```
So, if we browse to that point to the port specified before we must see something like this.
  
  ![Validation #1](https://i.ibb.co/KGcCL9c/1.png)
  
The image show us the two card that we created the Peer Admin Card, and the Network Admin Card, we will go inside our Business Network, clicking **Connect Now**, go to the **Test** tab.

  ![Validation #2](https://i.ibb.co/KjVcV8X/2.png)

We will add a new Participant, clicking the **Create New Participant** button in the top right.

  ![Validation #3](https://i.ibb.co/yprYkSC/3.png)
  
And let's click **Create New**, if everything its ok, we will see our participant in the grid.

  ![Validation #4](https://i.ibb.co/X75sbYL/4.png)

Now lets run the **docker ps** command in our first server.
  ```sh
$ docker ps
  ```
And let see the logs of our **Peer 0**
```sh
$ docker logs <Peer 0 Container Id>
```
We have to see some activity in our Peer.
```sh
2018-08-13 23:48:23.701 UTC [kvledger] CommitWithPvtData -> INFO 03c Channel [mychannel]: Committed block [3] with 1 transaction(s)
```
Let's repeat this process to our **Peer 1** in our second server.
  ```sh
$ docker ps
  ```
And let see the logs of our **Peer 1**
```sh
$ docker logs <Peer 1 Container Id>
```
We have to see some activity in our Peer.
```sh
2018-08-13 23:48:23.731 UTC [kvledger] CommitWithPvtData -> INFO 03b Channel [mychannel]: Committed block [3] with 1 transaction(s)
```

And that's all, we see that our Hyperledger Fabric with an Business Network is working like charm! :)