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

Coin receiver generates a transfer address for `wallet2` and sends tothe coin sender. 

```
cargo run new-transfer-address wallet2 -b
```

Coin sender generate a payment hash and `batch_id` from the mercury server using the statechain ID of the coin being swaped:

```
cargo run payment-hash wallet1 <statechain_id>
```

The sender then generates a Lightning invoice using the payment hash (e.g. using the `AddHoldInvoice` RPC function in `LND`). 

The sender then performs transfer-sender with the `batch_id`:

```
cargo run transfer-sender wallet1 <statechain_id> <transfer-address> <batch_id>
```

The reciver then performes transfer-reciever to confirm coin validity and the coin is bacth-locked. 

```
cargo run transfer-reciever wallet2
```

The sender gives the invoice and to the coin receiver to pay. 

The reciever pays the HOLD-invoice. 

The sender confirms the HOLD-invoice is pending (e.g. using the `LookupInvoiceV2` RPC function in `LND`).

The sender then unlocks the coin:

```
cargo run confirm-pending-invoice wallet1 <statechain_id>
```

The reciver can now complete the transfer-reciver function. 

The sender can now retrie the hash pre-image to complete the lightning payment using:

```
cargo run retrieve-preimage wallet1 <statechain_id> <batch_id>
```

And then recieve the payment with LND command `SettleInvoice`. 



