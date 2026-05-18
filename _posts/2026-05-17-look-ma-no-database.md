---
layout: post
title: "Look Ma, No Database: Documents With Shared Lifecycle State"
date: 2026-05-17
---

Most business software already runs on "documents".

Vouchers, tickets, payment claims, receipts, licenses, account statements,
certificates, shipping documents, credentials. They may be JSON, PDF, XML, CBOR,
protobuf, plain text, or something more enterprise-shaped, but the pattern is
familiar: create a document, sign it, send it, store it, process it.

Developers already know how to do this. We know how to hash documents, sign
them, move them through apps, QR codes, NFC, RFID, HTTPS endpoints, files,
merchant systems, and mobile wallets. We know how to transform and store them.

The hard part starts when the document crosses an organizational boundary.

- A shop receives a voucher and wants to know whether it was already spent
somewhere else.

- A venue scans a ticket and wants to know whether the holder still controls it.

- A merchant receives a payment claim and wants to know whether that same claim
was also sent to another merchant.

- A verifier receives a credential and wants to know whether the issuer has
revoked or superseded it.

These are lifecycle questions. Not "what does the document say?" but "is this
document still the current, valid version that someone is allowed to act on?"
That is global state.

Today, the usual answer is: ask the database. But whose database?

That is where banks, card networks, platforms, issuer portals, SaaS registers,
and reconciliation systems enter the picture. Someone governs the shared
register. Everyone else integrates with it, waits for it, pays for it, or trusts
it.

EC Protocol explores a different split:

> Keep the documents wherever the application wants. But track their lifecycle
> through handles in a shared verification network.

The document carries the meaning. The hash of it is the handle. EC tells you whether that handle is still current.

Let's explore a style of document application that becomes possible in that
world.

## Let's Build A Shared Gift Card

Suppose you want to build a gift card, voucher, or payment-claim system. That is
not simple with today's infrastructure.

You could define a document format:

```text
issuer: MallAuth
value: 100
currency: DKK
expiry: 2027-12-31
schema: https://example.net/schemas/voucher-v1
issuer_public_key: ...
issuer_signature: ...
```

Define canonical serialization, so everyone hashes the same bytes.

Define signature rules.

Define what counts as a valid transfer, redemption, revocation, or split.

None of that requires special infrastructure. It is ordinary document
processing.

When the system hands the document to a customer, it does this:

```text
token_id = hash(document)
create  token_id,   owner = user_public_key
```

This registers the token in EC under the user's public key. The user's wallet
checks the issuer signature, verifies that the issuer is trusted, computes the
document hash, and confirms that the resulting token ID is controlled by the
user's key.

At this point, EC does not know the voucher terms. It does not know the customer
identity. It does not need the document contents.

It only knows that this hash exists as a token, and that the user's key
currently controls it.

The issuer created the meaning. EC tracks the lifecycle.

## Spending Part Of A Document

Now the user wants to spend 30 units at a shop and keep 70 as change.

The wallet, or a transformer endpoint, creates two child
documents. Each child is a full copy of the parent with one new section appended.

The 30-unit child:

```text
issuer: MallAuth
value: 100
currency: DKK
expiry: 2027-12-31
issuer_public_key: ...
issuer_signature: ...

---
split:
  value: 30
  salt: random_salt
```

The 70-unit change document:

```text
issuer: MallAuth
value: 100
currency: DKK
expiry: 2027-12-31
issuer_public_key: ...
issuer_signature: ...

---
split:
  value: 70
  salt: same_random_salt
```

The wallet or a transformer endpoint computes:

```text
parent_token = hash(parent_document)
shop_token   = hash(child_document_30)
change_token = hash(child_document_70)
```

Then it submits one atomic EC transaction:

```text
destroy parent_token, signature(user_public_key)
create  shop_token,   owner = shop_public_key
create  change_token, owner = user_public_key
```

Notice that the signature for the existing parent_token is part of the
transaction, and that the public key of the change_token does not need to be the
same key used for the parent.

This write requires access to an EC node with write capability.
A wallet provider, issuer, merchant, gateway, or user-operated node could provide that
path. Reads are open, so even if the merchant submits the actual transaction,
the user can check the result independently.

But the document logic itself is ordinary:

1. Copy parent document.
2. Append a section.
3. Hash the new document.
4. Submit the lifecycle update with valid signatures.

There is no voucher balance database in the middle.

There is still shared infrastructure. The difference is that it tracks document
lifecycle by hash, not application-specific balances, accounts, or voucher
records.

## Verification Is Mostly Hashing

When the shop receives the 30-unit document, it can verify it without asking the
issuer for the full document history.

Because each child is a copy of the parent with a section appended, the shop can
reconstruct the ancestry by removing sections from the end. For this split, the
30-unit child also contains enough information to derive the matching 70-unit
change document: the original value, the split amount, and the shared salt.

For example:

```text
hash(synthetic_child_document_70)
hash(child_document_30)
hash(parent_document)
```

If the document had gone through five transfers or splits, the shop could derive
those earlier states the same way: remove the last section, hash; remove the
next section, hash; continue back to the issuer-signed root.

Then the shop queries EC for those token IDs.

Those reads are open. A verifier does not need to be the issuer, the owner, or a
member of a private platform. Any node can be an entry point for queries, and a
client can discover the network from a small set of bootstrap nodes.

The shop checks:

- the issuer signature on the root document
- each previous token was consumed correctly
- the current document state is live
- the current token is owned by the expected key
- no visible conflict exists

For low-value cases, one query path may be enough. For higher-value cases, the
shop can query multiple nodes, store signed answers, and wait for stronger
confidence before granting value.

The important thing is that read verification is not locked behind the issuer's
API.

Users, merchants, issuers, auditors, intermediaries, and validators can all
check current and recent historical state directly from the EC network. But only
if they hold the document, and that matters. EC nodes never see the document
contents. There is no global document store to leak, steal from, or scrape.

## Redemption

When the shop accepts the voucher, it can consume the 30-unit token or transfer
it back to the issuer for settlement.

For example:

```text
destroy shop_token
```

or:

```text
transfer shop_token, owner = issuer_public_key
```

The issuer then settles according to its own business rules.

EC does not make the voucher valuable. The issuer does. EC makes the voucher
independently transferable and verifiable before redemption.

A shop does not need to trust the customer's wallet. It does not need to trust
another shop. It does not need direct access to the mall's live database.

It needs the issuer key, the document, and EC queries.

## The Same Shape Works For Payment Claims

A payment claim can use the same pattern.

A bank, fintech, marketplace, local currency system, or prepaid issuer signs a
claim document. A user holds it. A merchant receives it. The merchant checks EC
before granting value. The issuer settles when the claim is redeemed.

This is not "trustless money."

It is issuer-backed value with portable document custody.

The issuer remains the trust anchor. The merchant still cares whether the issuer
is good for settlement. Legal and commercial relationships still matter.

But the shared state register no longer has to be governed by one bank, one
platform, one card network, or one merchant database.

The issuer signs meaning. The user carries custody. EC tracks lifecycle. The
merchant verifies before acting.

If the user signs two incompatible successors, the result is not two clean
payments. The conflict becomes visible at verification or redemption.

The attack does not create money. It creates evidence. Since the issuer will
eventually check the document chain and redeem only valid claims, intermediaries
have the same incentive: verify before accepting, or risk holding worthless
claims.

## What Infrastructure Is Actually Needed?

An EC-based document application would still need normal software.

The issuer needs:

- document generation
- issuer signing keys
- schemas or document-type definitions
- optional transformer endpoints
- redemption policy
- optional EC write access for issuance and settlement

The user wallet needs:

- document storage
- key management
- document transfer UI
- hashing and signing
- EC query access
- optional EC write access for transfers or splits

The merchant or verifier needs:

- document parser
- schema or transformer support
- issuer-key trust rules
- EC query access
- acceptance policy
- optional evidence storage
- EC write access for redemption

Gateways and infrastructure providers can offer:

- public query endpoints
- write-access nodes
- stronger verification services
- cached signed answers
- monitoring for high-value tokens

But none of these actors need to host the canonical voucher database.

The document is carried by the parties. EC provides the shared lifecycle check.

## Why This Matters

Today, shared lifecycle usually means one of a few things.

Trust the issuer's live database. Integrate with a dominant platform. Use a bank
or card network. Publish state to a global ledger. Reconcile later and accept
fraud windows.

EC proposes a different split:

- private documents carry semantics
- users carry custody
- EC carries public lifecycle
- issuers and redeemers keep their normal systems

That is the interesting part.

A voucher can move between parties without every shop sharing a backend.

A payment claim can be verified before redemption without every merchant
depending on the issuer's live API for every step.

A ticket can be held by the user, transferred as a document, and consumed at
entry.

A credential can be presented from a wallet while EC answers whether it is still
live, revoked, or superseded.

EC does not replace existing institutions. It changes who has to govern the
shared state register.

## Honest Limits

This does not solve everything.

EC does not automatically provide issuer honesty, legal enforcement, privacy,
key recovery, offline finality, good redemption policy, or good document design.

If a document leaks private data, EC does not make it private. If an issuer
refuses to redeem, EC does not make them honest. If a user loses their key, the
application needs a recovery policy. If a high-value claim lives for a long time,
it may need monitoring and stronger verification.

But those are application design problems around a smaller primitive.

Instead of asking every participant to join the same platform, you ask them to
agree on document formats, issuer keys, and EC verification rules.

## The Part Worth Exploring

If EC worked at global scale, many multi-party applications would not need a
shared application database for document lifecycle.

They would need shared document rules and a public lifecycle check.

That is the part worth exploring: not a new asset class, not a platform that
owns the register, not a world where every business object is uploaded into
someone else's infrastructure.

Just signed documents, hashes, ordinary transport, and an open network that
answers one question:

> Is the thing I hold still current?

Which workflows already move documents but depend on someone else's register?

What could be built if users could safely carry the state themselves?

Where does this model fail?

The EC Protocol repo, simulator reports, and design documents are at [EcProtocol/EcNode](https://github.com/EcProtocol/EcNode). The
README is the best starting point.
