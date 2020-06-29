## RFC Notice

This draft spec is released as an RFC (request for comment) as part of the public review process. Any comments, criticisms or suggestions should be directed toward the [issues page](https://github.com/bitcoin-sv-specs/brfc-merchantapi/issues) on this github repository.

A beta reference implementation of the Merchant API server is available [here](https://github.com/bitcoin-sv/merchantapi-reference).

# Merchant API Specification

|     BRFC     |    title     | authors | version |
| :----------: | :----------: | :-----: | :-----: |
| 3e76b76bcfea | merchant_api | nChain  |   1.1   |

## Overview

Merchant API is an additional service that miners can offer to merchants.

> Note: this protocol uses the [JSON envelopes BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) as well as the [Fee Spec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec).

---

## Implementation

## Requests

The **REST API** has 5 endpoints:

**1. [Get fee quote](#get-fee-quote)**

```
GET /mapi/feeQuote
```

**2. [Submit transaction](#Submit-transaction)**

```
POST /mapi/tx
```

body:

```json
{
  "rawtx": "[transaction_hex_string]"
}
```

**3. [Query transaction status](#Query-transaction-status)**

```
GET /mapi/tx/{hash:[0-9a-fA-F]+}
```

**4. [Submit multiple transactions](#Submit-multiple-transactions)**

```
POST /mapi/txs/submit
```

body:

```json
[
  "[transaction_hex_string]",
  "[transaction_hex_string]",
  "[transaction_hex_string]"
]
```

**5. [Send multi-transaction status query](#Send-multi-transaction-status-query)**

```
POST /mapi/txs/status
```

body:

```json
[
  "[txid]",
  "[txid]",
  "[txid]"
]
```

---

## Responses

### Get fee quote

#### Purpose:

This endpoint is used to get the different fees quoted by a miner. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the fees charged by a specific BSV miner. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-28T11:15:03.722Z\",\"expiryTime\":\"2020-01-28T11:25:03.722Z\",\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"currentHighestBlockHash\":\"000000000000000001cedc3dec00ecd29943a275498e812e72b2afdf5df8814a\",\"currentHighestBlockHeight\":619574,\"minerReputation\":\"N/A\",\"fees\":[{\"feeType\":\"standard\",\"miningFee\":{\"satoshis\":1,\"bytes\":1},\"relayFee\":{\"satoshis\":1,\"bytes\":1}},{\"feeType\":\"data\",\"miningFee\":{\"satoshis\":1,\"bytes\":1},\"relayFee\":{\"satoshis\":1,\"bytes\":1}}]}",
  "signature": "304402202a7f70855739a6948c00c2a85dd733f087c4f1ae4beb256c225eadab767d5e1d02207870c57728166f61b0334bd89640d6d6c26f31ada4aac42b29971ebfa5c414e1",
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
| `fees`                      | fees charged by miner ([feeSpec BRFC](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/feespec)) |

### Submit transaction

#### Purpose:

This endpoint is used to send a raw transaction to a miner for inclusion in the next block that the miner creates. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the response to the transaction submission. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

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
| `returnResult`              | will contain either `success` or `failure`                  |
| `resultDescription`         | will contain the error on `failure` or empty on `success`   |
| `minerId`                   | minerId public key of miner                             |
| `currentHighestBlockHash`   | hash of current blockchain tip                          |
| `currentHighestBlockHeight` | hash of current blockchain tip                          |
| `txSecondMempoolExpiry`     | duration (minutes) Tx will be kept in secondary mempool |

### Query transaction status

#### Purpose:

This endpoint is used to check the current status of a previously submitted transaction. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the transaction status. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-15T11:41:29.032Z\",\"returnResult\":\"failure\",\"resultDescription\":\"Transaction in mempool but not yet in block\",\"blockHash\":\"\",\"blockHeight\":0,\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"confirmations\":0,\"txSecondMempoolExpiry\":0}",
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
  "apiVersion": "0.1.0",
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
  "payload": "{\"apiVersion\":\"0.1.0\",\"timestamp\":\"2020-01-15T12:09:37.394Z\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"blockHash\":\"745093bb0c80780092d4ce6926e0caa753fe3accdc09c761aee89bafa85f05f4\",\"blockHeight\":208,\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"confirmations\":2,\"txSecondMempoolExpiry\":0}",
  "signature": "3045022100c9a712a124ff3100e26f7bbcc87204848cc2ff1effacd8d8e8daac5d81bce74c02201dd661aad00d2cde443a076475cfb7d6523e0ef98a1112e938af002ca5222fbe",
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
  "apiVersion": "0.1.0",
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

| field                   | function                                                |
| ----------------------- | ------------------------------------------------------- |
| `apiVersion`            | version of merchant api spec                            |
| `timestamp`             | timestamp of payload document                           |
| `txid`                  | transaction ID                                          |
| `returnResult`          | will contain either `success` or `failure`                  |
| `resultDescription`     | will contain the error on `failure` or empty on `success`   |
| `blockHash`             | hash of tx block                                        |
| `blockHeight`           | hash of tx block                                        |
| `minerId`               | minerId public key of miner                             |
| `confirmations`         | number of block confirmations                           |
| `txSecondMempoolExpiry` | duration (minutes) Tx will be kept in secondary mempool |

### Submit multiple transactions

#### Purpose:

This endpoint is used to send multiple raw transactions to a miner for inclusion in the next block that the miner creates. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the responses to the transaction submissions. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

> **body** when `Content-Type` is `application/json`:

```json
[
  "0200000001f56216b33513cf621839d584f0e31566059537ef184070733a56c3b5c41d0d6d0000000049483045022100cddd5c304ff87f1733262d34b56a90061aa4b97fdc0b0e42fc3065c04e231ca402202f0c65cb6d04c2b8fc82bdcfcfd7295b6e0f717362ec395e02f6905c68ac7b7741ffffffff01805b6d29010000001976a9142a5acfb9a647a03a758afaa5c359284d4b95c0be88ac00000000",
  "0200000001698562b56805aee866fb835ee1a5e874fe666e037a5c70c1e9a53223bd95d0350000000000ffffffff01805b6d29010000001976a914b0b9356ec35de3d83e261570bf7c91046c209d4488ac00000000",
  "0100000001afb6269f0193eaf62d3f4e8615cef451157bf05e483f504215da0c813e41bf97030000006a47304402207758c41cdec96a8e62a4e183beb8b828f6b1a74f9339ddaeb993ddfd51fd4416022005608570cfb83c5d418a7960d8dff3a30f91fc023549b0c6d56ed2d1b8c7fe0141210368cc5b24e57d1c10dd7e5f269c250ed305d96cb4df686d58e65e8a47598b9bf2ffffffff040000000000000000fa006a223150755161374b36324d694b43747373534c4b79316b683536575755374d74555235035345540361707008746f6e6963706f770474797065106f666665725f636f6e76657273696f6e1a6f666665725f636f6e76657273696f6e5f636f6e6669675f69644037396361353766306261666439303434326634336631653862336131353566323339343536333533636663313363346537316435623835383734663062346239106f666665725f73657373696f6e5f69644061623537363231343564656430386161353664653435303566386230316131366639333739363832623237333439376561373730356530316233623436353231c4060000000000001976a91409cc4559bdcb84cb35c107743f0dbb10d66679cc88aca7430000000000001976a914685d569aa5341f7c034d2564ea96d75383468ffe88ac1a440000000000001976a9142566463dbd7e615046d4952fb5ebf3604d9b2c9688ac00000000"
]
```

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"1.1.0\",\"timestamp\":\"2020-06-15T12:09:37.394Z\",\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"currentHighestBlockHash\":\"000000000000000002b960f6569b5b968d84de40a85cf7831ed61afe935ed04c\",\"currentHighestBlockHeight\":640821,\"txSecondMempoolExpiry\":0,\"failureCount\":1,\"txs\":[{\"txid\":\"c013c468b94a600f9c505155e5bd23f9ce12bf07242f6251c5778aa0d087b4de\",\"returnResult\":\"success\",\"resultDescription\":\"\"},{\"txid\":\"2dfe4b326d0ae541a3a01bd567428d358009ebaedeb1cc0d6b30d7732525c863\",\"returnResult\":\"failure\",\"resultDescription\":\"Missing inputs\"},{\"txid\":\"18e6b339551fd177587aaf968281c0d400e20aa200349903bed0883b10b7612b\",\"returnResult\":\"success\",\"resultDescription\":\"\"}]}",
  "signature": "304402204d520b905c2c3b52a254f4b8b661473235139530b892696bfce52bcc992ff4f302203b8fbd33b1c2a1f92b559c5930ba95682a8d77d069863aef5d0d2344b3fa7355",
  "publicKey": "03da105d3eb8d5d8b1e69c02be4b1499f4fa81926202cc53ff17d21ea5652ab3cb",
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
  "apiVersion": "1.1.0",
  "timestamp": "2020-06-15T12:09:37.394Z",
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "currentHighestBlockHash": "000000000000000002b960f6569b5b968d84de40a85cf7831ed61afe935ed04c",
  "currentHighestBlockHeight": 640821,
  "txSecondMempoolExpiry": 0,
  "failureCount": 1,
  "txs": [
    {
      "txid": "c013c468b94a600f9c505155e5bd23f9ce12bf07242f6251c5778aa0d087b4de",
      "returnResult": "success",
      "resultDescription": "",
    },
    {
      "txid": "2dfe4b326d0ae541a3a01bd567428d358009ebaedeb1cc0d6b30d7732525c863",
      "returnResult": "failure",
      "resultDescription": "Missing inputs",
    },
    {
      "txid": "18e6b339551fd177587aaf968281c0d400e20aa200349903bed0883b10b7612b",
      "returnResult": "success",
      "resultDescription": "",
    }
  ]
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
| `failureCount`                 | number of failed transaction submissions |
| `txs`                      | list of transaction responses                            |
| `txid`                      | transaction ID                                          |
| `resultDescription`         | will contain the error on `failure` or empty on `success`   |
| `returnResult`              | will contain either `success` or `failure`                  |

### Send multi-transaction status query

#### Purpose:

This endpoint is used to check the statuses of multiple previously submitted transactions. It returns a [JSONEnvelope](https://github.com/bitcoin-sv-specs/brfc-misc/tree/master/jsonenvelope) with a payload that contains the transaction statuses. The purpose of the envelope is to ensure strict consistency in the message content for the purpose of signing responses.

> **body** when `Content-Type` is `application/json`:

```json
[
  "6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0",
  "aee1ca20212b2e9ac879c1d31639d4bcc34f9b761465d6567ceeb0365ba584e9",
  "18e6b339551fd177587aaf968281c0d400e20aa200349903bed0883b10b7612b"
]
```

#### Returns:

```json
{
  "payload": "{\"apiVersion\":\"1.1.0\",\"timestamp\":\"2020-06-15T12:09:37.394Z\",\"minerId\":\"03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031\",\"txSecondMempoolExpiry\":0,\"failureCount\":1,\"txs\":[{\"txid\":\"6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"blockHash\":\"00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09\",\"blockHeight\":208,\"confirmations\":2},{\"txid\":\"aee1ca20212b2e9ac879c1d31639d4bcc34f9b761465d6567ceeb0365ba584e9\",\"returnResult\":\"failure\",\"resultDescription\":\"Transaction in mempool but not yet in block\",\"blockHash\":\"\",\"blockHeight\":0,\"confirmations\":0},{\"txid\":\"18e6b339551fd177587aaf968281c0d400e20aa200349903bed0883b10b7612b\",\"returnResult\":\"success\",\"resultDescription\":\"\",\"blockHash\":\"0000000000000000030f9231d25b599c0e84a53e7011f29a50fbe4b87837ae79\",\"blockHeight\":206,\"confirmations\":4}]}",
  "signature": "3044022100a5ecc6825777e4d9effea54ccfaa1b9a73e01c71eed477d314e473b514f2e433021f5fa090020f95f85317faf0fcb3e535d6294f6e7f44c1c58be6557690e921b7",
  "publicKey": "023aa659d7141c9b09ab7e406f041f20d901076cb4ed822e95338f1499e2292070",
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
  "apiVersion": "1.1.0",
  "timestamp": "2020-06-15T12:09:37.394Z",
  "minerId": "03fcfcfcd0841b0a6ed2057fa8ed404788de47ceb3390c53e79c4ecd1e05819031",
  "txSecondMempoolExpiry": 0,
  "failureCount": 1,
  "txs": [
    {
      "txid": "6bdbcfab0526d30e8d68279f79dff61fb4026ace8b7b32789af016336e54f2f0",
      "returnResult": "success",
      "resultDescription": "",
      "blockHash": "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09",
      "blockHeight": 208,
      "confirmations": 2,
    },
    {
      "txid": "aee1ca20212b2e9ac879c1d31639d4bcc34f9b761465d6567ceeb0365ba584e9",
      "returnResult": "failure",
      "resultDescription": "Transaction in mempool but not yet in block",
      "blockHash": "",
      "blockHeight": 0,
      "confirmations": 0,
    },
    {
      "txid": "18e6b339551fd177587aaf968281c0d400e20aa200349903bed0883b10b7612b",
      "returnResult": "success",
      "resultDescription": "",
      "blockHash": "0000000000000000030f9231d25b599c0e84a53e7011f29a50fbe4b87837ae79",
      "blockHeight": 206,
      "confirmations": 4,
    }
  ]
}
```

| field                   | function                                                |
| ----------------------- | ------------------------------------------------------- |
| `apiVersion`            | version of merchant api spec                            |
| `timestamp`             | timestamp of payload document                           |
| `minerId`               | minerId public key of miner                             |
| `txSecondMempoolExpiry` | duration (minutes) Tx will be kept in secondary mempool |
| `failureCount`                 | number of failed transaction queries |
| `txs`                      | list of transaction responses                            |
| `txid`                  | transaction ID                                          |
| `returnResult`          | will contain either `success` or `failure`                  |
| `resultDescription`     | will contain the error on `failure` or empty on `success`   |
| `blockHash`             | hash of tx block                                        |
| `blockHeight`           | hash of tx block                                        |
| `confirmations`         | number of block confirmations                           |

---

### Authorization/Authentication and Special Rates

Merchant API providers may wish to offer different fee rates to specific customers. To do this they would need to add an extra layer to enable authorization/authentication. An example implementation would be to use JSON Web Tokens ([JWT](https://jwt.io/)) issued to specific users. The users would then include that token in their HTTP header and as a result recieve their user specific fee rates.

---

## Data Flow Diagram

![data flow](out/mapi-bip270/data_flow.png)
