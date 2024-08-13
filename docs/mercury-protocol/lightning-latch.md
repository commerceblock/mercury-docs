---
sidebar_position: 3
---

# Lightning Latch

A *latch* transfer enables a statecoin to be transferred only on condition of the sucessful payment of a Lightning network invoice. This is a protocol that enforces atomicity of a statecoin transfer and lightning payment - it can be used for the sale of a statecoin UTXO for an arbtrary amount of bitcoin in a private and non-custodial way without counterparty risk. 

The latch transfer protocol uses the method of *Hodl invoices* (https://guide.bolt.fun/guide/invoices/hodl-invoice) where funds are locked until the payment hash pre-image is revealed. A hodl invoice can be resolved in one of two ways: 

1. The payment is complete when the recipient releases the preimage (to the payment route). 
2. The payment is canceled if the recipient does not release the preimage and the invoice expires.

Using this mechanism, a lightning network payment can be made but only completed if some condition is met - specifically if a specified statecoin transfer is made and verified. In this case the statechain entity (i.e. the mercury server) can both generate and release the payment hash pre-image on successful completion of the statecoin transfer in order to enforce atomicity. In this case the mercury server is trusted, but never has custody of any assets and never learns anything about the payment. 

## Protocol

The statecoin server database will have 4 additional values for each `statechain_id`: `batch_id` (string), `batch-time` (integer), `pre-image` (string), `locked_1` (boolen) and `locked_2` (boolen). By default `batch_id`, `batch-time` are null and `locked` is `false`. The server is configured with `batch-timeout` parameter in seconds (in practice this will be some number of minutes to enable both parties to complete the trade). The `batch-timeout` must be less than the hold payment locktime, in order to prevent the lightning network payment to be double-spent within the batch time. 

The transfer will then proceed as follows:

1. There are two parties: party 1 has a statecoin they want to sell and party 2 wants to receivie it and pay an agreed price via LN. 
2. Party 1 generates a `batch_id` (this is just a random UUID) and shares with party 2.
3. Party 1 calls `/transfer/paymenthash` with the `batch_id` and authenticated `statechain_id` for the coin. The server generates a secret `preimage` and stores it in the statecoin table with a new row and `batch-time` set as current time and `locked_1` to `true` and `locked_2` to `true`, and returns the `paymenthash` to Party 1. 
4. Party 2 generates a `sc` addresses that they want the UTXO sent to. This is sent to Party 1. 
5. Party 1 generates an invoice with the `paymenthash` and sends to Party 2. Party 2 can verify that the server has generated the pre-image to this by calling `/transfer/paymenthash` with the `batch_id` provided by Party 1. 
6. Party 1 performs `/transfer/sender {statechain_id, batch_id, auth_sig, new_user_auth_key}` with the `batch_id` paying to the party 1 `new_user_auth_key` (derived from their address) and then creates and uploads the encrypted transfer message.
7. Party 2 makes the LN payment for the invoice, but cannot complete this as they don't know the pre-image.
8. Party 1 verifies the payment is pending and calls `/transfer/unlock` with `statechain_id` and signed with their `auth_key`. If time is within `batch-time` + `batch-timeout` this sets the `locked_1` value to `false`.
9. Party 2 retrives and verifies the transfer message for the UTXO with their `new_user_auth_key` and calls `/transfer/unlock` with `statechain_id` and signed with their `new_user_auth_key`. If time is within `batch-time` + `batch-timeout` this sets the `locked_2` value to `false`.
10. Party 1 then calls `/transfer/preimage` with `statechain_id` and signed with their `new_user_auth_key`. If `locked_1` and `locked_2` and both false, then the `preimage` is returned and the LN payment can be finalised.
11. Party 2 completes `/transfer/receiver`. If `locked_1` and `locked_2` and both false then the keyupdate is completed. If either is `true` then `/transfer/receiver` will return an error. 

If both of the `/transfer/unlock` operations are not completed within the `batch-timeout` then both: 1. the LN invoice will timeout and cancel and 2. Party 1 performs another `/transfer/sender` and `/transfer/receiver` operation returning control of the coin. 

The outcomes of this protocol:

All participants call `/transfer/unlock` within the `batch-time` then `/transfer/receiver` can be performed and the pre-image revealed. 

Any participant doesn't call `/transfer/unlock` within the `batch-time` (because they have failed to verify either that the transfer message is incorrect, or the LN payment was not made correctly) then no-one can call `/transfer/receiver` but party 1 can call `/transfer/sender` again to recover the coin, and the LN payment will fail. 

Calling `/transfer/sender` on the coin a second time will be blocked until `batch-time` expires. 
