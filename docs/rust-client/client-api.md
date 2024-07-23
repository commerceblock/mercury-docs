---
sidebar_position: 4
---

# Client API

`cargo run create-wallet <wallet_name>` to create a wallet

`cargo run new-token` to create a new token

`cargo run new-deposit-address <wallet_name> <token> <amount>` creates a deposit address

`cargo run list-statecoins <wallet_name>` shows wallet coins

`cargo run new-transfer-address <wallet_name>` generates a statechain address to receive statechain coins

`cargo run transfer-send <wallet_name> <statechain-id> <statechain-address> ` transfers the specified statechain coin to the specified address

`cargo run transfer-receive <wallet_name>` scans for new statechain transfers

`cargo run withdraw <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>` withdraws the statechain coin to the specified bitcoin address

`cargo run broadcast-backup-transaction <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>` broadcasts the backup transaction to the network
