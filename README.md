# Multi-chains SubQuery

The multi-chains [SubQuery](https://subquery.network) project for NFT2.0 protocol. This includes indexer for all protocol on-chain events, including but not limited to:
> 1. Factory events: create data-registries, collections, derived-accounts
> 2. Data-registry events: write, derive.
> 3. Collection events: mint, burn, transfer.
> 4. Derived-account events: claim royalty.
> 5. Addons events: addons created.

## Prerequisites
- [NodeJS v20 LTS](https://nodejs.org/en/blog/release/v20.9.0)
- [Docker LTS](https://docs.docker.com/engine/install/)
- [Docker Compose Plugin LTS](https://docs.docker.com/compose/install/linux/)
- [SubQuery CLI](https://academy.subquery.network/quickstart/quickstart.html#_1-install-the-subquery-cli)

## Setup
- Install dependencies
```bash
$ yarn
```

- Create .env file of special chain from template. Ex for **avax**:
```bash
$ cp .env.example .env.avax
```

- Populate contract address, block number, etc to `.env.avax` file

## Build
- Generate file yaml and all manifest for a chain:
```bash
$ CHAIN=<chain> yarn codegen -f project-multichain.ts && mv project-multichain.yaml project-<chain>.yaml
```

> CHAIN env var can be one of supported chains, including: *dera-testnet, avax, bnb, avax-testnet, bnb-testnet*
>
> Eg., command for `avax`:
```bash
$ CHAIN="avax" yarn codegen -f project-multichain.ts && mv project-multichain.yaml project-avax.yaml
```

> *This command will also generate types manifest in /src/types folder*

- Compile code

```bash
$ yarn build -f subquery-multichain.yaml
```

> *This command will compile TS code and store artifacts to /dist folder, every update logic, including configuration in project-multichain.ts would require to reexecute this command*

## Run locally
- Spin up infrastructure stack, which includes PostgreSQL, Subquery node, GraphQL engine by using docker compose
```bash
$ yarn start:docker
```

- Access [GraphQL playground local](http://localhost:3000/) to query

## Test
- Execute UT
```bash
$ yarn test
```

## Query
- Via [GraphQL playground local](http://localhost:3000/)

```graphql
query {
  collections(
    first: 5
    orderBy: [BLOCK_HEIGHT_DESC]
    filter: { chainId: { equalTo: 20240801 } }
  ) {
    nodes {
      chainId
      id
      blockHeight
      address
    }
  }
}
```

The result should look something like this:

```json
{
  "data": {
    "collections": {
      "nodes": [
        {
          "chainId": 20240801,
          "id": "0xc477ef1211e3c20f9069b1d159a88ae2a8024978",
          "blockHeight": "1460",
          "address": "0xc477ef1211e3c20f9069b1d159a88ae2a8024978"
        },
        {
          "chainId": 20240801,
          "id": "0x97a50967fd59b2aab8841cff7304f77f74e22dcf",
          "blockHeight": "1459",
          "address": "0x97a50967fd59b2aab8841cff7304f77f74e22dcf"
        },
        {
          "chainId": 20240801,
          "id": "0x17e5896d4fdb55c79481cb96e07b01dfda0f1382",
          "blockHeight": "212",
          "address": "0x17e5896d4fdb55c79481cb96e07b01dfda0f1382"
        },
        {
          "chainId": 20240801,
          "id": "0x7136c629e76c0dcad52c48bfa41ad35c46aeecf4",
          "blockHeight": "187",
          "address": "0x7136c629e76c0dcad52c48bfa41ad35c46aeecf4"
        }
      ]
    }
  }
}
```

- Via cURL (after publishing project to Managed Service)
```bash
$ curl -g -X POST -H "Content-type: application/json" \
-d '{"query": "query{collections(first:5) {nodes {chainId id blockHeight address}}}"}' \
<subquery-query-url>
```

## Publish
- Prerequisites: step [Build](#build) must be done before doing publishment.

- Publish project manifest to IPFS

> Mainnets
```bash
$ SUBQL_ACCESS_TOKEN="<access-token>" subql publish -f subquery-multichain.yaml
```

> Testnets
```bash
$ SUBQL_ACCESS_TOKEN="<access-token>" subql publish -f subquery-multichain-testnet.yaml
```

> *The IPFS CID will be saved in .project-cid file in root folder*

- Deployment can be done either via Console or SubQL CLI
```bash
$ SUBQL_ACCESS_TOKEN="<access-token>" subql deployment:deploy -d \
--org "<your-org>" \
--projectName "<project-name>" \
--endpoint "<rpc-endpoint>" \
--ipfsCID "<ipfs-cid>"
```

## Cleanup
- Down Docker compose
```bash
$ docker-compose down
```

- Cleanup PostgreSQL data
```bash
$ sudo rm -rf ./.data/
```

- Cleanup compiled source
```bash
$ sudo rm -rf ./dist/
$ sudo rm -rf ./src/types/
```

- Delete deployment on Managed service
```bash
$ SUBQL_ACCESS_TOKEN="<access-token>" subql deployment:delete -d \
--org "<your-org>" \
--project_name "<project-name>" \
--deploymentID "<deployment-id>"
```

## Troubleshoot
- Subquery-node unhealthy upon docker-compose start, it might occur due to RPC endpoint rate limit, which leads to failure on sync up latest chain status, the solution for this issue is simply waiting for RPC connection success, by monitoring subquery-node logs
```bash
$ docker logs -f <subquery-node-container>
```

## License
Copyright belongs to [DERA chain](https://derachain.com), 2023-2024.

