# chainflip-perseverance

## Pre-requisites
- Docker (https://docs.docker.com/get-docker/)
- JQ (https://stedolan.github.io/jq/download/)

## Setup
### Clone the repo

```bash
git clone https://github.com/chainflip-io/chainflip-perseverance.git
cd chainflip-perseverance
```

### Generating Keys

```bash
mkdir -p ./chainflip/keys/lp
mkdir -p ./chainflip/keys/broker
docker run --platform=linux/amd64 --entrypoint=/usr/local/bin/chainflip-node chainfliplabs/chainflip-node:perseverance key generate --output-type=json > chainflip/lp-keys.json
docker run --platform=linux/amd64 --entrypoint=/usr/local/bin/chainflip-node chainfliplabs/chainflip-node:perseverance key generate --output-type=json > chainflip/broker-keys.json
cat chainflip/broker-keys.json | jq -r '.secretSeed' | cut -c 3- > chainflip/keys/broker/signing_key_file
cat chainflip/lp-keys.json | jq -r '.secretSeed' | cut -c 3- > chainflip/keys/lp/signing_key_file
```

### Fund Accounts

1. Get some `tFLIP`

    a. Add the `tFLIP` token to your wallet using the following address: `0x0485D65da68b2A6b48C3fA28D7CCAce196798B94`

    b. Get in touch with us on [Discord](https://discord.com/channels/824147014140952596/1045323960339935342) and we'll send you some `tFLIP`

2. Get the public key of the Broker or LP account:
```bash
# Broker
cat chainflip/broker-keys.json | jq -r '.ss58Address'

# LP
cat chainflip/lp-keys.json | jq -r '.ss58Address'
```

3. Then head to the [Auctions Web App](https://auctions-perseverance.chainflip.io/nodes)
4. Connect your wallet
5. Click "Add Node"
6. Follow the instructions to fund the account

### Running the APIs

```bash
docker-compose up -d
```

### Interacting with the APIs
You can connect to your local RPC using [PolkadotJS](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer) to see chain events.
> Note: The following commands take a little while to respond because it submits and waits for finality.
#### Broker

Register a broker account:

```bash
curl -H "Content-Type: application/json" \
    -d '{"id":1, "jsonrpc":"2.0", "method": "broker_registerAccount"}' \
    http://localhost:10997
```

Request a swap deposit address:

```bash
curl -H "Content-Type: application/json" \
    -d '{"id":1, "jsonrpc":"2.0", "method": "broker_requestSwapDepositAddress", "params": ["Eth", "Flip","0xabababababababababababababababababababab", 0]}' \
    http://localhost:10997
```

#### LP

Register a broker account:

```bash
curl -H "Content-Type: application/json" \
    -d '{"id":1, "jsonrpc":"2.0", "method": "lp_registerAccount", "params": [0]}' \
    http://localhost:10589
```

Request a liquidity deposit address:

```bash
curl -H "Content-Type: application/json" \
    -d '{"id":1, "jsonrpc":"2.0", "method": "lp_liquidityDeposit", "params": ["Eth"]}' \
    http://localhost:10589
```
