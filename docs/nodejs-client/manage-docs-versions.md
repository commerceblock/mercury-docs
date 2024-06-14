---
sidebar_position: 1
---

# Tutorial - nodejs

To use the NodeJS client app, first clone the mercurylayer repository:

```
git clone https://github.com/commerceblock/mercurylayer.git
```

Then switch to the `dev` branch:

```
cd mercurylayer
git checkout dev
```

Then change directory to the nodejs client library and link:

```
cd clients/libs/nodejs
npm link
```

Then change directory to the standalone client and link the library, and install dependencies:

```
cd ../../apps/nodejs
npm link mercurynodejslib
npm install
```

In this directory is the `config/default.json` file for the client. This needs to be edited (using any text editor, e.g. vi or nano) to set the mercury server endpoint and Electrum server URL. 

For the purposes of demonstration, use the following `Settings.toml`: 

```
{
    "statechainEntity": "http://test.mercurylayer.com:8500",
    "electrumServer": "tcp://mutinynet.com:50001",
    "electrumType": "electrs",
    "network": "signet",
    "feeRateTolerance": 5,
    "databaseFile": "wallet.db",
    "confirmationTarget": 2,
    "maxFeeRate": 1
}
```

The test mercury key server URL is: `http://test.mercurylayer.com:8500`

> Note that this is a test server. It is free to use, but there is no guarantee of persistence or security. Use only for testnet or signet coins. 

Once the settings are complete, initialise a wallet using:

```
node index.js create-wallet <wallet_name>
```

Running this with name `test_wallet` should return an object like:

```
{
  "name": "test_wallet",
  "mnemonic": "chronic ensure limb ripple avocado satisfy doll text lunar among quantum pact",
  "version": "0.1.0",
  "state_entity_endpoint": "http://test.mercurylayer.com:8500",
  "electrum_endpoint": "tcp://mutinynet.com:50001",
  "network": "signet",
  "blockheight": 1169081,
  "initlock": 100000,
  "interval": 6,
  "tokens": [],
  "activities": [],
  "coins": [],
  "settings": {
    "network": "signet",
    "block_explorerURL": null,
    "torProxyHost": null,
    "torProxyPort": null,
    "torProxyControlPassword": null,
    "torProxyControlPort": null,
    "statechainEntityApi": "http://test.mercurylayer.com:8500",
    "torStatechainEntityApi": null,
    "electrumProtocol": "tcp",
    "electrumHost": "mutinynet.com",
    "electrumPort": "50001",
    "electrumType": "electrs",
    "notifications": false,
    "tutorials": false
  }
}
```

In order to use the mercury layer key server, an access token is required in order to create a shared key. For the test server, tokens can be generated as follows from the command line:

```
node index.js new-token
```

Which will return a `token_id` e.g. `e7d8c299-7121-48b3-bc78-8c70bf4c9691`

With a valid token recieved by the client, it is then possible to generate a shared key and address to deposit testnet bitcoin into:

```
node index.js new-deposit-address <wallet_name> <amount>
```

For example the following will initialise a coin for an amount of 200000 sats. 

```
node index.js new-deposit-address test_wallet 200000
```

This command will then generate the shared key with the server and display the address along with the `statechain_id`, e.g.:

```
{
  "deposit_address": "tb1p6df39hvwsaaahyg9h9q4w48r3k9vdzqrtcw5shccudnavcs3cduq83ue9p",
  "statechain_id": "b77944a3deec40809a50db19327dce2d"
}
```

Copy the address and pay the specified amount to it. For the mutinynet signet network, this payment can be made directly via the faucet: [faucet.mutinynet.com](https://faucet.mutinynet.com/)

Once paid, run:

```
node index.js list-statecoins <wallet-name>
```

To list the coins in the wallet and complete the deposit process. 

This will return e.g.:

```
[
  {
    "statechain_id": "b77944a3deec40809a50db19327dce2d",
    "amount": 100000,
    "status": "INITIALISED",
    "adress": "tml1qqpjchvn6ruamvvwq2hmffq6vedvvv7dv8krqg6h3e4wmjyjuzrcslgrepkddgq8yggf0pa6wjcugs4vsruf98p3f067zdc2ych02rp423ls3nhe64"
  }
]
```

Once the `status` shows `CONFIRMED`, the coin can be transferred. 

To generate a statecoin receieve address, run:

```
node index.js new-transfer-address <wallet_name>
```

Which will generate a mercury layer address to recieve a coin to. e.g.:

```
{
  "new_transfer_address:": "tml1qqp7m5tc9auxgwka84tez9vksky2lsp5uftyhhduakt75j8yq46wh3crh26fwzxw2akc43d9m0pvuhmuq57tcdtw7pz96zfsz8ck3d3jjf3q04re26"
}
```

This address can then be used to send a specified coin (with `statechain_id`) coin to a specified statechain address:

```
node index.js transfer-send <wallet_name> <statechain-id> <statechain-address>
```

For example, with the confirmed coin above (`statechain_id: "b77944a3deec40809a50db19327dce2d"`)

```
node index.js transfer-send test_wallet tml1qqp7m5tc9auxgwka84tez9vksky2lsp5uftyhhduakt75j8yq46wh3crh26fwzxw2akc43d9m0pvuhmuq57tcdtw7pz96zfsz8ck3d3jjf3q04re26 b77944a3deec40809a50db19327dce2d
```

This will return:

```
{
  "Transfer": "sent"
}
```

The recevier then runs the following command to finalise the receipt of the coin and update the key share:

```
node index.js transfer-receive <wallet_name>
```

The coin will then appear in the coins list:

```
node index.js list-statecoins <wallet-name>
```

To send the coin back to a standard on-chain bitcoin address, simply run:

```
node index.js withdraw <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>
```

For the demo test coins, these can be sent back to an address generated by the faucet: [faucet.mutinynet.com](https://faucet.mutinynet.com/)

In case the coin expires and the mercury server is unavailable, the backup transaction can be used as follows:

```
node index.js broadcast-backup-transaction <wallet_name> <statechain-id> <btc-address> <optional_fee_rate>
```
