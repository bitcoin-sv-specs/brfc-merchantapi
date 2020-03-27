# Merchant API Spec

|     BRFC     |    title     | authors | version |
| :----------: | :----------: | :-----: | :-----: |
| ce852c4c2cd1 | merchant_api | nChain  |   0.1   |

## Overview

Merchant API is an additional service that miners can offer to merchants.

> Note: this protocol uses the [JSON envelopes BRFC](../brfc-misc/jsonenvelope/README.md) as well as the [Fee Spec BRFC](../brfc-misc/feespec/README.md).

## Implementation

The **REST API** has 3 endpoints:

**1. Get fee quote**

```
GET /mapi/feeQuote
```

**2. Submit transaction**

```
POST /mapi/tx
```

body:

```json
{
  "rawtx": "[transaction_hex_string]"
}
```

**3. Query transaction status**

```
GET /mapi/tx/{hash:[0-9a-fA-F]+}
```

### Get fee quote

#### Purpose:

This endpoint returns a [JSONEnvelope](../brfc-misc/jsonenvelope/README.md) with a payload that contains the fees charged by a specific BSV miner. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-28T11:15:03.722Z\",\"expiryTime\":\"2020-01-28T11:25:03.722Z\",\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"currentHighestBlockHash\":\"000000000000000001cedc3dec00ecd29943a275498e812e72b2afdf5df8814a\",\"currentHighestBlockHeight\":619574,\"minerReputation\":\"N/A\",\"fees\":[{\"feeType\":\"standard\",\"miningFee\":{\"satoshis\":1,\"bytes\":1},\"relayFee\":{\"satoshis\":1,\"bytes\":1}},{\"feeType\":\"data\",\"miningFee\":{\"satoshis\":1,\"bytes\":1},\"relayFee\":{\"satoshis\":1,\"bytes\":1}}]}",
  "signature": "304402202a7f70855739a6948c00c2a85dd733f087c4f1ae4beb256c225eadab767d5e1d02207870c57728166f61b0334bd89640d6d6c26f31ada4aac42b29971ebfa5c414e1",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "encoding": "json",
  "mimetype": "applicaton/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | Multipurpose Internet Mail Extensions type          |

Payload:

```json
{
  "apiVersion": "0.1.0",
  "timestamp": "2020-01-28T11: 15: 03.722Z",
  "expiryTime": "2020-01-28T11: 25: 03.722Z",
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "currentHighestBlockHash": "000000000000000001cedc3dec00ecd29943a275498e812e72b2afdf5df8814a",
  "currentHighestBlockHeight": 619574,
  "minerReputation": null,
  "fees": [
    {
      "feeType": "standard",
      "miningFee": {
        "satoshis": 1,
        "bytes": 1
      },
      "relayFee": {
        "satoshis": 1,
        "bytes": 1
      }
    },
    {
      "feeType": "data",
      "miningFee": {
        "satoshis": 1,
        "bytes": 1
      },
      "relayFee": {
        "satoshis": 1,
        "bytes": 1
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
| `minerReputation`           | reputation of miner                                                                          |
| `fees`                      | fees charged by miner ([feeSpec BRFC](https://bitbucket.org/nchteamnch/feespec/src/master/)) |

### Submit transaction

#### Purpose:

This endpoint is used to send a raw transaction to a miner for inclusion in the next block that the miner creates.

> **body** when `Content-Type` is `application/json`:

```json
{
  "rawtx": "0200000001f56216b33513cf621839d584f0e31566059537ef184070733a56c3b5c41d0d6d0000000049483045022100cddd5c304ff87f1733262d34b56a90061aa4b97fdc0b0e42fc3065c04e231ca402202f0c65cb6d04c2b8fc82bdcfcfd7295b6e0f717362ec395e02f6905c68ac7b7741ffffffff01805b6d29010000001976a9142a5acfb9a647a03a758afaa5c359284d4b95c0be88ac00000000"
}
```

When Content-Type is application/octet-stream, it is possible to upload the rawtx as a binary stream. For large transactions, this is half the size of the hexadecimal equivalent although this gain is largely minimized through the use of gzip encoding of hex data.

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-15T11:40:29.826Z\",\"txid\":\"6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"currentHighestBlockHash\":\"71a7374389afaec80fcabbbf08dcd82d392cf68c9a13fe29da1a0c853facef01\",\"currentHighestBlockHeight\":207,\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100f65ae83b20bc60e7a5f0e9c1bd9aceb2b26962ad0ee35472264e83e059f4b9be022010ca2334ff088d6e085eb3c2118306e61ec97781e8e1544e75224533dcc32379",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "encoding": "json",
  "mimetype": "applicaton/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | Multipurpose Internet Mail Extensions type          |

Payload:

```json
{
  "apiVersion": "0.1.0",
  "timestamp": "2020-01-15T11:40:29.826Z",
  "txid": "6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0",
  "returnResult": "success",
  "resultDescription": "",
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "currentHighestBlockHash": "71a7374389afaec80fcabbbf08dcd82d392cf68c9a13fe29da1a0c853facef01",
  "currentHighestBlockHeight": 207,
  "txSecondMempoolExpiry": 0
}
```

| field                       | function                                                |
| --------------------------- | ------------------------------------------------------- |
| `apiVersion`                | version of merchant api spec                            |
| `timestamp`                 | timestamp of payload document                           |
| `txid`                      | transaction ID                                          |
| `returnResult`              | will contain either success or failure                  |
| `resultDescription`         | will contain the error on failure or empty on success   |
| `minerId`                   | minerId public key of miner                             |
| `currentHighestBlockHash`   | hash of current blockchain tip                          |
| `currentHighestBlockHeight` | hash of current blockchain tip                          |
| `txSecondMempoolExpiry`     | Duration (minutes) Tx will be kept in secondary mempool |

### Query transaction status

#### Purpose:

This endpoint is used to check the current status of a previously submitted transaction.

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-15T11:41:29.032Z\",\"returnResult\":\"failure\",\"resultDescription\":\"Transaction in mempool but not yet in block\",\"blockHash\":\"\",\"blockHeight\":0,\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"confirmations\":0,\"signatureValidFrom\":\"0001-01-01T00:00:00.000Z\",\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100f78a6ac49ef38fbe68db609ff194d22932d865d93a98ee04d2ecef5016872ba50220387bf7e4df323bf4a977dd22a34ea3ad42de1a2ec4e5af59baa13258f64fe0e5",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "encoding": "json",
  "mimetype": "applicaton/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | Multipurpose Internet Mail Extensions type          |

Payload:

```json
{
  "apiVersion": "0.1.0",
  "timestamp": "2020-01-15T11:41:29.032Z",
  "returnResult": "failure",
  "resultDescription": "Transaction in mempool but not yet in block",
  "blockHash": "",
  "blockHeight": 0,
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "confirmations": 0,
  "signatureValidFrom": "0001-01-01T00:00:00.000Z",
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
| `signatureValidFrom`    | number of block confirmations                           |
| `txSecondMempoolExpiry` | Duration (minutes) Tx will be kept in secondary mempool |

OR

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-15T12:09:37.394Z\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"blockHash\":\"745093bb0c80780092d4ce6926e0caa753fe3accdc09c761aee89bafa85f05f4\",\"blockHeight\":208,\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"confirmations\":2,\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100c9a712a124ff3100e26f7bbcc87204848cc2ff1effacd8d8e8daac5d81bce74c02201dd661aad00d2cde443a076475cfb7d6523e0ef98a1112e938af002ca5222fbe",
  "publicKey": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "encoding": "json",
  "mimetype": "applicaton/json"
}
```

| field       | function                                            |
| ----------- | --------------------------------------------------- |
| `payload`   | main data payload encoded in a specific format type |
| `signature` | signature on payload string. This may be _null_.    |
| `publicKey` | public key to verify signature. This may be _null_. |
| `encoding`  | encoding type                                       |
| `mimetype`  | Multipurpose Internet Mail Extensions type          |

Payload:

```json
{
  "apiVersion": "0.1.0",
  "timestamp": "2020-01-15T12:09:37.394Z",
  "returnResult": "success",
  "resultDescription": "",
  "blockHash": "745093bb0c80780092d4ce6926e0caa753fe3accdc09c761aee89bafa85f05f4",
  "blockHeight": 208,
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "confirmations": 2,
  "txSecondMempoolExpiry": 0
}
```

### Authorization/Authentication and Special Rates

Merchant API providers may wish to offer different fee rates to specific customers. To do this they would need to add an extra layer to enable authorization/authentication. An example implementation would be to use JSON Web Tokens ([JWT](https://jwt.io/)) issued to specific users. The users would then include that token in their HTTP header and as a result recieve their user specific fee rates.

## Data Flow Diagram

![data flow](out/mapi-bip270/data_flow.png)
