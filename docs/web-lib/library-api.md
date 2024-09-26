---
sidebar_position: 2
---

# Web library API

Config:

```js
const clientConfig = {
  esploraServer: "https://0.0.0.0",
  statechainEntity: "http://0.0.0.0:8500",
  network: "testnet",
  feeRateTolerance: 5,
  confirmationTarget: 2,
  maxFee: 1
};
```

`createWallet(clientConfig, <wallet_name>);` to create a wallet

`newToken(clientConfig, <wallet_name>);` to create a new deposit token

`getDepositBitcoinAddress(clientConfig, <wallet_name>, <amount>)` creates a deposit address for amount (in sats). 

`listStatecoins(clientConfig, <wallet_name>)` shows wallet coins and status

`newTransferAddress(<wallet_name>)` generates a statechain address to receive statechain coins

`transferSend(clientConfig, <wallet_name>, <statechain-id>, <statechain-address>)` transfers the specified statechain coin to the specified address

`transferReceive(clientConfig, <wallet_name>)` scans for new statechain transfers and completes transfer receive. 

`withdrawCoin(clientConfig, <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>)` withdraws the statechain coin to the specified bitcoin address

`broadcastBackupTransaction(clientConfig, <wallet_name>, <statechain-id>, <btc-address>, <optional_fee_rate>)` broadcasts the backup transaction to the network