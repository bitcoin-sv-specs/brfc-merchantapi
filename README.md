## RFC Notice

ReadMe version 1.4.9-g.

This draft spec is released as an RFC (request for comment) as part of the public review process. Any comments, criticisms or suggestions should be directed toward the [issues page](https://github.com/bitcoin-sv-specs/brfc-merchantapi/issues) on this github repository.

A reference implementation of the Merchant API server is available [here](https://github.com/bitcoin-sv/merchantapi-reference).

# mAPI Specification

|     BRFC     |    title     | authors | version |
| :----------: | :----------: | :-----: | :-----: |
| 8eeed98377bd TODO | mAPI         | nChain  |   1.5.0 |

## Dependency
mAPI v1.5.0 requires BSV Node v1.0.10 or later.

## Overview

Merchant API (mAPI) is an additional service that miners can offer to merchants.
It enables merchants to get policy and fee quotes for submitting transactions, submit the transaction and query the transaction status.

### Data Flow Diagram

![data flow](out/mapi-bip270/data_flow.png)

> Note: This protocol uses the [JSON envelopes BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) as well as the [Fee Spec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec).
> Note: The example JSON below illustrates the syntax and typical data
---

## Implementation

The **REST API** has these endpoints:

1. [Get policy quote](#1-get-policy-quote)
2. [Get fee quote](#2-get-fee-quote)
3. [Submit transaction](#3-submit-transaction)
4. [Query transaction status](#4-query-transaction-status)
5. [Submit multiple transactions](#5-submit-multiple-transactions)
6. [Query transaction outputs](#6-query-transaction-outputs)


### 1. Get policy quote

#### Purpose:

This endpoint is used to get the different policies quoted by a miner. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the policies used by a specific BSV miner. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.
This is a superset of the fee quote service, as it also includes information on DSNT IP addresses and miner policies.

#### Request:

```
GET /mapi/policyQuote
```

#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.4.0\",\"timestamp\":\"2021-11-12T13:17:47.7498672Z\",\"expiryTime\":\"2021-11-12T13:27:47.7498672Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"45628be2fe616167b7da399ab63455e60ffcf84147730f4af4affca90c7d437e\",\"currentHighestBlockHeight\":234,\"fees\":[{\"feeType\":\"standard\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}},{\"feeType\":\"data\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}}],\"callbacks\":[{\"ipAddress\":\"123.456.789.123\"}],\"policies\":{\"skipscriptflags\":[\"MINIMALDATA\",\"DERSIG\",\"NULLDUMMY\",\"DISCOURAGE_UPGRADABLE_NOPS\",\"CLEANSTACK\"],\"maxtxsizepolicy\":99999,\"datacarriersize\":100000,\"maxscriptsizepolicy\":100000,\"maxscriptnumlengthpolicy\":100000,\"maxstackmemoryusagepolicy\":10000000,\"limitancestorcount\":1000,\"limitcpfpgroupmemberscount\":10,\"acceptnonstdoutputs\":true,\"datacarrier\":true,\"dustrelayfee\":150,\"maxstdtxvalidationduration\":99,\"maxnonstdtxvalidationduration\":100,\"dustlimitfactor\":10}}",
    "signature": "30440220708e2e62a393f53c43d172bc1459b4daccf9cf23ff77cff923f09b2b49b94e0a022033792bee7bc3952f4b1bfbe9df6407086b5dbfc161df34fdee684dc97be72731",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a compressed JSON format  |
| `signature` | signature on the payload string or _null_       |
| `publicKey` | public key to verify signature or _null_    |
| `encoding`  | encoding type for the payload                              |
| `mimetype`  | multipurpose Internet Mail Extensions type for the payload |

##### Expanded Payload:

```json
{
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-12T13:17:47.7498672Z",
    "expiryTime": "2021-11-12T13:27:47.7498672Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "currentHighestBlockHash": "45628be2fe616167b7da399ab63455e60ffcf84147730f4af4affca90c7d437e",
    "currentHighestBlockHeight": 234,
    "fees": [
        {
            "feeType": "standard",
            "miningFee": {
                "satoshis": 500,
                "bytes": 1000
            },
            "relayFee": {
                "satoshis": 250,
                "bytes": 1000
            }
        },
        {
            "feeType": "data",
            "miningFee": {
                "satoshis": 500,
                "bytes": 1000
            },
            "relayFee": {
                "satoshis": 250,
                "bytes": 1000
            }
        }
    ],
    "callbacks": [
        {
            "ipAddress": "123.456.789.123"
        }
    ],
    "policies": {
        "skipscriptflags": [ "MINIMALDATA", "DERSIG", "NULLDUMMY", "DISCOURAGE_UPGRADABLE_NOPS", "CLEANSTACK" ],
        "maxtxsizepolicy": 99999,
        "datacarriersize": 100000,
        "maxscriptsizepolicy": 100000,
        "maxscriptnumlengthpolicy": 100000,
        "maxstackmemoryusagepolicy": 10000000,
        "limitancestorcount": 1000,
        "limitcpfpgroupmemberscount": 10,
        "acceptnonstdoutputs": true,
        "datacarrier": true,
        "dustrelayfee": 150,
        "maxstdtxvalidationduration": 99,
        "maxnonstdtxvalidationduration": 100,
        "dustlimitfactor": 10 
    }
}
```
> Note: BSV Node v1.0.11 does not support "dustlimitfactor" and "dustrelayfee" policies, so they should not have been configured and will not be contained within the response

| field                       | function                                                                                     |
| --------------------------- | -------------------------------------------------------------------------------------------- |
| `apiVersion`                | version of the merchant API specification used                                                                 |
| `timestamp`                 | timestamp of the payload document                                                                |
| `expiryTime`                | expiry time of quote                                                                         |
| `minerId`                   | minerID / public key of miner or _null_                                           |
| `currentHighestBlockHash`   | hash of the current blockchain tip                                                               |
| `currentHighestBlockHeight` | height of the current blockchain tip                                                               |
| `fees`          | fees charged by the miner (see [feeSpec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec))  |
| `callbacks`     | IP addresses of double-spend notification servers such as mAPI reference implementation |
| `policies`      | values of miner policies as configured by the mAPI reference implementation administrator (examples above) |

### 2. Get fee quote

#### Purpose:

This endpoint is used to get the different fees quoted by a miner. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the fees charged by a specific BSV miner. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.
This is a subset of the policy quote service.

#### Request:

```
GET /mapi/feeQuote
```

#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.4.0\",\"timestamp\":\"2021-11-12T13:17:47.7498672Z\",\"expiryTime\":\"2021-11-12T13:27:47.7498672Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"45628be2fe616167b7da399ab63455e60ffcf84147730f4af4affca90c7d437e\",\"currentHighestBlockHeight\":234,\"fees\":[{\"feeType\":\"standard\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}},{\"feeType\":\"data\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}}]}",
    "signature": "30440220708e2e62a393f53c43d172bc1459b4daccf9cf23ff77cff923f09b2b49b94e0a022033792bee7bc3952f4b1bfbe9df6407086b5dbfc161df34fdee684dc97be72731",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

The field definitions are the same as policy quote, above.

##### Expanded Payload:

```json
{
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-12T13:17:47.7498672Z",
    "expiryTime": "2021-11-12T13:27:47.7498672Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "currentHighestBlockHash": "45628be2fe616167b7da399ab63455e60ffcf84147730f4af4affca90c7d437e",
    "currentHighestBlockHeight": 234,
    "fees": [
        {
            "feeType": "standard",
            "miningFee": {
                "satoshis": 500,
                "bytes": 1000
            },
            "relayFee": {
                "satoshis": 250,
                "bytes": 1000
            }
        },
        {
            "feeType": "data",
            "miningFee": {
                "satoshis": 500,
                "bytes": 1000
            },
            "relayFee": {
                "satoshis": 250,
                "bytes": 1000
            }
        }
    ]
}
```

The field definitions are the same as policy quote, above.

### 3. Submit transaction

#### Purpose:

This endpoint is used to send a raw transaction to a miner for inclusion in the next block that the miner creates. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the response to the transaction submission. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:

```
POST /mapi/tx
```

##### JSON Body:

Set `Content-Type` to `application/json`:

```json
{
    "rawtx": "[transaction_hex_string]",
    "callBackUrl": "https://your.service.callback/endpoint",
    "callBackToken": "<channel token>",
    "merkleProof": true,
    "merkleFormat": "TSC",
    "dsCheck": true,
    "callBackEncryption": "<parameter>"
}
```
##### Binary Data

Set `Content-Type` to `application/octet-stream`.

Submit the transaction with the binary serialized transaction in the request body. You can specify the fields in the query string.

For large transactions, binary is half the size of the hexadecimal equivalent although this gain is largely minimized through the use of gzip encoding of the hex data.

| field                       | function            |
| ----------------------------|---------------------|
| `rawtx`         | Hex encoded transaction   |
| `callbackURL`   | HTTP(S) endpoint used to receive messages from the miner   |
| `callbackToken` | HTTP authorization header used when authenticating against callbackURL |
| `merkleProof`   | used to request a Merkle proof    |
| `merkleFormat`  | (optional) returns TSC compliant Merkle proof format if set to "TSC"   |
| `dsCheck`       | used to request double spend notification  |
| `callbackEncryption`   | (optional) parameter to encrypt callback data     |


*Note:*  In mAPI 1.4.0, the supported encryption method is libsodium sealed_box which is an anonymous (you can not identify the sender) public key encryption with integrity check (for more details see: https://libsodium.gitbook.io/doc/public-key_cryptography/sealed_boxes)

The format of the callbackEncryption parameter is:

    libsodium sealed_box <base64 encoded encryption key>


#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.4.0\",\"timestamp\":\"2021-11-13T07:37:44.8783319Z\",\"txid\":\"fed22f5ab54202e2ec39cb745d427fcfff960254cde0cf283493ac545f5737f6\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"39e3a2a0e7ba1b9e331cfd396cef1a2d3baffa51624af2f5512e530f35a8aa43\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0}",
    "signature": "30440220160ff70b73297043a8ce9636f5abdc7d91918b4428e79876a405b080882d1c920220161b919399f50f16d8c9f922c13b94909d6e3a25480f09812c73f9ef52f8f542",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

The fields are specified above.

##### Expanded Payload:

```json
{
    "apiVersion": "1.5.0",
    "timestamp": "2022-11-13T07:37:44.8783319Z",
    "txid": "fed22f5ab54202e2ec39cb745d427fcfff960254cde0cf283493ac545f5737f6",
    "returnResult": "success",
    "resultDescription": "",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "currentHighestBlockHash": "39e3a2a0e7ba1b9e331cfd396cef1a2d3baffa51624af2f5512e530f35a8aa43",
    "currentHighestBlockHeight": 151,
    "txSecondMempoolExpiry" : 0,
    "warnings": "",
    "failureRetryable": true,
    "conflictedWith"" : ""
}
```

| field                   | function       |
| ----------------------- | -------------- |
| `txid`                  | transaction ID |
| `returnResult`          | will contain either `success` or `failure` |
| `resultDescription`     | will contain the error on `failure` or empty on `success`|
| `txSecondMempoolExpiry` | duration (minutes) Tx will be kept in the secondary mempool |
| `warnings`              | warnings provided by the system |
| `failureRetryable`      | if true indicates that the transaction may be resubmitted later |
| `conflictedWith`        | list of double spend transactions |


If a double spend notification or Merkle proof is requested in Submit transaction, the Merkle proof or double spend notification will be sent to the specified callbackURL. Where recipients are using [SPV Channels](https://github.com/bitcoin-sv-specs/brfc-spvchannels), this would require the recipient to have a channel set up and ready to receive messages. See [Callback Notifications](#callback-notifications) for details.

#### Callback Reason

There is an option for the miner to provide a callback reason using `{callbackReason}` to enable client applications to determine what type of callback is returned, for example, Merkle proof or double spend.

#### Request:
```json
{
    "rawTx": "0100000001b753fbcb4e99659468067c2512b64d80c593bf46d4b60f750dd77c59391c4210000000006a473044022000cc88f3feadbacfd93e2a1a723e4fa4a20ef329ab5daac3be962d973bee3fb5022031642f58b5fce72e531f9dcae49e74be95cdb2f59a312865517fa536581d85584121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000",
    "callbackUrl": "https://your.service.callback/endpoint/{callbackReason}",
    "callbackToken": "<channel token>",
    "merkleProof": true,
    "merkleFormat": "TSC",
    "dsCheck": true,
    "callbackEncryption": "<parameter>"
}
```
#### Response:

An example TSC compliant Merkle proof callback, which will be sent to `https://your.service.callback/endpoint/merkleProof`:
```json
{
    "callbackPayload": "{\"index\":1,\"txOrId\":\"e7b3eefab33072e62283255f193ef5d22f26bbcfc0a80688fe2cc5178a32dda6\",\"targetType\":\"header\",\"target\":\"00000020a552fb757cf80b7341063e108884504212da2f1e1ce2ad9ffc3c6163955a27274b53d185c6b216d9f4f8831af1249d7b4b8c8ab16096cb49dda5e5fbd59517c775ba8b60ffff7f2000000000\",\"nodes\":[\"30361d1b60b8ca43d5cec3efc0a0c166d777ada0543ace64c4034fa25d253909\",\"e7aa15058daf38236965670467ade59f96cfc6ec6b7b8bb05c9a7ed6926b884d\",\"dad635ff856c81bdba518f82d224c048efd9aae2a045ad9abc74f2b18cde4322\",\"6f806a80720b0603d2ad3b6dfecc3801f42a2ea402789d8e2a77a6826b50303a\"]}",
    "apiVersion": "1.4.0",
    "timestamp": "2021-04-30T08:06:13.4129624Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "blockHash": "2ad8af91739e9dc41ea155a9ab4b14ab88fe2a0934f14420139867babf5953c4",
    "blockHeight": 105,
    "callbackTxId": "e7b3eefab33072e62283255f193ef5d22f26bbcfc0a80688fe2cc5178a32dda6",
    "callbackReason": "merkleProof"
}
```

### 4. Query transaction status

#### Purpose:
This endpoint is used to check the current status of a previously submitted transaction. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the transaction status. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:
```
GET /mapi/tx/{txid}?merkleProof=bool&merkleFormat=TSC
```

| parameter | description |
| ----------| ----------- |
| `txid` | hex encoded transaction ID |
| `merkleProof` | optional specifying whether any available Merkle proof should be provided – default is false |
| `merkleFormat` | optional specifying the format of any Merkle proof returned – default is the recommended [TSC](https://tsc.bitcoinassociation.net/standards/merkle-proof-standardised-format/) |

#### Response:

An example response with TSC compliant Merkle proof requested:
```json
{
  "payload": "{\"apiVersion\":\"1.5.0\",\"timestamp\":\"2022-10-14T07:37:32.8272043Z\",\"txid\":\"1aa420c0266f70c4e48440c23f3f4154ea80ab58d82177583fbbf9e661dbc0db\",\"returnResult\":\"success\",\"blockHash\":\"4da321c1550c35e13b5ba6ea252d32bc3d31d2d51d6182a1afdbc6d8492497e1\",\"blockHeight\":324,\"confirmations\":1,\"minerId\":\"029d809c6e2c400dab4e65f1a4bc3749876b905651348ebc1629798ae05fba9da1\",\"txSecondMempoolExpiry\":0,\"merkleProof\":{\"index\":2,\"txOrId\":\"1aa420c0266f70c4e48440c23f3f4154ea80ab58d82177583fbbf9e661dbc0db\",\"targetType\":\"header\",\"target\":\"0000002069191183f973fdb08b9880f7797bfcee39b7e5be179bb5f28d48c665095a1a51e6439c79ba1446eeae0cf1d5419acca760adce07b3bfba912f37bf828d06c8bafe104963ffff7f2000000000\",\"nodes\":[\"6969cfc4ca0956afb5bc06d9ad7595dc3ebbcf533bbfbf3d0901d81ee94f3826\",\"de34519da2777febd98027347a439bf83c9011177cef85116bdb66f609f94214\"]}}",
  "signature": "304402207b92df19b708ffb55581b1cc401ad6ca044d485059520d6832b68feb6cc7f6cd0220553e50bfdacd0d9dec030a33faef6d3f18850c9c098324dcc6ceed8af81d0841",
  "publicKey": "029d809c6e2c400dab4e65f1a4bc3749876b905651348ebc1629798ae05fba9da1",
  "encoding": "UTF-8",
  "mimetype": "application/json"
}
```

The fields are specified above.

##### Expanded Payload:

```json
{
  "apiVersion": "1.5.0",
  "timestamp": "2022-10-14T07:37:32.8272043Z",
  "txid": "1aa420c0266f70c4e48440c23f3f4154ea80ab58d82177583fbbf9e661dbc0db",
  "returnResult": "success",
  "blockHash": "4da321c1550c35e13b5ba6ea252d32bc3d31d2d51d6182a1afdbc6d8492497e1",
  "blockHeight": 324,
  "confirmations": 1,
  "minerId": "029d809c6e2c400dab4e65f1a4bc3749876b905651348ebc1629798ae05fba9da1",
  "txSecondMempoolExpiry":0,
  "merkleProof": {
    "index": 2,
    "txOrId": "1aa420c0266f70c4e48440c23f3f4154ea80ab58d82177583fbbf9e661dbc0db",
    "targetType": "header",
    "target": "0000002069191183f973fdb08b9880f77...0bf828d06c8bafe104963ffff7f2000000000",
    "nodes": [
      "6969cfc4ca0956afb5bc06d9ad7595dc3ebbcf533bbfbf3d0901d81ee94f3826",
      "de34519da2777febd98027347a439bf83c9011177cef85116bdb66f609f94214"
    ]
  }
}
```
The Merkle proof is provided and is compliant with the [TSC Specification](https://tsc.bitcoinassociation.net/standards/merkle-proof-standardised-format/).

#### Alternative Response:

```json
{
    "payload": "{\"apiVersion\":\"1.4.0\",\"timestamp\":\"2021-11-13T07:48:49.7666999Z\",\"txid\":\"6886cd04977d4cd26df3689b2d3c40b13685edb41fcc21c2962c5fc64560acff\",\"returnResult\":\"success\",\"blockHash\":\"236337115fef235a5d59cfcdf213cd70a86d6249a10eacf041468f15607094de\",\"blockHeight\":152,\"confirmations\":1,\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"txSecondMempoolExpiry\":0}",
    "signature": "3044022039b759b4e80f13e1c91b40a42ac777704d3866b2f24432f57236b44fdfae13fe0220544a3853eb8d58f973a14dbec7e16c56a8242a8e68722102ed70de79a2b1455e",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

The fields are specified above.

##### Expanded Payload:

```json
{
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-13T07:48:49.7666999Z",
    "txid": "6886cd04977d4cd26df3689b2d3c40b13685edb41fcc21c2962c5fc64560acff",
    "returnResult": "success",
    "blockHash": "236337115fef235a5d59cfcdf213cd70a86d6249a10eacf041468f15607094de",
    "blockHeight": 152,
    "confirmations": 1,
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "txSecondMempoolExpiry": 0
}
```

| field                   | function                                                |
| ----------------------- | ------------------------------------------------------- |
| `blockHash`             | hash of tx block                                        |
| `blockHeight`           | height of tx block                                      |
| `confirmations`         | number of block confirmations                           |

### 5. Submit multiple transactions

#### Purpose:

This endpoint is used to send multiple raw transactions to a miner for inclusion in the next block that the miner creates. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the responses to the transaction submissions. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:

```
POST /mapi/txs
```

##### JSON Body:

Where `Content-Type` is `application/json`.

```json
[
    {
        "a transaction as above"
    },
    {
        "any number of additional transactions"
    }
]
```

You can also omit *callbackUrl*, *callbackToken*, *merkleProof*,*merkleFormat* and *dsCheck* from the request body and provide the default values in the query string.

#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.4.0\",\"timestamp\":\"2021-11-13T08:31:56.5722511Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"08dc4bb006fc7e7186544343c3ccbb5a773d0a19cd2ccff1fa52f51eb6faf2ab\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0,\"txs\":[{\"txid\":\"3145011f34a00d0666ea265b87c8e44108f87d3b53b853976906519ee8e1475f\",\"returnResult\":\"failure\",\"resultDescription\":\"Missing inputs\",\"conflictedWith\":[{\"txid\":\"86e1b384d3d169fd6aa4d34cf2d6f487436da54154befaab5a1fb25f844d65a8\",\"size\":191,\"hex\":\"01000000010136836d73f29cbe648bc2aeea20286502a3c2f2d3cff54522d0cc76bb755e9f000000006a4730440220761fb63128d4184fc142f2e854c499c52422db0136191f29f0bbe0969b6021770220536d72606d49dbbd244d2633b8b19031234138f045c530cc773e6e72bb34c62c4121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}]},{\"txid\":\"c8a087b1ee775fa29697511ecd64e800941c8a22db6ed0989fb27a1d2d6798da\",\"returnResult\":\"success\",\"resultDescription\":\"\"}],\"failureCount\":1}",
    "signature": "304402200c4b0dc179906581eb32953abeddbef5799d302d82367aa9a469d79c15f932f3022029e827af6122290d5e1b80c50676b0336f4c7658ca67ba4819396dff9c6239a6",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

The fields are specified above.

##### Expanded Payload:

```json
{
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-13T08:31:56.5722511Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "currentHighestBlockHash": "08dc4bb006fc7e7186544343c3ccbb5a773d0a19cd2ccff1fa52f51eb6faf2ab",
    "currentHighestBlockHeight": 151,
    "txSecondMempoolExpiry": 0,
    "txs": [
        {
            "txid": "3145011f34a00d0666ea265b87c8e44108f87d3b53b853976906519ee8e1475f",
            "returnResult": "failure",
            "resultDescription": "Missing inputs",
            "conflictedWith": [
                {
                    "txid": "86e1b384d3d169fd6aa4d34cf2d6f487436da54154befaab5a1fb25f844d65a8",
                    "size": 191,
                    "hex": "01000000010136836d73f29cbe648bc2aeea20286502a3c2f2d3cff54522d0cc76bb755e9f000000006a4730440220761fb63128d4184fc142f2e854c499c52422db0136191f29f0bbe0969b6021770220536d72606d49dbbd244d2633b8b19031234138f045c530cc773e6e72bb34c62c4121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000"
                }
            ]
        },
        {
            "txid": "c8a087b1ee775fa29697511ecd64e800941c8a22db6ed0989fb27a1d2d6798da",
            "returnResult": "success",
            "resultDescription": ""
        }
    ],
    "failureCount": 1
}
```

| field          | function                                 |
| -------------- | ---------------------------------------- |
| `txs`          | list of transaction responses            |
| `failureCount` | number of failed transaction submissions |


To submit a transaction in binary format use `Content-Type: application/octet-stream` Content-Type: application/octet-stream with the binary serialized transactions in the request body. Use query string to specify the remaining parameters.

#### Double Spend Attempts

It is possible that more than one transaction in the batch attempts to spend the same input from the batch. This is a double-spend attempt. mAPI does not attempt to detect this, instead the transactions are passed to the BSV Node which will detect the double spend attempt and reject one or more transactions, in a similar manner to the response shown above. This will also happen when the inputs are a double-spend attempt from earlier transactions i.e. not just in the same batch of transactions.

### 5. Query transaction outputs

#### Purpose:
This endpoint is used to check the status of transaction outputs. This could be because an interested party wants to know their value, or some other information. 

The user may elect to search just the blockchain, or the blockchain and the mempool, for the transaction. The user may also elect to only return some information fields, or all information fields.

Multiple outputs may be queried in each request.

It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the transaction status. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:
```
POST /mapi/txouts?includeMempool=bool&returnField=confirmations&returnField=value
```

| parameter | description |
| ----------| ----------- |
| `includeMempool` | optional “search mempool” flag, in addition to always searching the blockchain – default is true |
| `returnField` | Optional return named field specifier. Any number of these parameters may be specified in the command. Possible field names are: scriptPubKey, scriptPubKeyLen, value, isStandard and confirmations. The default is all fields |

##### JSON Body:

```json
{
  [
    { "txid": "<transaction-id>", "n": 0 },
    Et cetera
  ]
}
```

| field  | function                                 |
| ------ | ---------------------------------------- |
| `txid` | transaction ID of the transaction in question  |
| `n`    | index of the output in the transaction |

#### Example Request:

```
POST /mapi/txouts?includeMempool=false
```

```json
{
  [
    { "txid": "0bc1733f05aae146c3641fd...57f60f19a430ffe867020619d54800", "n": 0 },
    { "txid": "d013adf525ed5feaffc6e9d...40566470181f099f1560343cdcfd00", "n": 0 },
    { "txid": "d013adf525ed5feaffc6e9d...40566470181f099f1560343cdcfd00", "n": 1 }
  ]
}
```

#### Example Response:

```json
{
    "payload": "{\"apiVersion\":\"1.5.0\",\"timestamp\":\"2022-10-17T05:43:12.9408765Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"returnResult\":\"success\",\"txouts\":[{\"error\":\"spent\",\"collidedWith\":{\"txid\":\"c34ef6de079a1e3c738b0681fca06257f16369766ae3c768bb179c0a85550b32\",\"size\":259,\"hex\":\"01000000010048d519060267e8ff30a4190ff635e754cc5cdf0f1964c346e1aa053f73c10b000000006a47304402201bcc6e4ca8aa2e1c7eaec7f27451b1a19cdfe5f2dd91e4f11080d6345d1a921d02206f41d8068e34287f8e4cb79a5e10f986f8507818f602f28a7d946965e3361b57412103b1fb82639861ef2a329aa3d879bc42c813df793935ad2f0196c2245240393b25ffffffff0355a8ab31000000001976a914798889cb2e7002facbb3c8367c02321cbcc3b13388ac55a8ab31000000001976a914798889cb2e7002facbb3c8367c02321cbcc3b13388ac68a7ab31000000001976a914d4bbba35bec09f4ea9a61573233a1608705a509d88ac00000000\"}},{\"scriptPubKey\":\"76a914d4bbba35bec09f4ea9a61573233a1608705a509d88ac\",\"scriptPubKeyLen\":25,\"value\":25.00000000,\"isStandard\":true,\"confirmations\":201},{\"error\":\"missing\"}]}",
    "signature": "304402207a6469871bc290c3e98a05febf7863b261c274904b8f8ea8eae1f9b18ce8253602203f5bdc8c9a6d0126f0ee8cb5c057cad9b64ea1e7df9515a98a50012f48c1c9b3",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

##### Expanded Payload:

```json
{
  "apiVersion": "1.5.0",
  "timestamp": "2022-10-17T05:43:12.9408765Z",
  "minerId": "030d1fe5c1b560efe196ba324...0daa9504c4c4cec6184fc702d9f274e",
  "returnResult": "success",
  "txouts": [
    {
      "error": "spent",
      "collidedWith": { 
        "txid": "c34ef6de079a1ev54tg...257f16369766ae3c768bb179c0a85550b32",
        "size": 259,
        "hex": "01000000010048d9060267e8ff30a4635e754c...3233a1608705a509d88ac00000000"
      }
    },
    {
      "scriptPubKey": "76a914d4bbba35bec09f9...83233a1608705a509d88ac",
      "scriptPubKeyLen": 25,
      "value": 25.00000000,
      "isStandard": true,
      "confirmations": 201
    },
    {
      "error": "missing"
    }
  ]
}
```
In this example, the 3 transaction outputs align with the 3 requests. 
The first “txouts” array entry shows that the txid collided with a different txid. 
The second entry returns all the fields (the default) about the transaction output. 
The third entry shows that the transaction output is “missing”. 
This is either because the transaction output has not reached the blockchain, or because it has been spent.

### Callback Notifications

Merchants can request callbacks for *Merkle proofs* and/or *double spend notifications* in Submit transaction.

The tool that builds the transaction, can build a special double-spend notification output according to the [DSNT specification](https://github.com/bitcoin-sv-specs/protocol/blob/master/updates/double-spend-notifications.md).
A DSNT server must be used to communicate with node. The mAPI reference implementation is a DSNT server, so the IP address of that can be used in the DSNT transaction output.
That information is made available in response to a GET policy quote command (above).

This will allow node to invoke the mAPI reference implmentation if a double spend is detected. That, in turn, will cause a callback to any callbackUrl supplied in submit transaction (above).

![callback data flow](out/callbacks/callbackNotifications.png)

Double Spend example:
```
POST /mapi/tx
```

#### Request:

Request Body:

```json
{
    "rawtx": "01000000015d7d8ffefc2b95a68a95d8e3c50715f8affc0e56ef58a05c773789e6fa3eb537010000006a47304402206c1ba36989bdca944c4ac1e74c23afaaf93fb6ded3a3d6e01f2c28667373c26e0220676085f6fe30071022ea5c8e790e7d9cf52671d0bc3c4d374991be65b6e11bc34121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000",
    "callbackUrl": "https://your-server/api/v1/channel/533",
    "callbackToken": "CNaecHA44nGNJCvvccx3TSxwb4F490574knnkf44S19W6cNmbumVa6k3ESQw",
    "merkleProof": false,
    "merkleFormat": "",
    "dsCheck": true
}
```

#### Response:
```json
{
    "payload": "{\"apiVersion\":\"1.4.0\",\"timestamp\":\"2021-11-13T08:04:25.9291559Z\",\"txid\":\"0d0ad5677eb0862f94b3eda7f13633f91cf7c4c8c14e1451ffd333d52ff8e207\",\"returnResult\":\"failure\",\"resultDescription\":\"Missing inputs\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"100677f99bdd7d4f0b8ea3f35d575d0f69a80f89b5b5f14e11005f57e5e63ef5\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0,\"conflictedWith\":[{\"txid\":\"9f817649adde97338bcda695ee13ae1c71960eac60e49671fed0bdcf45581d94\",\"size\":191,\"hex\":\"01000000015d7d8ffefc2b95a68a95d8e3c50715f8affc0e56ef58a05c773789e6fa3eb537010000006a47304402206a9372778ff1ea314cfb2ec4e6bc93a57fe67c5ca915d004850f8079c876977c022066e3581cbec0eb2d525d4d83d01fff4f4e0b13a477f4f6a07d9168cc40bbabe54121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}]}",
    "signature": "3044022048739a74a7f14b870d410f02c60dafcee2899348c7cd1184977e9ac5096ba63a022038ca0066645d1201ba0f385bd88da4c9bc7410582ae7bb3e248d79b7dbcfd205",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

Merkle proof callback can be requested by specifying:
```json
{
    "merkleProof": true,
    "merkleFormat": "TSC"
}
```
If the optional merkleFormat is set to "TSC" then a TSC compliant version of the merkle proof is returned.

If callback was requested on Submit transaction, the merchant will receive a notification of doublespend and/or merkle proof via the callback URL. mAPI processes all requested notifications and sends them out in batches.
Callbacks have these possible callbackReasons: "doubleSpend", "doubleSpendAttempt" and "merkleProof". DoubleSpendAttempt implies that a double spend was detected in the mempool.

Double spend callback example:
```json
{	
    "callbackPayload": "{\"doubleSpendTxId\":\"f1f8d3de162f3558b97b052064ce1d0c45805490c210bdbc4d4f8b44cd0f143e\", \"payload\":\"01000000014979e6d8237d7579a19aa657a568a3db46a973f737c120dffd6a8ba9432fa3f6010000006a47304402205fc740f902ccdadc2c3323f0258895f597fb75f92b13d14dd034119bee96e5f302207fd0feb68812dfa4a8e281f9af3a5b341a6fe0d14ff27648ae58c9a8aacee7d94121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}",
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-03T13:24:31.233647Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "blockHash": "34bbc00697512058cb040e1c7bbba5d03a2e94270093eb28114747430137f9b7",
    "blockHeight": 153,
    "callbackTxId": "8750e986a296d39262736ed8b8f8061c6dce1c262844e1ad674a3bc134772167",
    "callbackReason": "doubleSpend"
}
```

| field                   | function                                                |
| ----------------------- | ------------------------------------------------------- |
| `callbackPayload`       | payload with information about the new transaction          |
| `blockHash`             | hash of tx block                                        |
| `blockHeight`           | height of tx block                                      |
| `callbackTxId`          | tx with requested dsCheck                               |
| `callbackReason`        | the reason for the callback                                 |

Double spend attempt callback example:
```json
{	
    "callbackPayload": "{\"doubleSpendTxId\":\"7ea230b1610768374285150537323add313c1b9271b1b8110f5ddc629bf77f46\", \"payload\":\"0100000001e75284dc47cb0beae5ebc7041d04dd2c6d29644a000af67810aad48567e879a0000000006a47304402203d13c692142b4b50737141145795ccb5bb9f5f8505b2d9b5a35f2f838b11feb102201cee2f2fe33c3d592f5e990700861baf9605b3b0199142bbc69ae88d1a28fa964121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}",
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-03T13:24:31.233647Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "blockHash": "34bbc00697512058cb040e1c7bbba5d03a2e94270093eb28114747430137f9b7",
    "blockHeight": 153,
    "callbackTxId": "8750e986a296d39262736ed8b8f8061c6dce1c262844e1ad674a3bc134772167",
    "callbackReason": "doubleSpendAttempt"
}
```

Merkle proof callback example:
```json
{
    "callbackPayload": "{\"flags\":2,\"index\":1,\"txOrId\":\"acad8d40b3a17117026ace82ef56d269283753d310ddaeabe7b5d226e8dbe973\",\"target\": {\"hash\":\"0e9a2af27919b30a066383d512d64d4569590f935007198dacad9824af643177\",\"confirmations\":1,\"height\":152,\"version\":536870912,\"versionHex\":\"20000000\",\"merkleroot\":\"0298acf415976238163cd82b9aab9826fb8fbfbbf438e55185a668d97bf721a8\",\"num_tx\":2,\"time\":1604409778,\"mediantime\":1604409777,\"nonce\":0,\"bits\":\"207fffff\",\"difficulty\":4.656542373906925E-10,\"chainwork\":\"0000000000000000000000000000000000000000000000000000000000000132\",\"previousblockhash\":\"62ae67b463764d045f4cbe54f1f7eb63ccf70d52647981ffdfde43ca4979a8ee\"},\"nodes\":[{\"5b537f8fba7b4057971f7e904794c59913d9a9038e6900669d08c1cf0cc48133\"}]}",
    "apiVersion": "1.4.0",
    "timestamp": "2021-11-03T13:22:42.1341243Z",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "blockHash": "0e9a2af27919b30a066383d512d64d4569590f935007198dacad9824af643177",
    "blockHeight": 152,
    "callbackTxId": "acad8d40b3a17117026ace82ef56d269283753d310ddaeabe7b5d226e8dbe973",
    "callbackReason": "merkleProof"
}
```

TSC Merkle proof callback example:
```json
{
  "callbackPayload": "{\"index\":1,\"txOrId\":\"e7b3eefab33072e62283255f193ef5d22f26bbcfc0a80688fe2cc5178a32dda6\",\"targetType\":\"header\",\"target\":\"00000020a552fb757cf80b7341063e108884504212da2f1e1ce2ad9ffc3c6163955a27274b53d185c6b216d9f4f8831af1249d7b4b8c8ab16096cb49dda5e5fbd59517c775ba8b60ffff7f2000000000\",\"nodes\":[\"30361d1b60b8ca43d5cec3efc0a0c166d777ada0543ace64c4034fa25d253909\",\"e7aa15058daf38236965670467ade59f96cfc6ec6b7b8bb05c9a7ed6926b884d\",\"dad635ff856c81bdba518f82d224c048efd9aae2a045ad9abc74f2b18cde4322\",\"6f806a80720b0603d2ad3b6dfecc3801f42a2ea402789d8e2a77a6826b50303a\"]}",
  "apiVersion": "1.4.0",
  "timestamp": "2021-04-30T08:06:13.4129624Z",
  "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
  "blockHash": "2ad8af91739e9dc41ea155a9ab4b14ab88fe2a0934f14420139867babf5953c4",
  "blockHeight": 105,
  "callbackTxId": "e7b3eefab33072e62283255f193ef5d22f26bbcfc0a80688fe2cc5178a32dda6",
  "callbackReason": "merkleProof"
}
```

--oOo--



