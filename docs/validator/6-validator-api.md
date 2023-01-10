---
description: Skip Validator API
title: Validator API
sidebar_position: 6
---

# Validator API

## GET Disconnected Validators

- This request returns the operator addresses of all disconnected validators for the requested chain
  :::info Disconnected Validator
  A validator is considered disconnected if none of the validator's nodes (sentry / validator) are currently peered with the sentinel. **this means that the node is not receiving bundles!!**
  :::
### URI: `/disconnected_validators`

### Query String Params:
* `chain_id`: **(Required)** Chain ID of the target chain (e.g. "juno-1")
### Example:

- **Request**
  - `curl -X GET https://skip.money/disconnected_validators?chain_id=evmos_9001-2`
- **Response**
  ```JSON
  {
      "disconnected_validators": [
          "evmosvaloper1dk4ste222kp256ky00553nexna0rh38hx2ur6c",
          "evmosvaloper10qpycc2egucukw8afcz4us7xlxxmfwh6rscvjz",
          "evmosvaloper10xhy8xurfwts9ckjkq0ga92mrjz9txyy6y8l2g",
          "evmosvaloper1rw4nendrehkr5tkhjswzy8ts97dvwg7rc0u0kp",
          "evmosvaloper1w9m6p7ctu4gkdsr8plle997h25rzsa96xlzfat",
          "evmosvaloper1qp49y6vh8vvv5yf8ule8fwx6sss82ncz39tunl",
          "evmosvaloper1tczjwjrr36e0l5jjqnwrp95kv8mk57zdmxse4m"
      ]
  }
  ```

## GET Registered Validators with Config Info

- This request returns the information (detailed below) for each validator running skip on the specified chain
  - `OperatorAddress` - Operator address of the validator
  - `Moniker` - Moniker of the validator
  - `FrontRunningProtection` - Whether or not the validator accepts front-running bundles
  - `PaymentPercentage` - Percentage of rewards earned from auction-fees that goes to the validator
  - `ValProfit` - The profit this validator has accrued to date
  - `NetworkProfit` - The profit this validator has generated in network fees from bundles for the network
  - `ChainID` - Chain id of network requested.
### URI: `/validator_infos`

### Supported Verb: `GET`
### Query String Params:
* `chain_id`: **(Required)** Chain ID of the target chain (e.g. "juno-1")


### Example:

- **Request**
  - ` curl -X GET https://skip.money/validator_infos?chain_id=uni-5`
- **Response**
  ```JSON
  {
      {
  		"OperatorAddress": "junovaloper1gfaavqqg79tgcmgws6ys7yvchtc3fl42zjw43l",
  		"Moniker": "web34ever",
  		"FrontRunningProtection": true,
  		"PaymentPercentage": 10,
  		"ValProfit": 10945,
  		"NetworkProfit": 1089145,
  		"ChainID": "uni-5"
  	},
  	{
  		"OperatorAddress": "junovaloper1p0vtyfaej2u6fdwlxcdp55v5uxn6433uc4msdz",
  		"Moniker": "Stake\u0026Relax Validator",
  		"FrontRunningProtection": true,
  		"PaymentPercentage": 50,
  		"ValProfit": 2605750,
  		"NetworkProfit": 2984250,
  		"ChainID": "uni-5"
  	}
      ...
  }
  ```

## GET Validator Connection Status 

- This request returns the configuration information for each validator registered for Skip Select on the specified chain
  - `moniker` - Moniker of the validator
  - `connection_status` - Whether or not the validator is currently peered with the sentinel ("disconnected" or "connected")
  - `time_since_connected` - Time in hours since the validator was last peered with the sentinel
    - `time_since_connected`: will be `-1` for validators who have never connected to the sentinel
  - `version` - Which version of `mev-tendermint` the validator is running

### URI: `/connection_data`

### Supported Verb: `GET`
### Query String Params:
* `chain_id`: **(Required)** Chain ID of the target chain (e.g. "juno-1")

### Example:

- **Request**
  - `curl -X GET https://skip.money/connection_data?chain_id=uni-5`
- **Response**
  ```JSON
  {
      {
  		"moniker": "Stake\u0026Relax Validator",
  		"connection_status": "connected",
  		"time_since_connected": 4,
  		"version": "NA"
  	},
  	{
  		"moniker": "Please valiDate Me",
  		"connection_status": "disconnected",
  		"time_since_connected": -1,
  		"version": "NA"
  	}
      ...
  }
  ```
