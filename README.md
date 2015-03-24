MUEWallet
an open-source secure MonetaryUnit wallet platform for friends and companies.
Easy-to-use multisignature MonetaryUnit wallet, bringing corporate-level security to ordinary people.

Based on https://github.com/bitpay/copay

# About MUEWallet

General
-------

*MUEWallet* implements a multisig wallet using p2sh addresses. It supports multiple wallet configurations, such as 3-of-5
(3 required signatures from 5 participant peers) or 2-of-3.  To create a multisig wallet shared between multiple participants,
*MUEWallet* needs the public keys of all the wallet participants. Those public keys are incorporated into the
wallet configuration and are combined to generate a payment address with which funds can be sent into the wallet.  

To unlock the payment and spend the wallet's funds, a quorum of participant signatures must be collected
and assembled in the transaction. The funds cannot be spent without at least the minimum number of
signatures required by the wallet configuration (2 of 3, 3 of 5, 6 of 6, etc).
Each participant manages their own private key, and that private key is never transmitted anywhere.
Once a transaction proposal is created, the proposal is distributed among the
wallet participants for each participant to sign the transaction locally.
Once the transaction is signed, the last signing participant will broadcast the
transaction to the MonetaryUnit network using a public API (defaults to the [MUE Insight API](https://github.com/MonetaryUnit/insight-api)).

*MUEWallet* also implements BIP32 to generate new addresses for the peers. The public key each participant contributes
to the wallet is a BIP32 extended public key. As additional public keys are needed for wallet operations (to produce
new addresses to receive payments into the wallet, for example) new public keys can be derived from the participants'
original extended public keys. Each participant keeps their own private keys locally. Private keys are not shared.
Private keys are used to sign transaction proposals to make a payment from the shared wallet.

Security model
--------------
*MUEWallet* peers encrypt and sign each message using
ECIES (a.k.a. asynchronous encryption) as decribed on
[http://en.wikipedia.org/wiki/Integrated_Encryption_Scheme].

The *identity key* is a ECDSA public key derived from the participant's extended public
key using a specific BIP32 branch. This special public key is never used for MUE address creation, and
should only be known by members of the WR.
In *MUEWallet* this special public key is named *copayerId*.  The copayerId is hashed and the hash is used to
register with the peerjs server (See SINs at https://en.bitcoin.it/wiki/Identity_protocol_v1). This hash
is named *peerId*.

Registering with a hash avoids disclosing the copayerId to parties outside of the WR.
Peer discovery is accomplished using only the hashes of the WR members' copayerIds. All members of the WR
know the full copayerIds of all the other members of the WR.


Payment Protocol
----------------

*MUEWallet* supports BIP70 (Payment Protocol), with the following current limitations:

  * Only one output is allowed. Payment requests is more that one output are not supported.
  * Only standard Pay-to-pubkeyhash and Pay-to-scripthash scripts are supported (on payment requests). Other script types will cause the entire payment request to be rejected.
  * Memos from the customer to the server are not supported (i.e. there is no place to write messages to the server in the current UX)
