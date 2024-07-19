---
sidebar_position: 2
---

# Atomic swap

To perform an atomic swap of two separate mercurylayer coins, first create four separate wallets:

```
cargo run create-wallet wallet1
cargo run create-wallet wallet2
cargo run create-wallet wallet3
cargo run create-wallet wallet4
```

Generate a new deposit token:

```
cargo run new-token
```

```
cargo run cargo run new-deposit-address wallet1 <token> 100000
```

Deposit signet bitcoin to the generated address and wait for coin status getting `CONFIRMED`, then:

```
cargo run list-statecoins wallet1 
```

Repeat this for `wallet2`:

```
cargo run new-token
```

```
cargo run cargo run new-deposit-address wallet2 <token> 100000
```

Deposit signet bitcoin to the generated address and wait for coin status getting `CONFIRMED`, then:

```
cargo run list-statecoins wallet2 
```

Generate a transfer address for `wallet3` with the `-b` flag (this generates a `batch_id` for the atomic transfer. 

```
cargo run new-transfer-address wallet3 -b
```

Returning, e.g.:

```
New transfer address: tml1qqprzt7lf9p2zcjwflh6cywsce2v9mkl6ns8gucahd8mhlcq95cxj6gz5mrr3m9yjekk75pshe2reylpud0utvtj88g86qvzng2d20rrs36qmlg40j # (example)
Batch Id: 2e9ac416-24e1-4c29-b4d7-5f7d35b062f8 # (example)
```

Generate a transfer address for `wallet4`:

```
cargo run new-transfer-address wallet4
```

Returning, e.g.:

```
New transfer address: tml1qqplxlutx9asvxycyd9yqf9vf0gk2j4cnxzqgt7csz09mt8hdttq25grrsr32ma25el9c760je3w3m305r0hskul3cjguwlx39jrsnr94ljswq87hv # (example)
```

Then `wallet1` send to `wallet3` supplying the `batch_id`:

```
cargo run transfer-send <wallet_name> <statechain_id> <recipient_address> <optional_batch_id>
```

E.g.:

```
cargo run transfer-send wallet1 b8a2a15d508743609600bb93b7d75c9e tml1qqprzt7lf9p2zcjwflh6cywsce2v9mkl6ns8gucahd8mhlcq95cxj6gz5mrr3m9yjekk75pshe2reylpud0utvtj88g86qvzng2d20rrs36qmlg40j 2e9ac416-24e1-4c29-b4d7-5f7d35b062f8
```

Then `wallet2` send their coin to `wallet4` supplying the same `batch_id`:

```
cargo run transfer-send wallet2 c3da091f8c3f46438c4f51aa6f7de2e4 tml1qqplxlutx9asvxycyd9yqf9vf0gk2j4cnxzqgt7csz09mt8hdttq25grrsr32ma25el9c760je3w3m305r0hskul3cjguwlx39jrsnr94ljswq87hv 2e9ac416-24e1-4c29-b4d7-5f7d35b062f8
```

Both receiving wallets then need to perform `transfer-recieve` within the timeout period specified by the server. 

```
cargo run transfer-receive wallet3
```

This will show the message `Statecoin batch still locked. Waiting until expiration or unlock.`, until the other coin is also succesfully recieved:

```
cargo run transfer-receive wallet4
```