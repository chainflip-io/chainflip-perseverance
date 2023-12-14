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
docker run --platform=linux/amd64 --entrypoint=/usr/local/bin/chainflip-cli chainfliplabs/chainflip-cli:perseverance generate-keys --json > chainflip/lp-keys.json
docker run --platform=linux/amd64 --entrypoint=/usr/local/bin/chainflip-cli chainfliplabs/chainflip-cli:perseverance generate-keys --json > chainflip/broker-keys.json
cat chainflip/broker-keys.json | jq -r '.signing_key.secret_key' > chainflip/keys/broker/signing_key_file
cat chainflip/lp-keys.json | jq -r '.signing_key.secret_key' > chainflip/keys/lp/signing_key_file
```

### Fund Accounts

1. Get some `tFLIP`

    a. Add the `tFLIP` token to your wallet using the following address: `0x0485D65da68b2A6b48C3fA28D7CCAce196798B94`

    b. Get in touch with us on [Discord](https://discord.com/channels/824147014140952596/1045323960339935342) and we'll send you some `tFLIP`

2. Get the public key of the Broker or LP account:
```bash
# Broker
cat chainflip/broker-keys.json | jq -r '.signing_account_id'

# LP
cat chainflip/lp-keys.json | jq -r '.signing_account_id'
```

3. Then head to the [Auctions Web App](https://auctions-perseverance.chainflip.io/nodes)
4. Connect your wallet
5. Click "Add Node"
6. Follow the instructions to fund the account

### Running the APIs

#### Important Note

> 💡 Note: By default, the Node, LP and Broker APIs accept connection from any host. This is intentional to make testing easier. However, if you wish to make this more secure, feel free to update the ports in the `docker-compose.yml` file to only accept connections from `localhost`.

> This can be achieved by adding `127.0.0.1:`   before the port number. For example:
```yaml
  lp:
    image: chainfliplabs/chainflip-lp-api:perseverance
    pull_policy: always
    stop_grace_period: 5s
    stop_signal: SIGINT
    platform: linux/amd64
    restart: unless-stopped
    ports:
      - "127.0.0.1:10589:80"
    volumes:
      - ./chainflip/keys/lp:/etc/chainflip/keys
    entrypoint:
      - /usr/local/bin/chainflip-lp-api
    command:
      - --state_chain.ws_endpoint=ws://node:9944
    depends_on:
      - node
```

#### Starting the Node and APIs
Start by starting the node and wait for it to sync:
```bash
docker compose up node -d
docker compose logs -f
```
> 💡 Note: You know that your node is synced once you start seeing logs similar to the following:

```log
chainflip-perseverance-node-1  | 2023-10-13 16:02:00 ✨ Imported #614404 (0x990b…be63)
```

Once the node is synced you can start the APIs:
```bash
docker compose up -d
docker compose logs -f
```

If you want to only start the Broker API, you can run:
```bash
docker compose up -d broker
docker compose logs -f broker
```

If you want to only start the LP API, you can run:
```bash
docker compose up -d lp
docker compose logs -f lp
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
    -d '{"id":1, "jsonrpc":"2.0", "method": "broker_requestSwapDepositAddress", "params": ["ETH", "FLIP","0xabababababababababababababababababababab", 0]}' \
    http://localhost:10997
```

#### LP

Register a broker account:

```bash
curl -H "Content-Type: application/json" \
    -d '{"id":1, "jsonrpc":"2.0", "method": "lp_register_account", "params": [0]}' \
    http://localhost:10589
```

Request a liquidity deposit address:

```bash
curl -H "Content-Type: application/json" \
    -d '{"id":1, "jsonrpc":"2.0", "method": "lp_liquidity_deposit", "params": ["ETH"]}' \
    http://localhost:10589
```

For more details please refer to the [Integrations documentation](https://docs.chainflip.io/integration/liquidity-provision/lp-api).
