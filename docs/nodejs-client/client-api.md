---
sidebar_position: 2
---

# Client API

`node index.js create-wallet <wallet_name>` to create a wallet

`node index.js new-token` to create a new token

`node index.js new-deposit-address <wallet_name> <amount>` creates a deposit address

`node index.js list-statecoins <wallet_name>` shows wallet coins

`node index.js new-transfer-address <wallet_name>` generates a statechain address to receive statechain coins

`node index.js transfer-send <wallet_name> <statechain-id> <statechain-address> ` transfers the specified statechain coin to the specified address

`node index.js transfer-receive <wallet_name>` scans for new statechain transfers

`node index.js withdraw <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>` withdraws the statechain coin to the specified bitcoin address

`node index.js broadcast-backup-transaction <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>` broadcasts the backup transaction to the network