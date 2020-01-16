# small dev network

A basic single organisation network for when you need something small and simple to get started.

_Based on the fabric-samples basic-network_

## Prereqs

Download the Hyperledger Fabric docker images.

```
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh | bash -s -- 2.0.0-beta 1.4.4 0.4.18 -s -b
```

## Usage

Thinking about creating a docker image based on fabric-tools with additional scripts to automate these steps, but for now run the following to set up a small network.

Generate:

```
docker run --rm -v small_config:/etc/hyperledger/fabric -w /etc/hyperledger/fabric --entrypoint=/bin/bash hyperledgendary/fabric-tools -c "setnet.sh"
```

Check what ended up in _/etc/hyperledger/fabric_! 

```
docker run --rm -it -v small_config:/etc/hyperledger/fabric --entrypoint=/bin/bash hyperledgendary/fabric-tools
```

Start the fans!

TODO copy docker compose file out of hyperlegendary image?

```
docker-compose -f docker-compose.yml up -d ca.humboldt.example.com orderer.example.com peer0.humboldt.example.com couchdb cli
```

Check everything looks ok (see logging below)

```
docker ps
```

Create the network

```
docker exec humboldt.cli mknet.sh
```

Check what happened!

```
docker exec humboldt.cli peer channel list
```

```
docker exec humboldt.cli peer channel getinfo -c smallchannel
```

## Chaincode lifecycle

```
cd <CHAINCODE_DIR>
docker run --rm -e CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/fabric/crypto-config/peerOrganizations/humboldt.example.com/users/Admin@humboldt.example.com/msp -v ${PWD}:/local:ro -v small_config:/etc/hyperledger/fabric -v small_cli:/var/hyperledgendary/fdn -w /var/hyperledgendary/fdn --entrypoint=/bin/bash hyperledgendary/fabric-tools -c "peer lifecycle chaincode package ./fabcar.tar.gz --path /local --lang <golang/java/node> --label fabcar_v1"
```

**Note:** currently fails for java implementation of fabcar

```
docker exec humboldt.cli peer lifecycle chaincode install /var/hyperledgendary/fdn/fabcar.tar.gz
```

```
docker exec humboldt.cli peer lifecycle chaincode queryinstalled
```

```
docker exec humboldt.cli peer lifecycle chaincode approveformyorg -o orderer.example.com:7050 --channelID smallchannel --name fabcar --version 1 --init-required --sequence 1 --waitForEvent --package-id <PACKAGE_ID>
```

```
docker exec humboldt.cli peer lifecycle chaincode checkcommitreadiness -o orderer.example.com:7050 --channelID smallchannel --name fabcar --version 1 --sequence 1 --output json --init-required
```

```
docker exec humboldt.cli peer lifecycle chaincode commit -o orderer.example.com:7050 --channelID smallchannel --name fabcar --version 1 --sequence 1 --init-required
```

```
docker exec humboldt.cli peer lifecycle chaincode querycommitted --channelID smallchannel --name fabcar
```

```
docker exec humboldt.cli peer chaincode invoke -o orderer.example.com:7050 -C smallchannel -n fabcar --isInit -c '{"function":"initLedger","Args":[]}'
```

```
docker exec humboldt.cli peer chaincode query -C smallchannel -n fabcar -c '{"Args":["queryAllCars"]}'
```

## Logging

```
docker-compose -f docker-compose.yml up -d logspout && curl http://127.0.0.1:8000/logs
```

## Cleaning up

```
docker-compose -f docker-compose.yml down --volumes --remove-orphans
```

```
docker rm -f $(docker ps -aq)
docker container prune
docker volume prune
docker network prune
```

```
docker rmi $(docker images -a -q)
```

```
docker system prune -a
```