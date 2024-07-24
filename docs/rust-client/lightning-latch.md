---
sidebar_position: 3
---

# Lightning Latch example

To perform an lightning latch swap of a mercurylayer coins and a lightning network payment, first create two separate wallets:

```
cargo run create-wallet wallet1
cargo run create-wallet wallet2
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

Generate a transfer address for `wallet2` with the `-b` flag (this generates a `batch_id` for the atomic transfer. 

```
cargo run new-transfer-address wallet2 -b
```
