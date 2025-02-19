# VeChain Rosetta Data API

Implementing Rosetta Data API of VeChainThor BlockChain

[![](https://badgen.net/badge/VeChainThorNode/>=2.0.1)]()
[![](https://badgen.net/badge/Network/main,test?list=|)]()
[![](https://badgen.net/badge/Rosetta/=1.4.12)]()

[![](https://badgen.net/badge/node/>=16.6)]()
[![](https://badgen.net/badge/typescript/>=4.7.4?icon=typescript&label)]()
[![](https://badgen.net/badge/docker/>=20.10.17?icon=docker&label)]()
[![](https://badgen.net/badge/Status/testing/orange)]()

## Installation

### Getting the source

- Clone the repo

``` shell
    git clone https://github.com/vechain/rosetta.git
    cd rosetta
```

### Building

- Building Docker

``` shell
    docker build ./ -t vechain/rosetta-server:latest
```

### Docker

``` sh
    docker run -d\
    -v {path-to-data-directory}:/data\
    -p {host_address_port}:8080 -p {host_address_port}:8669 -p 11235:11235 -p 11235:11235/udp\
    --env NETWORK={network_type} --env MODE={run_mode}\
    vechain/rosetta-server:latest
```

- `path-to-data-directory` directory for data
- `host_address_port` rosetta api service listening address
- `network_type` rosetta and thornode to join network type (main|test)
- `run_mode` the api service run mode (online|offline),if the mode is offline, some apis can not be used.

#### Example

``` shell
    docker run -d\
    -v /Users/rosetta/data/:/data\
    -p 0.0.0.0:8080:8080 -p 0.0.0.0:8669:8669 -p 11235:11235 -p 11235:11235/udp\
    --env NETWORK=main --env MODE=online\
    vechain/rosetta-server:latest
```

## Endpoints

### Account

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /account/balance | Yes | Get an Account Balance | online
POST | /account/coins | No |

### Block

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /block | Yes | Get a Block | online
POST | /block/transaction | Yes | Get a Block Transaction | online

### Call

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /call | No | |

### Construction

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /construction/combine | Yes | Create Network Transaction from Signatures | online & offline
POST | /construction/derive | Yes | Derive an AccountIdentifier from a PublicKey | online & offline
POST | /construction/hash | Yes | Get the Hash of a Signed Transaction | online & offline
POST | /construction/metadata | Yes | Get Metadata for Transaction Construction | online
POST | /construction/parse | Yes | Parse a Transaction | online & offline
POST | /construction/payloads | Yes | Generate an Unsigned Transaction and Signing Payloads | online & offline
POST | /construction/preprocess | Yes | Create a Request to Fetch Metadata | online & offline
POST | /construction/submit | Yes | Submit a Signed Transaction | online

### Events

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /events/blocks | Yes | [INDEXER] Get a range of BlockEvents

### Mempool

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /construction/metadata | No | Node API no support
POST | /construction/submit | No | Node API no support

### Network

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /network/list | Yes | Get List of Available Networks | online & offline
POST | /network/options | Yes | Get Network Options | online & offline
POST | /network/status | Yes | Get Network Status | online

### Search

Method| Endpoint | Implemented | Description | Mode
---------|----------|---------|---------|---------
POST | /search/transactions | Yes | [INDEXER] Search for Transactions | online

## About Fee Delegation (VIP191)

- VeChain Rosetta implement support Fee Delegation (**[VIP-191](https://github.com/vechain/VIPs/blob/master/vips/VIP-191.md)**).

### Why exchanges need VIP191

- The exchange will transfer currencies from hot wallet to the safe cold wallet from time to time. When there's not enough VTHO in the hot wallet or want to transfer all currencies includ VTHO, the exchange can use the exchange can use VIP191 function to pay the transaction fee.

### How to use it

- Step1: Create a VIP191 payloads

Create a VIP191 payload, add `FeeDelegation` operation to operations, `account` is fee delegator address,`sub_account.address` is VTHO smart contract address. `amount.value` default 0 .

``` json
    {
    "network_identifier": {
        "blockchain": "vechainthor",
        "network": "test"
    },
    "operations": [
        {
            "operation_identifier": {
                "index": 1,
                "network_index": 0
            },
            "type": "Transfer",
            "status": "None",
            "account": {
                "address": "0xD3ae78222BEADB038203bE21eD5ce7C9B1BfF602"
            },
            "amount": {
                "value": "12000000000000000000",
                "currency": {
                    "symbol": "VET",
                    "decimals": 18
                }
            }
        },
        {
            "operation_identifier": {
                "index": 2,
                "network_index": 0
            },
            "type": "Transfer",
            "status": "None",
            "account": {
                "address": "0x7567D83b7b8d80ADdCb281A71d54Fc7B3364ffed"
            },
            "amount": {
                "value": "-12000000000000000000",
                "currency": {
                    "symbol": "VET",
                    "decimals": 18
                }
            }
        },
        {
            "operation_identifier": {
                "index": 3,
                "network_index": 1
            },
            "type": "Transfer",
            "status": "None",
            "account": {
                "address": "0xD3ae78222BEADB038203bE21eD5ce7C9B1BfF602",
                "sub_account": {
                    "address": "0x0000000000000000000000000000456e65726779"
                }
            },
            "amount": {
                "value": "12000000000000000000",
                "currency": {
                    "symbol": "VTHO",
                    "decimals": 18
                }
            }
        },
        {
            "operation_identifier": {
                "index": 4,
                "network_index": 1
            },
            "type": "Transfer",
            "status": "None",
            "account": {
                "address": "0x7567D83b7b8d80ADdCb281A71d54Fc7B3364ffed",
                "sub_account": {
                    "address": "0x0000000000000000000000000000456e65726779"
                }
            },
            "amount": {
                "value": "-12000000000000000000",
                "currency": {
                    "symbol": "VTHO",
                    "decimals": 18
                }
            }
        },
        {
            "operation_identifier": {
                "index": 5,
                "network_index": 2
            },
            "type": "FeeDelegation",
            "status": "None",
            "account": {
                "address": "0xd3ae78222beadb038203be21ed5ce7c9b1bff602",
                "sub_account": {
                    "address": "0x0000000000000000000000000000456e65726779"
                }
            },
            "amount": {
                "value": "0",
                "currency": {
                    "symbol": "VTHO",
                    "decimals": 18
                }
            }
        }
    ],
    "metadata": {
        "chainTag":"0x27",
        "blockRef":"0x00663ef78585491f"
    }
}
```

- Step2: Call /construction/payloads

When calling /construction/payloads use VIP191 payloads, the api will return two payload infomations. The first payload is the transaction origin infomation, the second payload is fee delegator infomation. The api will auto calculate transaction fee add the unsigned_transaction raw.

``` json
    {
        "unsigned_transaction": "0xf89c2787663ef78585491f81b4f87edf94d3ae78222beadb038203be21ed5ce7c9b1bff60288a688906bd8b0000080f85c940000000000000000000000000000456e6572677980b844a9059cbb000000000000000000000000d3ae78222beadb038203be21ed5ce7c9b1bff602000000000000000000000000000000000000000000000000a688906bd8b0000080830101d08088a363332020238c1dc101",
        "payloads": [
            {
                "address": "0x7567D83b7b8d80ADdCb281A71d54Fc7B3364ffed",
                "hex_bytes": "0xf184335272ea0b534e653844d7c79d3de78902c41f98d86418028871c2c46b43",
                "signature_type": "ecdsa_recovery"
            },
            {
                "address": "0xd3ae78222beadb038203be21ed5ce7c9b1bff602",
                "hex_bytes": "0x7573de904a86cf9d78147434a0caa3db0f68625e52a9907d4268f92d07acce64",
                "signature_type": "ecdsa_recovery"
            }
        ]
    }
```
  
