## RFC Notice

This draft spec is released as an RFC (request for comment) as part of the public review process. Any comments, criticisms or suggestions should be directed toward the [issues page](https://github.com/bitcoin-sv-specs/brfc-merchantapi/issues) on this github repository.

A reference implementation of the Merchant API server is available [here](https://github.com/bitcoin-sv/merchantapi-reference).

# mAPI Specification

|     BRFC     |    title     | authors | version |
| :----------: | :----------: | :-----: | :-----: |
| 8eeed98377bd | mAPI         | nChain  |   1.3.0 |

## Overview

Merchant API (mAPI) is an additional service that miners can offer to merchants.

> Note: this protocol uses the [JSON envelopes BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) as well as the [Fee Spec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec).

---

## Implementation

The **REST API** has 4 endpoints:

1. [Get fee quote](#1-get-fee-quote)
2. [Submit transaction](#2-submit-transaction)
3. [Query transaction status](#3-query-transaction-status)
4. [Submit multiple transactions](#4-submit-multiple-transactions)


### 1. Get fee quote

#### Purpose:

This endpoint is used to get the different fees quoted by a miner. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the fees charged by a specific BSV miner. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:

~~~

GET /mapi/feeQuote

~~~


#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.3.0\",\"timestamp\":\"2020-11-12T13:17:47.7498672Z\",\"expiryTime\":\"2020-11-12T13:27:47.7498672Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"45628be2fe616167b7da399ab63455e60ffcf84147730f4af4affca90c7d437e\",\"currentHighestBlockHeight\":234,\"fees\":[{\"feeType\":\"standard\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}},{\"feeType\":\"data\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}}]}",
    "signature": "30440220708e2e62a393f53c43d172bc1459b4daccf9cf23ff77cff923f09b2b49b94e0a022033792bee7bc3952f4b1bfbe9df6407086b5dbfc161df34fdee684dc97be72731",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | multipurpose Internet Mail Extensions type          |
|                                                                   |

#### Payload:

```json
{
    "apiVersion": "1.3.0",
    "timestamp": "2020-11-12T13:17:47.7498672Z",
    "expiryTime": "2020-11-12T13:27:47.7498672Z",
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

| field                       | function                                                                                     |
| --------------------------- | -------------------------------------------------------------------------------------------- |
| `apiVersion`                | version of merchant api spec                                                                 |
| `timestamp`                 | timestamp of payload document                                                                |
| `expiryTime`                | expiry time of quote                                                                         |
| `minerId`                   | minerID / public key of miner. This may be _null_.                                           |
| `currentHighestBlockHash`   | hash of current blockchain tip                                                               |
| `currentHighestBlockHeight` | height of current blockchain tip                                                               |
| `fees`          | fees charged by miner ([feeSpec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec))                                                                          



### 2. Submit transaction

#### Purpose:

This endpoint is used to send a raw transaction to a miner for inclusion in the next block that the miner creates. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the response to the transaction submission. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:

```
POST /mapi/tx
```
body: when `Content-Type` is `application/json`:

```json
{
    "rawtx":        "[transaction_hex_string]",
    "callBackUrl":  "https://your.service.callback/endpoint",
    "callBackToken" : <channel token>,
    "merkleProof" : true,
    "merkleFormat" : "TSC",
    "dsCheck" : true,
    "callBackEncryption" : <parameter>
}
```
To submit transaction in binary format use `Content-Type: application/octet-stream` with the binary serialized transaction in the request body. You can specify `callbackUrl`, `callbackToken`, `merkleProof`,`merkleFormat`,`dsCheck` and `callbackEncryption` in the query string.

`merkleFormat` is optional - If merkleFormat is set to "TSC" then a TSC compliant version of the merkle proof is returned.

When Content-Type is application/octet-stream, it is possible to upload the rawtx as a binary stream. For large transactions, this is half the size of the hexadecimal equivalent although this gain is largely minimized through the use of gzip encoding of hex data.


| field                       | function            |
| ----------------------------|---------------------|
| `callbackURL`   | HTTP(S) endpoint used to receive messages from miner                                                                 |
| `callbackToken` | HTTP authorization header used when authenticating against callbackURL                                                                |
| `merkleProof`   | used to request a merkle proof    |
| `merkleFormat`   | (optional) returns TSC compliant merkle proof format if set to "TSC"   |
| `dsCheck`       | used to request double spend notification  |
| `callbackEncryption`   |   optional parameter to encrypt callback data     |


*Note:*  In mAPI 1.3.0, the only encryption method supported is libsodium sealed_box which is an anonymous (you can not identify the sender) public key encryption with integrity check (see here for more details: https://libsodium.gitbook.io/doc/public-key_cryptography/sealed_boxes )

The format of callbackEncryption parameter is:

    libsodium sealed_box <base64 encoded encryption key>


#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.3.0\",\"timestamp\":\"2020-11-13T07:37:44.8783319Z\",\"txid\":\"fed22f5ab54202e2ec39cb745d427fcfff960254cde0cf283493ac545f5737f6\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"39e3a2a0e7ba1b9e331cfd396cef1a2d3baffa51624af2f5512e530f35a8aa43\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0}",
    "signature": "30440220160ff70b73297043a8ce9636f5abdc7d91918b4428e79876a405b080882d1c920220161b919399f50f16d8c9f922c13b94909d6e3a25480f09812c73f9ef52f8f542",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | multipurpose Internet Mail Extensions type          |

Payload:

```json
{
    "apiVersion": "1.3.0",
    "timestamp": "2020-11-13T07:37:44.8783319Z",
    "txid": "fed22f5ab54202e2ec39cb745d427fcfff960254cde0cf283493ac545f5737f6",
    "returnResult": "success",
    "resultDescription": "",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "currentHighestBlockHash": "39e3a2a0e7ba1b9e331cfd396cef1a2d3baffa51624af2f5512e530f35a8aa43",
    "currentHighestBlockHeight": 151,
    "txSecondMempoolExpiry": 0
}


| field                       | function                                                |
| --------------------------- | ------------------------------------------------------- |
| `apiVersion`                | version of merchant api spec                            |
| `timestamp`                 | timestamp of payload document                           |
| `txid`                      | transaction ID                                          |
| `returnResult`              | will contain either `success` or `failure`                  |
| `resultDescription`         | will contain the error on `failure` or empty on `success`   |
| `minerId`                   | minerId public key of miner                             |
| `currentHighestBlockHash`   | hash of current blockchain tip                          |
| `currentHighestBlockHeight` | hash of current blockchain tip                          |
| `txSecondMempoolExpiry`     | duration (minutes) Tx will be kept in secondary mempool |
| `conflictedWith`            | list of all double spend transactions                   |


If a double spend notification or merkle proof is requested in Submit transaction, the merkle proof or  double spend notification is sent to the specified callbackURL. Where recipients are using [SPV Channels](https://github.com/bitcoin-sv-specs/brfc-spvchannels), this would require the recipient to have a channel setup and ready to receive messages. Check [Callback Notifications](#callback-notifications) for details.

#### Callback Reason

There is an option for the miner to provide a callback reason to enable client applications to determine what type of callback is returned e.g. merkle proof or double spend

Request:

{
"rawTx": "0100000001b753fbcb4e99659468067c2512b64d80c593bf46d4b60f750dd77c59391c4210000000006a473044022000cc88f3feadbacfd93e2a1a723e4fa4a20ef329ab5daac3be962d973bee3fb5022031642f58b5fce72e531f9dcae49e74be95cdb2f59a312865517fa536581d85584121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000",
"callbackUrl":  "https://your.service.callback/endpoint/{callbackReason}",
"callbackToken" : <channel token>,
"merkleProof": true,
"merkleFormat" : "TSC",
"dsCheck": true,
"callbackEncryption" : <parameter>
}

Response:

Sample TSC compliant merkle proof callback:

{ "callbackPayload":"{\"index\":1,\"txOrId\":\"e7b3eefab33072e62283255f193ef5d22f26bbcfc0a80688fe2cc5178a32dda6\",\"targetType\":\"header\",\"target\":\"00000020a552fb757cf80b7341063e108884504212da2f1e1ce2ad9ffc3c6163955a27274b53d185c6b216d9f4f8831af1249d7b4b8c8ab16096cb49dda5e5fbd59517c775ba8b60ffff7f2000000000\",\"nodes\":[\"30361d1b60b8ca43d5cec3efc0a0c166d777ada0543ace64c4034fa25d253909\",\"e7aa15058daf38236965670467ade59f96cfc6ec6b7b8bb05c9a7ed6926b884d\",\"dad635ff856c81bdba518f82d224c048efd9aae2a045ad9abc74f2b18cde4322\",\"6f806a80720b0603d2ad3b6dfecc3801f42a2ea402789d8e2a77a6826b50303a\"]}",
   "apiVersion":"1.3.0",
   "timestamp":"2021-04-30T08:06:13.4129624Z",
   "minerId":"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
   "blockHash":"2ad8af91739e9dc41ea155a9ab4b14ab88fe2a0934f14420139867babf5953c4",
   "blockHeight":105,
   "callbackTxId":"e7b3eefab33072e62283255f193ef5d22f26bbcfc0a80688fe2cc5178a32dda6",
   "callbackReason":"merkleProof"
}
```


### 3. Query transaction status

#### Purpose
This endpoint is used to check the current status of a previously submitted transaction. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the transaction status. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

Request
```
GET /mapi/tx/{hash:[0-9a-fA-F]+}
```

#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.3.0\",\"timestamp\":\"2020-11-13T07:53:59.580061Z\",\"txid\":\"76bb952edbdd649e92f5f74c0341f5afa8679629a7e36bda30f6d2d530ffef6f\",\"returnResult\":\"failure\",\"resultDescription\":\"Mixed results\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"txSecondMempoolExpiry\":0}",
    "signature": "3044022038e82aff0decea8c95da0764c46e93613af2d5577468a89d0901de04f45fc83002205e073b1f5a8c275204873a6f14d5fa2d2acf5958f157201661d71fcbd57bbfe4",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | multipurpose Internet Mail Extensions type          |

Payload:

```json
{
    "apiVersion": "1.3.0",
    "timestamp": "2020-11-13T07:53:59.580061Z",
    "txid": "76bb952edbdd649e92f5f74c0341f5afa8679629a7e36bda30f6d2d530ffef6f",
    "returnResult": "failure",
    "resultDescription": "Mixed results",
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "txSecondMempoolExpiry": 0
}
```

| field                   | function                                                |
| ----------------------- | ------------------------------------------------------- |
| `apiVersion`            | version of merchant api spec                            |
| `timestamp`             | timestamp of payload document                           |
| `returnResult`          | will contain either success or failure                  |
| `resultDescription`     | will contain the error on failure or empty on success   |
| `blockHash`             | hash of tx block                                        |
| `blockHeight`           | height of tx block                                      |
| `minerId`               | minerId public key of miner                             |
| `confirmations`         | number of block confirmations                           |
| `txSecondMempoolExpiry` | duration (minutes) Tx will be kept in secondary mempool |

OR

```json
{
    "payload": "{\"apiVersion\":\"1.3.0\",\"timestamp\":\"2020-11-13T07:48:49.7666999Z\",\"txid\":\"6886cd04977d4cd26df3689b2d3c40b13685edb41fcc21c2962c5fc64560acff\",\"returnResult\":\"success\",\"blockHash\":\"236337115fef235a5d59cfcdf213cd70a86d6249a10eacf041468f15607094de\",\"blockHeight\":152,\"confirmations\":1,\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"txSecondMempoolExpiry\":0}",
    "signature": "3044022039b759b4e80f13e1c91b40a42ac777704d3866b2f24432f57236b44fdfae13fe0220544a3853eb8d58f973a14dbec7e16c56a8242a8e68722102ed70de79a2b1455e",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

Payload:

```json
{
    "apiVersion": "1.3.0",
    "timestamp": "2020-11-13T07:48:49.7666999Z",
    "txid": "6886cd04977d4cd26df3689b2d3c40b13685edb41fcc21c2962c5fc64560acff",
    "returnResult": "success",
    "blockHash": "236337115fef235a5d59cfcdf213cd70a86d6249a10eacf041468f15607094de",
    "blockHeight": 152,
    "confirmations": 1,
    "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "txSecondMempoolExpiry": 0
}
```

### 4. Submit multiple transactions

#### Purpose:

This endpoint is used to send multiple raw transactions to a miner for inclusion in the next block that the miner creates. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the responses to the transaction submissions. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

Request

```
POST /mapi/txs
```

body: where `Content-Type` is `application/json`:

```json
[
  {
    "rawTx": "01000000010136836d73f29cbe648bc2aeea20286502a3c2f2d3cff54522d0cc76bb755e9f000000006a4730440220533430d6f29d9437f94c60f0a59c857d254108fb9c375415fa53e9248d8bec5d0220606810830a6175dbee71da54fff6378a7aadfdb4b4714dc500cb3466ad4500004121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000",
    "callbackUrl":  "https://your.service.callback/endpoint",
    "callbackToken" : <channel token>,
    "merkleProof": true,
    "merkleFormat": "TSC",
    "dsCheck": true,
    "callbackEncryption" : <parameter>
  },
  {
    "rawTx": "0100000001b753fbcb4e99659468067c2512b64d80c593bf46d4b60f750dd77c59391c4210000000006a473044022000cc88f3feadbacfd93e2a1a723e4fa4a20ef329ab5daac3be962d973bee3fb5022031642f58b5fce72e531f9dcae49e74be95cdb2f59a312865517fa536581d85584121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000",
    "callbackUrl":  "https://your.service.callback/endpoint",
    "callbackToken" : <channel token>,
    "merkleProof": true,
    "merkleFormat": "TSC",
    "dsCheck": true,
    "callbackEncryption" : <parameter>
  }
]
```

You can also omit *callbackUrl*, *callbackToken*, *merkleProof*,*merkleFormat* and *dsCheck* from the request body and provide the default values in the query string.

#### Response:

```json
{
    "payload": "{\"apiVersion\":\"1.3.0\",\"timestamp\":\"2020-11-13T08:31:56.5722511Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"08dc4bb006fc7e7186544343c3ccbb5a773d0a19cd2ccff1fa52f51eb6faf2ab\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0,\"txs\":[{\"txid\":\"3145011f34a00d0666ea265b87c8e44108f87d3b53b853976906519ee8e1475f\",\"returnResult\":\"failure\",\"resultDescription\":\"Missing inputs\",\"conflictedWith\":[{\"txid\":\"86e1b384d3d169fd6aa4d34cf2d6f487436da54154befaab5a1fb25f844d65a8\",\"size\":191,\"hex\":\"01000000010136836d73f29cbe648bc2aeea20286502a3c2f2d3cff54522d0cc76bb755e9f000000006a4730440220761fb63128d4184fc142f2e854c499c52422db0136191f29f0bbe0969b6021770220536d72606d49dbbd244d2633b8b19031234138f045c530cc773e6e72bb34c62c4121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}]},{\"txid\":\"c8a087b1ee775fa29697511ecd64e800941c8a22db6ed0989fb27a1d2d6798da\",\"returnResult\":\"success\",\"resultDescription\":\"\"}],\"failureCount\":1}",
    "signature": "304402200c4b0dc179906581eb32953abeddbef5799d302d82367aa9a469d79c15f932f3022029e827af6122290d5e1b80c50676b0336f4c7658ca67ba4819396dff9c6239a6",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "UTF-8",
    "mimetype": "application/json"
}
```

Payload:

```json
{
    "apiVersion": "1.3.0",
    "timestamp": "2020-11-13T08:31:56.5722511Z",
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

| field                       | function                                                |
| --------------------------- | ------------------------------------------------------- |
| `apiVersion`                | version of merchant api spec                            |
| `timestamp`                 | timestamp of payload document                           |
| `minerId`                   | minerId public key of miner                             |
| `currentHighestBlockHash`   | hash of current blockchain tip                          |
| `currentHighestBlockHeight` | height of current blockchain tip                          |
| `txSecondMempoolExpiry`     | duration (minutes) Tx will be kept in secondary mempool |
| `txs`                       | list of transaction responses                            |
| `txid`                      | transaction ID                                          |
| `resultDescription`         | will contain the error on `failure` or empty on `success`   |
| `returnResult`              | will contain either `success` or `failure`                  |
| `failureCount`              | number of failed transaction submissions |
|                             |                                           |


To submit transaction in binary format use `Content-Type: application/octet-stream` Content-Type: application/octet-stream with the binary serialized transactions in the request body. Use query string to specify the remaining parameters.


### Callback Notifications

Merchants can request callbacks for *merkle proofs* and/or *double spend notifications* in Submit transaction.

Double Spend example:
```
POST /mapi/tx
```

#### Request:

Request Body:

```json
{
    "rawtx": "01000000015d7d8ffefc2b95a68a95d8e3c50715f8affc0e56ef58a05c773789e6fa3eb537010000006a47304402206c1ba36989bdca944c4ac1e74c23afaaf93fb6ded3a3d6e01f2c28667373c26e0220676085f6fe30071022ea5c8e790e7d9cf52671d0bc3c4d374991be65b6e11bc34121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000",
    "callbackUrl":"https://your-server/api/v1/channel/533",
    "callbackToken":"CNaecHA44nGNJCvvccx3TSxwb4F490574knnkf44S19W6cNmbumVa6k3ESQw",
    "merkleProof":false,
    "merkleFormat":"",
    "dsCheck": true
}
```

#### Response:
```json
{
    "payload": "{\"apiVersion\":\"1.3.0\",\"timestamp\":\"2020-11-13T08:04:25.9291559Z\",\"txid\":\"0d0ad5677eb0862f94b3eda7f13633f91cf7c4c8c14e1451ffd333d52ff8e207\",\"returnResult\":\"failure\",\"resultDescription\":\"Missing inputs\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"100677f99bdd7d4f0b8ea3f35d575d0f69a80f89b5b5f14e11005f57e5e63ef5\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0,\"conflictedWith\":[{\"txid\":\"9f817649adde97338bcda695ee13ae1c71960eac60e49671fed0bdcf45581d94\",\"size\":191,\"hex\":\"01000000015d7d8ffefc2b95a68a95d8e3c50715f8affc0e56ef58a05c773789e6fa3eb537010000006a47304402206a9372778ff1ea314cfb2ec4e6bc93a57fe67c5ca915d004850f8079c876977c022066e3581cbec0eb2d525d4d83d01fff4f4e0b13a477f4f6a07d9168cc40bbabe54121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}]}",
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
merkleFormat is optional - If merkleFormat is set to "TSC" then a TSC compliant version of the merkle proof is returned.

If callback was requested on transaction submit, merchant should receive a notification of doublespend and/or merkle proof via callback URL. mAPI process all requested notifications and sends them out in batches.
Callbacks have three possible callbackReasons: "doubleSpend", "doubleSpendAttempt" and "merkleProof". DoubleSpendAttempt implies, that double spend was detected in mempool.

Double spend callback example:
```json
{	
  "callbackPayload": "{\"doubleSpendTxId\":\"f1f8d3de162f3558b97b052064ce1d0c45805490c210bdbc4d4f8b44cd0f143e\", \"payload\":\"01000000014979e6d8237d7579a19aa657a568a3db46a973f737c120dffd6a8ba9432fa3f6010000006a47304402205fc740f902ccdadc2c3323f0258895f597fb75f92b13d14dd034119bee96e5f302207fd0feb68812dfa4a8e281f9af3a5b341a6fe0d14ff27648ae58c9a8aacee7d94121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}",
  "apiVersion": "1.3.0",
  "timestamp": "2020-11-03T13:24:31.233647Z",
  "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
  "blockHash": "34bbc00697512058cb040e1c7bbba5d03a2e94270093eb28114747430137f9b7",
  "blockHeight": 153,
  "callbackTxId": "8750e986a296d39262736ed8b8f8061c6dce1c262844e1ad674a3bc134772167",
  "callbackReason": "doubleSpend"
}
```

| field                   | function                                                |
| ----------------------- | ------------------------------------------------------- |
| `callbackPayload`       | payload with information about new transaction          |
| `apiVersion`            | version of merchant api spec                            |
| `timestamp`             | timestamp of payload document                           |
| `minerId`               | minerId public key of miner                             |
| `blockHash`             | hash of tx block                                        |
| `blockHeight`           | height of tx block                                      |
| `callbackTxId`          | tx with requested dsCheck                               |
| `callbackReason`        | the reason for callback                                 |

Double spend attempt callback example:
```json
{	
  "callbackPayload": "{\"doubleSpendTxId\":\"7ea230b1610768374285150537323add313c1b9271b1b8110f5ddc629bf77f46\", \"payload\":\"0100000001e75284dc47cb0beae5ebc7041d04dd2c6d29644a000af67810aad48567e879a0000000006a47304402203d13c692142b4b50737141145795ccb5bb9f5f8505b2d9b5a35f2f838b11feb102201cee2f2fe33c3d592f5e990700861baf9605b3b0199142bbc69ae88d1a28fa964121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff018c949800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}",
  "apiVersion": "1.3.0",
  "timestamp": "2020-11-03T13:24:31.233647Z",
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
  "callbackPayload": "{\"flags\":2,\"index\":1,\"txOrId\":\"acad8d40b3a17117026ace82ef56d269283753d310ddaeabe7b5d226e8dbe973\",\"target\": {\"hash\":\"0e9a2af27919b30a066383d512d64d4569590f935007198dacad9824af643177\",\"confirmations\":1,\"height\":152,\"version\":536870912,\"versionHex\":"20000000",\"merkleroot\":\"0298acf415976238163cd82b9aab9826fb8fbfbbf438e55185a668d97bf721a8\",\"num_tx\":2,\"time\":1604409778,\"mediantime\":1604409777,\"nonce\":0,\"bits\":\"207fffff\",\"difficulty\":4.656542373906925E-10,\"chainwork\":\"0000000000000000000000000000000000000000000000000000000000000132\",\"previousblockhash\":\"62ae67b463764d045f4cbe54f1f7eb63ccf70d52647981ffdfde43ca4979a8ee\"},\"nodes\":[\"5b537f8fba7b4057971f7e904794c59913d9a9038e6900669d08c1cf0cc48133\"]}",
  "apiVersion":"1.3.0",
  "timestamp":"2020-11-03T13:22:42.1341243Z",
  "minerId":"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
  "blockHash":"0e9a2af27919b30a066383d512d64d4569590f935007198dacad9824af643177",
  "blockHeight":152,
  "callbackTxId":"acad8d40b3a17117026ace82ef56d269283753d310ddaeabe7b5d226e8dbe973",
  "callbackReason":"merkleProof"
}
```


### Authorization/Authentication and Special Rates

Merchant API providers would likely want to offer special or discounted rates to specific customers. To do this they would need to add an extra layer to enable authorization/authentication on public interface. Current implementation supports JSON Web Tokens (JWT) issued to specific users. The users can include that token in their HTTP header and as a result receive lower fee rates.

If no token is used and the call is done anonymously, then the default rate is supplied. If a JWT token (issued by merchant API or other identity provider) is used, then the caller will receive the corresponding fee rate. At the moment, for this version of the merchant API implementation, the token must be issued and sent to the customer manually.

### Authorization/Authentication Example

```console
$ curl -H "Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOiIyMDIwLTEwLTE0VDExOjQ0OjA3LjEyOTAwOCswMTowMCIsIm5hbWUiOiJsb3cifQ.LV8kz02bwxZ21qgqCvmgWfbGZCtdSo9px47wQ3_6Zrk" localhost:9004/mapi/feeQuote
```

### JWT Token Manager

The reference implementation contains a token manager that can be used to generate and verify validity of the tokens. Token manager currently only supports symmetric encryption `HS256`.

The following command line options can be specified when generating a token
```console
Options:
  -n, --name <name> (REQUIRED)        Unique name of the subject token is being issued to
  -d, --days <days> (REQUIRED)        Days the token will be valid for
  -k, --key <key> (REQUIRED)          Secret shared use to sign the token. At lest 16 characters
  -i, --issuer <issuer> (REQUIRED)    Unique issuer of the token (for example URI identifiably the miner)
  -a, --audience <audience>           Audience tha this token should be used for [default: merchant_api]
```

For example, you can generate the token by running 

```console
$ TokenManager generate -n specialuser -i http://mysite.com -k thisisadevelopmentkey -d 1000

Token:{"alg":"HS256","typ":"JWT"}.{"sub":"specialuser","nbf":1599494789,"exp":1685894789,"iat":1599494789,"iss":"http://mysite.com","aud":"merchant_api"}
Valid until UTC: 4. 06. 2023 16:06:29

The following should be used as authorization header:
Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzcGVjaWFsdXNlciIsIm5iZiI6MTU5OTQ5NDc4OSwiZXhwIjoxNjg1ODk0Nzg5LCJpYXQiOjE1OTk0OTQ3ODksImlzcyI6Imh0dHA6Ly9teXNpdGUuY29tIiwiYXVkIjoibWVyY2hhbnRfYXBpIn0.xbtwEKdbGv1AasXe_QYsmb5sURyrcr-812cX-Ps98Yk

```
Now any `specialuser` using this token will be offered special fee rates when uploaded. The special fees needs to be uploaded through admin interface.

To validate a token, you can use `validate` command:
```console
$ TokenManager validate -k thisisadevelopmentkey -t eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzcGVjaWFsdXNlciIsIm5iZiI6MTU5OTQ5NDc4OSwiZXhwIjoxNjg1ODk0Nzg5LCJpYXQiOjE1OTk0OTQ3ODksImlzcyI6Imh0dHA6Ly9teXNpdGUuY29tIiwiYXVkIjoibWVyY2hhbnRfYXBpIn0.xbtwEKdbGv1AasXe_QYsmb5sURyrcr-812cX-Ps98Yk

Token signature and time constraints are OK. Issuer and audience were not validated.

Token:
{"alg":"HS256","typ":"JWT"}.{"sub":"specialuser","nbf":1599494789,"exp":1685894789,"iat":1599494789,"iss":"http://mysite.com","aud":"merchant_api"}
```

## Admin interface  
Admin interface can be used to add update or remove connections to this node. It is only accessible to authenticated users. Authentication is performed through `Api-Key` HTTP header. The provided value must match the one provided in configuration variable `RestAdminAPIKey`.


### Managing fee quotes

To create a new fee quote use the following:
```
POST api/v1/FeeQuote
```

Example with curl - add feeQuote valid from 01/10/2020 for anonymous user:

```console
$ curl -H "Api-Key: [RestAdminAPIKey]" -H "Content-Type: application/json" -X POST https://localhost:5051/api/v1/FeeQuote -d "{ \"validFrom\": \"2020-10-01T12:00:00\", \"identity\": null, \"identityProvider\": null, \"fees\": [{ \"feeType\": \"standard\", \"miningFee\" : { \"satoshis\": 100, \"bytes\": 200 }, \"relayFee\" : { \"satoshis\": 100, \"bytes\": 200 } }, { \"feeType\": \"data\", \"miningFee\" : { \"satoshis\": 100, \"bytes\": 200 }, \"relayFee\" : { \"satoshis\": 100, \"bytes\": 200 } }] }"
```

To get list of all fee quotes, matching one or more criteria use the following
```
GET api/v1/FeeQuote
```

You can filter fee quotes by providing additional optional criteria in query string:
* `identity` - return only fee quotes for users that authenticate with a JWT token that was issued to specified identity 
* `identityProvider` - return only fee quotes for users that authenticate with a JWT token that was issued by specified token authority
* `anonymous` - specify  `true` to return only fee quotes for anonymous user.
* `current` - specify  `true` to return only fee quotes that are currently valid.
* `valid` - specify  `true` to return only fee quotes that are valid in interval with QuoteExpiryMinutes

To get list of all fee quotes (including expired ones) for all users use GET api/v1/FeeQuote without filters.


To get a specific fee quote by id use:
```
GET api/v1/FeeQuote/{id}
```

Note: it is not possible to delete or update a fee quote once it is published, but you can make it obsolete by publishing a new fee quote.

### Performance

mAPI 1.3.0 includes performance optimisation - in some cases submitTransaction throughput is 4x better than in mAPI 0.1.1 even though mAPI 1.3.0 stores data about transactions in database and mAPI 1.1. did not maintain any state. This is due to new RPC functions implemented on bitcoind:
-	Submittransactions RPC - enables submission of multiple transactions at the same time
-	Getutxos RPC -  enables retrieval of batch of UTXOs (mAPI 1.1. had to retrieve whole transactions to obtain desired UTXO)

## Data Flow Diagram

![data flow](out/mapi-bip270/data_flow.png)

## Callback Notifications

![callback data flow](out/callbacks/callbackNotifications.png)
