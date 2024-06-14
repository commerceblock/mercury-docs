---
sidebar_position: 1
---

# Introduction

Mercury layer is an implementation of a system the uses a blind co-signing and key-update service to enable statechains on Bitcoin. The statechain protocol allows the transfer of ownership of Bitcoin unspent transaction outputs (UTXOs) that remain under the full custody of the owner at all times, while benefiting from instant and zero cost transactions. The ability to perform this transfer without requiring the confirmation (mining) of on-chain transactions has advantages in a variety of different applications. 

This documentation covers the description of the Mercury Layer architecture and protocol, the specification Mercury API and instructions for the deployment and operation of the separate components of the system.

## Overview

The Mercury layer system employs a service provider (the mercury layer blind server) that generates and updates key shares (or key fragments) on request in addition to a count of partial blinded signatures. By updating (‘cycling’) a key share and reporting the number of  partial blinded signatures generated for the share, the ownership of individual UTXOs can be transferred between counterparties instantly and at zero marginal cost in a secure and fully self-custodial way. The blind key-update server never has control or custody, and is never aware of the identity of any specific UTXO. 

This system requires that all bitcoin transaction operations and statechain verification are performed entirely by client-side software. 

## Mercury layer service

The mercury layer service generates a private key share `s1` on initialisation of a session. In order to initialise a session, a client must provide a valid `token_id` which controls access to the service (and is generated separately typically after payment of a fee). 

The client then initialises a session with an `auth_pubkey` which is used to authenticate all subsequent messages with the server. The server responds with the public elliptic curve point corresponding to the session private key share `s1`. 

Once initialised, the server can then perform two operations using the key share: 

- Partial signature generation - the server uses the key share to compute a partial signature from a blinded challenge value provided by the client.
- Key update - an encrypted value is sent to the server and used to update the key share along with a new `auth_pubkey` for the updated key. The previous key share is then deleted securely. 

The server does not ever receive any other information regarding the client state. 

##  Statechain transfers

Using the mercury service, clients can use the key share update rules applied by the server to securely transfer ownership of a Bitcoin UTXO to a new client while maintaining self-custody and without requiring a blockchain transaction. 

This is achieved by depositing an amount of bitcoin to an address which is formed in part from the server public key share, and then requesting a partial signature from the server to either spend the coin or create ‘backup transactions’ to protect against server unavailability (i.e. unilateral on-chain exit). Transfers to new clients are secured by the server key update, enabling the UTXO deposit address to stay the same, while removing the ability of a previous owner to steal the funds. 

This key update mechanism is additionally combined with a system of backup transactions which can be used to claim the value of the UTXO by the current owner in the case the server does not cooperate or has disappeared. The backup transaction is created by the current owner at the point of transfer, paying to an address controlled by the new owner. To prevent a previous owner (i.e. not the current owner) from broadcasting their backup transaction and stealing the deposit, the nLocktime value of the transaction is set to a future specified block height. Each time the ownership of the UTXO is transferred, the `nLocktime` is decremented by a specified value, therefore enabling the current owner to claim the coin before any of the previous owners.

The decrementing timelock backup mechanism limits the number of transfers that can be made within the lock-out time. The user is responsible for submitting backup transactions to the Bitcoin network at the correct time, and applications can do this automatically.

The life-cycle of a coin in the statechain, key reassignment, and closure is summarised as follows:

1. The first owner initiates a statechain by paying an amount of bitcoin to an address where the corresponding public key is formed from both the first owner public key share and the server public key share. The first owner creates a timelocked backup transaction spending the statechain UTXO to an address fully controlled by the first owner which can be confirmed after the `nLocktime` block height in case the server stops cooperating.
2. The owner can verifiably transfer ownership of the UTXO to a new party (Owner 2) via a key update procedure that overwrites the private key share of server that invalidates the first owner private key and activates the new owner private key share. Additionally, the transfer incorporates the signing of a new backup transaction paying to an address controlled by the new owner which can be confirmed after a new nLocktime block height, which is reduced (by an accepted confirmation interval) from the previous owners backup transaction `nLocktime`.
3. This transfer can be repeated multiple times to new owners as required (up until the most recent recovery `nLocktime` reaches a lower limit determined by the current Bitcoin block height).

At any time the most recent owner can create and sign a transaction spending the UTXO to an address of the most recent owner's choice (i.e. closure of the statechain).

## Blinding

The Mercury layer server is *blind* - it is not aware of bitcoin, and does not perform any verifcation of transactions. 

The server cannot know or be able to derive in any way the following values:

- The TxID:vout of the statecoin UTXO
- The address (i.e. public key) of the UTXO
- Any signatures added to the bitcoin blockchain for the coin public key.

This means that the server cannot:

- Learn of the shared public key it is co-signing for.
- Learn of the final (unblinded) form of any signatures it co-generates.
- Verify ANY details of backup or closure transactions (as this would reveal the statecoin TxID).

All verification of backup transaction locktime decrementing sequence must be performed client side (i.e. by the receiving client). This requires that the full statechain and sequence of backup transactions is sent from sender to receiver and verified for each transfer (this can be done via the server with the data encrypted with the receiver public key). 

### Blind two-party Schnorr signatures

Mercury layer employs Schnorr signatures via Taproot addresses for statecoins. To enable a signature to be generated over a shared public key (by the two private key shares of the server and owner) a blinded variant of the Musig2 protocol is employed. In this variant, one of the co-signing parties (the server) does not learn of 1) The full shared public key or 2) The final signature generated. 

### Client transaction verification

In the blinded mercury layer protocol, the server cannot verify what it signs, but can only state **HOW MANY** unique signatures it has generated for a specific shared key, and it will return this number when queried. The client will then have to check that every single previous backup transaction signed has been correctly decremented, AND that the total number of value backup transactions it has verified matches the number of blind partial signatures the server has generated. This will then enable a receiving client to verify that no other valid transactions spending the statecoin output exist. 

When it comes to closure the client can just create any transaction it wants to end the chain. 
