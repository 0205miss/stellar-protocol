## Preamble

```
SEP: xxxx
Title: Offline Transaction Verification
Author: Leigh McCulloch (@leighmcculloch)
Status: Draft
Discussion: TBD
Created: 2021-10-14
Version 0.1.0
```

## Summary

This proposal defines an API and a process that a wallet can use to provide
confidence that a transaction is confirmed to another party who has no
internet connection and therefore no way to verify the state of the
transaction themselves.

## Dependencies

This proposal has no dependencies on Stellar protocol version, or on any
CAP, or another SEPs.

## Motivation

Wallets are popularly present on phones and phones typically have internet
connections, but in many regions it is common for merchants, vending
machines, and devices accepting payments to not have internet connections
today if they're current only form of payment is coins.

If one party in any payment has network connectivity, it is possible for
the other party to verify that the payment has succeeded on chain without
having network access. The cryptographic primitives exist to support this
scenario, and this scenario reduces the complexity of introducing
blockchain payments to use cases that are currently serviced by offline
machinary, or machinary currently only accepting cash.

People paying typically have network connected devices and wallets should
leverage this capability and the cryptographic tools available to them to
allow the payee to prove the validity of a payment without network
connectivity.

## Abstract

This proposal defines a process that one participant can use to prove to
another participant that a transaction has been confirmed on the Stellar
network.

This proposal speaks concretely about payments, because the Stellar
network is an open payments network, but any transaction operation is
supported.

## Specification

There are three components to implementing offline payment verification.

1. A transacting device, or **payer**, that has network connectivity.
2. A participating device, or **payee**, that may have no connectivity.
3. A transaction result verification server, or **server**, controlled by the **payee**.
4. A **private key** held only by the **server**.
5. A **public key**, derived from the **private key**, held by the **payee**.

### Payment Flow

The **payee** transmits locally an invoice with the following details:
- A payment amount.
- A destination account.
- A transaction memo that must be unique to this transaction, ideally either random, or some identifier that will not reasonably appear on another **payee** using the same destination account.
- The **server** URL.

The **payer** builds a transaction containing a payment operation. The transaction and operation have the memo, amount, and destination set per the details in the invoice.

The **payer** submits the transaction to the network and waits for a successful result.

The **payer** hashes the transaction and sends the transaction hash to the **server**.

The **server** looks up the transaction by its hash and collects the result of the transaction from a trusted validator or Horizon instance.

The **server** signs a payload formed from the serialized binary of the following XDR struct using the **private key**. The following struct is equivalent to concatenating the 32-byte transaction hash with the serialized binary of the `TransactionResult` XDR struct as defined in the Stellar protocol.

```xdr
struct
{
    Hash transactionHash;
    TransactionResult transactionResult;
};
```

The **server** returns the signature to the **payer**.

The **payer** encodes a proof of transaction result, made up of the transaction, transaction result, and the **server**'s signature into a message and transmits the message to the **payee**.

The **payee** hashes the transaction, produces the same payload as above, and verifies the **server**'s signature using the **public key**.

The **payee** checks that the transaction result indicates success, and that the transaction contains the payment to the destination account. If these checks pass, the **payee** accepts that the transaction as confirmed.

### Server API

TODO

### Transmission

The **payee** and **payer** transmit the invoice, and proof of transaction result.

#### Transmission Messages

TODO

#### Transmission Size

The data transferred is reasonably small and so suitable for many mediums where bandwidht is low or data size is limited.

The minimum size required to transmit the invoice from the **payee** to the **payer** is 80 bytes plus the number of bytes for the URL.

```
Payment Amount = 8 bytes
Destination Account = 36 bytes
Memo = 36 bytes
URL = N bytes
Total = 80+N bytes
```

The minimum size required to transmit the proof of transaction result from the **payer** to the **payee** 281 bytes.

```
Transaction = 216 bytes
Transaction Result = 33 bytes
Server Signature = 32 bytes (assuming an ed25519 signature)
Total = 281 bytes
```

Transmissions requiring a textual representation, such as QR codes, will increase the data size.

## Design Rationale

TODO

## Security Concerns

TODO

## Limitations

This proposal does not specify the transmission method.

## Implementations

None at this time, although ideally this proposal if accepted is implemented in
Stellar client SDKs.

## Inspiration




[SEP-1]: https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md
[`CURRENCIES`]: https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md#currency-documentation
[SEP-1 stellar.toml]: https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md
[SEP-2 Federation protocol]: https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0002.md
[SEP-11 Txrep]: https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0011.md
