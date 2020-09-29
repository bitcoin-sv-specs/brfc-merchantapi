## RFC Notice

This draft spec is released as an RFC (request for comment) as part of the public review process. Any comments, criticisms or suggestions should be directed toward the [issues page](https://github.com/bitcoin-sv-specs/brfc-merchantapi/issues) on this github repository.

A beta reference implementation of the Merchant API server is available [here](https://github.com/bitcoin-sv/merchantapi-reference).

# Merchant API Specification

|     BRFC     |    title     | authors | version |
| :----------: | :----------: | :-----: | :-----: |
| 7c0cb0dcde2f | merchant_api | nChain  |   1.2-beta   |

## Overview

Merchant API (mAPI) is an additional service that miners can offer to merchants.

> Note: this protocol uses the [JSON envelopes BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) as well as the [Fee Spec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec).

---

## Implementation

The **REST API** has 4 endpoints:

**1. [Get fee quote](#get-fee-quote)**

#### Purpose:

This endpoint is used to get the different fees quoted by a miner. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the fees charged by a specific BSV miner. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Request:

```
GET /mapi/feeQuote
```


#### Response:

```json

{
    "payload": "{\"apiVersion\":\"1.2.3\",\"timestamp\":\"2020-09-15T12:44:19.75812Z\",\"expiryTime\":\"2020-09-17T13:09:31.4573849Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"2084f6352e242c496cba0a3c45be9b69ff2ef69718b1286a9c0f9c2a1089f6ae\",\"currentHighestBlockHeight\":1334,\"fees\":[{\"feeType\":\"standard\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}},{\"feeType\":\"data\",\"miningFee\":{\"satoshis\":500,\"bytes\":1000},\"relayFee\":{\"satoshis\":250,\"bytes\":1000}}]}",
    "signature": "304402202a26a937aae0906e3a624bb636718aa2c731a9b9f3775328015321865a87eee102202bbd9c23cc21fa0e806f05803359aa22ad9e019d2076b44ae625f1ff85afd664",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "utf-8",
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
  "apiVersion": "1.2.3",
  "timestamp": "2020-09-15T12:44:19.75812Z",
  "expiryTime": "2020-09-17T13:09:31.4573849Z",
  "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
  "currentHighestBlockHash": "2084f6352e242c496cba0a3c45be9b69ff2ef69718b1286a9c0f9c2a1089f6ae",
  "currentHighestBlockHeight": 1334,
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
| `currentHighestBlockHeight` | hash of current blockchain tip                                                               |
| `fees`          | fees charged by miner ([feeSpec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec))                                                                          



**2. [Submit transaction](#Submit-transaction)**

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
  "dsCheck" : true,
  "callBackEncryption" : <parameter>
}

```
To submit transaction in binary format use `Content-Type: application/octet-stream` with the binary serialized transaction in the request body. You can specify `callBackUrl`, `callBackToken`, `merkleProof`,`dsCheck` and `callBackEncryption` in the query string.

When Content-Type is application/octet-stream, it is possible to upload the rawtx as a binary stream. For large transactions, this is half the size of the hexadecimal equivalent although this gain is largely minimized through the use of gzip encoding of hex data.


| field                       | function            |
| ----------------------------|---------------------|
| `callbackURL`   | HTTP(S) endpoint used to receive messages from miner                                                                 |
| `callbackToken` | HTTP authorization header used when authenticating against callbackURL                                                                |
| `merkleProof`   | used to request a merkle proof    |
| `dsCheck`       | used to request double spend notification  |
| `callBackEncryption`   |   optional parameter to encrypt callback data     |


*Note:*  In mAPI 1.2.3-beta, the only encryption method supported is libsodium sealed_box which is an anonymous (you can not identify the sender) public key encryption with integrity check (see here for more details: https://libsodium.gitbook.io/doc/public-key_cryptography/sealed_boxes )

The format of callBackEncryption parameter is:

    libsodium sealed_box <base64 encoded encryption key>


#### Response:

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-15T11:40:29.826Z\",\"txid\":\"6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"currentHighestBlockHash\":\"71a7374389afaec80fcabbbf08dcd82d392cf68c9a13fe29da1a0c853facef01\",\"currentHighestBlockHeight\":207,\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100f65ae83b20bc60e7a5f0e9c1bd9aceb2b26962ad0ee35472264e83e059f4b9be022010ca2334ff088d6e085eb3c2118306e61ec97781e8e1544e75224533dcc32379",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
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
  "apiVersion": "1.2.3",
  "timestamp": "2020-01-15T11:40:29.826Z",
  "txid": "6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0",
  "returnResult": "success",
  "resultDescription": "",
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "currentHighestBlockHash": "71a7374389afaec80fcabbbf08dcd82d392cf68c9a13fe29da1a0c853facef01",
  "currentHighestBlockHeight": 207,
  "txSecondMempoolExpiry": 0,
  "conflictedWith": ""

}
```
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


### Callback Notifications

If a double spend notifcation or merkle proof is requested in Submit transaction, the response is sent to the specified callbackURL. Where recipients are using [SPV Channels](https://github.com/bitcoin-sv/brfc-spvchannels), this would require the recipient to have a channel setup and ready to receive messages.


**3. [Query transaction status](#Query-transaction-status)**

#### Purpose:

This endpoint is used to check the current status of a previously submitted transaction. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the transaction status. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

Request
```
GET /mapi/tx/{hash:[0-9a-fA-F]+}
```

#### Response:

```json
{
  "payload": "{\"apiVersion\":\"1.2.3\",\"timestamp\":\"2020-01-15T11:41:29.032Z\",\"returnResult\":\"failure\",\"resultDescription\":\"Transaction in mempool but not yet in block\",\"blockHash\":\"\",\"blockHeight\":0,\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"confirmations\":0,\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100f78a6ac49ef38fbe68db609ff194d22932d865d93a98ee04d2ecef5016872ba50220387bf7e4df323bf4a977dd22a34ea3ad42de1a2ec4e5af59baa13258f64fe0e5",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
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
  "apiVersion": "1.2.3",
  "timestamp": "2020-01-15T11:41:29.032Z",
  "txid": "6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0",
  "returnResult": "failure",
  "resultDescription": "Transaction in mempool but not yet in block",
  "blockHash": "",
  "blockHeight": 0,
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "confirmations": 0,
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
| `blockHeight`           | hash of tx block                                        |
| `minerId`               | minerId public key of miner                             |
| `confirmations`         | number of block confirmations                           |
| `txSecondMempoolExpiry` | duration (minutes) Tx will be kept in secondary mempool |

OR

```json
{
  "payload": "{\"apiVersion\":\"1.2.3\",\"timestamp\":\"2020-01-15T12:09:37.394Z\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"blockHash\":\"745093bb0c80780092d4ce6926e0caa753fe3accdc09c761aee89bafa85f05f4\",\"blockHeight\":208,\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"confirmations\":2,\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100c9a712a124ff3100e26f7bbcc87204848cc2ff1effacd8d8e8daac5d81bce74c02201dd661aad00d2cde443a076475cfb7d6523e0ef98a1112e938af002ca5222fbe",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "encoding": "UTF-8",
  "mimetype": "application/json"
}
```

Payload:

```json
{
  "apiVersion": "1.2.3",
  "timestamp": "2020-01-15T12:09:37.394Z",
  "txid": "6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0",
  "returnResult": "success",
  "resultDescription": "",
  "blockHash": "745093bb0c80780092d4ce6926e0caa753fe3accdc09c761aee89bafa85f05f4",
  "blockHeight": 208,
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "confirmations": 2,
  "txSecondMempoolExpiry": 0
}
```

**4. [Submit multiple transactions](#Submit-multiple-transactions)**

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
    "rawTx": "02000000010f263640b9da7923505cbd129f585cfe49f94331e25697009e6d16f94b96de5e010000006a47304402200c27d24b1857a47bad6bb0b65db453174e2eb9135392ffa7e11d129cfc86a20f0220017e3c4116908c8aead06f0a3e16f5bfcd45e7b91ec537df7eacaee0e138c904412103610198d993495d3c4b301ab3d6ae1e48c59b271e9111947e10729c5df6176bfcffffffff0210270000000000001976a9144ea8733cb23c06ea923b7e9bb6b35463b38c465888ac78b95302000000001976a9144c0a725e2573e3800c234e9a8fe02e50cdb0f4b488ac00000000",
    "callBackUrl":  "https://your.service.callback/endpoint",
    "callBackToken" : <channel token>,
    "merkleProof":true,
    "dsCheck": true,
    "callBackEncryption" : <parameter>
  },
  {
    "rawTx": "02000000010f263640b9da7923505cbd129f585cfe49f94331e25697009e6d16f94b96de5e010000006a47304402200d3af721a04c00de11c4d7d136627c8361dc569e51169bd0ead03ac701acfe66022051f7d4c2fb188e4829572f66c351f78a4e4719ba29c27aebbf7eb1cedaf12f6d412103610198d993495d3c4b301ab3d6ae1e48c59b271e9111947e10729c5df6176bfcffffffff0210270000000000001976a91421d62b3787c18c991fe3a692a469db590023388088ac78b95302000000001976a914cffb48a93f2e22cc5da322728daa160847a19ea988ac00000000",
    "callBackUrl":  "https://your.service.callback/endpoint",
    "callBackToken" : <channel token>,
    "merkleProof":true,
    "dsCheck": true,
    "callBackEncryption" : <parameter>
  }
]

```

#### Response:

```json
{
  "payload": "{\"apiVersion\":\"1.2.3\",\"timestamp\":\"2020-09-23T09:23:02.1369987Z\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"76beb84c5c709b6db91e2ea871c197d1ae15b47cd2caae780a26321eb2e9a290\",\"currentHighestBlockHeight\":151,\"txSecondMempoolExpiry\":0,\"txs\":[{\"txid\":\"a48f0c1c06024bb6d4d649b77e2f9b2636fa6a609ee85fc76603cecb5de0ffcb\",\"returnResult\":\"failure\",\"resultDescription\":\"Missing inputs\",\"conflictedWith\":[{\"txid\":\"ea559ed57a2f7dfdb7f33cb8601de113d782cea169eced6fb460c3e60066c323\",\"size\":191,\"hex\":\"01000000010fff3df96efcbc5637d923e27de88c45ed26392c4b791db46b8f69bd1192fc53010000006a47304402207c03d95ba831fe3686a5a081e81346312c67e33a845ed3be4fd053b3898556c0022074c24b2ae143756b5bcd20857e1d96549fd31f9b647599e255c5fcdb776fdcb04121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000\"}]},{\"txid\":\"65d11409d204ea80c81152d4c12ddbd37df72a0ee73828497c14cd6a0086eaf3\",\"returnResult\":\"success\"}],\"failureCount\":1}",
  "signature": "3044022076680491fa27e832a2002abce70fcccc7b859ed21c3db87c8fa0dbf143809c59022024898bbc0ef220201dd636f9fff9ee21ce90226ff327b59a505a622c9b42717b",
  "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
  "encoding": "UTF-8",
  "mimetype": "application/json"
}   
   
```

Payload:

```json
{
  "apiVersion": "1.2.3",
  "timestamp": "2020-09-23T09:23:02.1369987Z",
  "minerId": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
  "currentHighestBlockHash": "76beb84c5c709b6db91e2ea871c197d1ae15b47cd2caae780a26321eb2e9a290",
  "currentHighestBlockHeight": 151,
  "txSecondMempoolExpiry": 0,
  "txs": [
    {
      "txid": "a48f0c1c06024bb6d4d649b77e2f9b2636fa6a609ee85fc76603cecb5de0ffcb",
      "returnResult": "failure",
      "resultDescription": "Missing inputs",
      "conflictedWith": [
        {
          "txid": "ea559ed57a2f7dfdb7f33cb8601de113d782cea169eced6fb460c3e60066c323",
          "size": 191,
          "hex": "01000000010fff3df96efcbc5637d923e27de88c45ed26392c4b791db46b8f69bd1192fc53010000006a47304402207c03d95ba831fe3686a5a081e81346312c67e33a845ed3be4fd053b3898556c0022074c24b2ae143756b5bcd20857e1d96549fd31f9b647599e255c5fcdb776fdcb04121027ae06a5b3fe1de495fa9d4e738e48810b8b06fa6c959a5305426f78f42b48f8cffffffff0198929800000000001976a91482932cf55b847ffa52832d2bbec2838f658f226788ac00000000"
        }
      ]
    },
    {
      "txid": "65d11409d204ea80c81152d4c12ddbd37df72a0ee73828497c14cd6a0086eaf3",
      "returnResult": "success"
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
| `currentHighestBlockHeight` | hash of current blockchain tip                          |
| `txSecondMempoolExpiry`     | duration (minutes) Tx will be kept in secondary mempool |
| `txs`                       | list of transaction responses                            |
| `txid`                      | transaction ID                                          |
| `resultDescription`         | will contain the error on `failure` or empty on `success`   |
| `returnResult`              | will contain either `success` or `failure`                  |
| `failureCount`              | number of failed transaction submissions |
|                             |                                           |


### Callback Notifications

Merchants can request callbacks for *merkle proofs* and *double spend notifications* in Submit transaction

Double Spend example:

#### Request:

Request Body:

```json
{
    "rawtx": "02000000010b86283aa3742d76c9756490e533c42785aba9c04bff1d1b03aefc5db9696a090000000049483045022100fa0710a65f3fc9989e202040a404661c750b2b8edcf4aeca63256db57cf2ec61022029cc2253ec21727e22727509fc6b80c493455c3c21c55863c7e61a917667c1d041ffffffff0210270000000000001976a91462870360a34f460be9f70cfa4d5b5bd6705909d588ac14df2901000000001976a9141d0ef42a5362089f66250df3b876767ab0eb4d3488ac00000000",
    "callBackUrl":"https://your-server/api/v1/channel/533",
    "callBackToken":"CNaecHA44nGNJCvvccx3TSxwb4F490574knnkf44S19W6cNmbumVa6k3ESQw",
    "merkleProof":false,
    "dsCheck": true
}
```

#### Response:
```json
{
    "payload": "{\"apiVersion\":\"1.2.3\",\"timestamp\":\"2020-09-17T12:52:54.5817432Z\",\"txid\":\"8750e986a296d39262736ed8b8f8061c6dce1c262844e1ad674a3bc134772167\",\"returnResult\":\"failure\",\"resultDescription\":\"258 txn-mempool-conflict\",\"minerId\":\"030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e\",\"currentHighestBlockHash\":\"5feae707b1dbd08fb2d58d961620e4316edeeff0c4a1ce8b007171aaf7374dd4\",\"currentHighestBlockHeight\":1333,\"txSecondMempoolExpiry\":0}",
    "signature": "304402200c3ac8054adf9b3e97e021cc1db07101df93fd80992b7e26b1643f7c2320677e02207cf8d470d048822a922015ecb1cee194a8a6268314f8ed0c36661d9c275fb25c",
    "publicKey": "030d1fe5c1b560efe196ba40540ce9017c20daa9504c4c4cec6184fc702d9f274e",
    "encoding": "utf-8",
    "mimetype": "application/json"
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
  -n, --name <name> (REQUIRED)        Unique name od the subject token is being issued to
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
Now anyone `specialuser` using this token will offered special fee rates when uploaded. The special fees needs to be uploaded through admin interface

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

mAPI 1.2.3 includes performance optimisation – in some cases submitTransaction throughput is 4x better than in mAPI 0.1.1 even though mAPI 1.2.3 stores data about transactions in database and mAPI 1.1. did not maintain any state. This is due to new RPC functions implemented on bitcoind:
-	Submittranscations RPC – enables submission of multiple transactions at the same time
-	Getutxos RPC -  enables retrieval of batch of UTXOs (mAPI 1.1. had to retrieve whole transactions to obtain desired UTXO)

## Data Flow Diagram

![data flow](out/mapi-bip270/data_flow.png)

## Callback Notifications

![callback data flow](out/callbacks/callbackNotifications.png)