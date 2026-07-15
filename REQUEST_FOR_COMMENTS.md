# Request for comments: BlobMail

BlobMail is a proposed private encrypted messaging system using shared Ethereum blob batches as the delivery/availability layer.

The motivating product is a browser-extension mail/chat surface for X/Twitter users. Alice selects Bob's X profile, resolves Bob's messaging public key, encrypts locally, submits a padded encrypted message into a shared pending layer, and a blob-aware bundler batches many such messages into one Ethereum blob. Bob's client scans blob batches and decrypts entries intended for him.

## What is being proposed

The architecture combines:

- recipient-key encryption, likely HPKE;
- public or semi-public identity-to-messaging-key registry;
- optional `.gwei` naming;
- padded message size classes;
- encrypted chunking for large messages;
- optional sender-side decryptability for sent-message audit/history;
- ERC-4337 UserOperations for payment/authorization;
- paymasters for sponsorship;
- optional Semaphore-style anonymous sponsorship;
- Paraclete-style browser gossip for pending operations;
- blob-aware bundlers/batchers;
- testnet-first settlement.

## Main architectural bet

The system should avoid a central message queue without pretending Ethereum contracts can batch messages by themselves.

The likely shape is:

```text
browser extension
  -> AA / Paraclete-style pending layer
  -> blob-aware bundler
  -> Ethereum blob transaction
  -> receiver scans and decrypts
```

## Specific feedback requested

### Registry

For v1, public membership is acceptable.

Current leading option:

```text
identity_hash -> messaging_public_key
```

Questions:

- Should the first registry be a simple public on-chain mapping?
- Should identity hash use X account ID, normalized handle, `.gwei`, or another identifier?
- How should the design preserve a path to salted commitments later?
- Is there existing registry work this should reuse?

### AA / Paraclete

Questions:

- Should encrypted payloads be embedded directly in UserOperations?
- Or should UserOperations carry commitments while encrypted payloads move as adjacent gossip objects?
- Would Paraclete support a BlobMail-specific topic/object type?
- What constraints would current ERC-4337 bundlers impose?

### Batching and settlement

The v1 testnet settlement path may use a manifest-based proof:

```text
entry_hash -> manifest_root -> batcher payment
```

Known weakness:

This proves inclusion in the manifest, not necessarily that the actual blob bytes at the claimed position contain the entry.

Production target:

```text
payment authorization
  -> entry_hash
  -> exact blob position or polynomial opening
  -> blob commitment / versioned blob hash
  -> batcher payment
```

Questions:

- What is the cheapest practical proof path for tying an entry to actual blob data?
- Can this be made compatible with ERC-4337 paymaster settlement?
- Should v1 include a challenge window, or only use client-side auditing while the proof model is hardened?

### Privacy defaults

Questions:

- Should private mode require relay-only networking by default?
- How should browser peers avoid direct IP exposure?
- Is two-hop relay practical in a browser extension context?
- How should sender privacy be presented without overclaiming anonymity?

## Current docs

- [Master architecture](./docs/master-architecture.md)
- [Privacy technology options](./docs/privacy-tech.md)
- [Posting client](./docs/posting-client.md)
- [Receiving client](./docs/receiving-client.md)
- [AA mempool and Paraclete layer](./docs/aa-mempool-paraclete.md)
- [Blob batching and settlement](./docs/blob-batching-settlement.md)
- [RemiNet OAuth and identity](./docs/reminet-oauth-identity.md)
- [Frontend privacy/cost modes](./docs/frontend-modes.md)
- [Open questions](./docs/open-questions.md)

