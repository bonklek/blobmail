# BlobMail

BlobMail is an RFC-stage private messaging architecture for sending recipient-private messages through shared Ethereum blob batches.

The initial client target is a browser extension integrated with [milXdy](https://github.com/bonklek/milXdy) on X/Twitter. The protocol explores recipient-key encryption, padded and chunked payloads, ERC-4337 UserOperations, paymaster sponsorship, Paraclete-style gossip collection, and blob-aware bundlers.

This repository is currently an architecture and design workspace. It is not a production implementation.

## Why this exists

BlobMail is trying to answer a specific design question:

> Can a browser-extension messaging app use Ethereum blobs as a shared encrypted mail availability layer, while preserving useful recipient privacy, optional sender privacy, and permissionless batching economics?

The current design assumes:

- recipient-hiding encryption is always on;
- users publish or resolve messaging public keys through a registry;
- messages are padded into common public size classes;
- large messages are encrypted, chunked, and optionally dispersed;
- pending messages can propagate through an AA/Paraclete-style mempool;
- ERC-4337 bundlers can be extended into blob-aware batchers;
- settlement starts on testnet with a weaker manifest-based proof, then moves toward production-grade blob-content linkage.

## Start here

- [Request for comments](./REQUEST_FOR_COMMENTS.md)
- [Master architecture](./docs/master-architecture.md)
- [Privacy technology options](./docs/privacy-tech.md)
- [Posting client](./docs/posting-client.md)
- [Receiving client](./docs/receiving-client.md)
- [AA mempool and Paraclete layer](./docs/aa-mempool-paraclete.md)
- [Blob batching and settlement](./docs/blob-batching-settlement.md)
- [RemiNet OAuth and identity](./docs/reminet-oauth-identity.md)
- [Frontend privacy/cost modes](./docs/frontend-modes.md)
- [Open questions](./docs/open-questions.md)

## Feedback requested

Feedback is especially useful on:

- public-key registry design;
- `.gwei` / on-chain identity-key mapping;
- committed or salted registry migration paths;
- ERC-4337 UserOperation compatibility;
- Paraclete integration;
- paymaster and anonymous sponsorship design;
- blob batch construction;
- inclusion proof and settlement design;
- receiver-side blob scanning;
- IP privacy defaults for browser gossip.

## Status

RFC / architecture draft.

The first implementation target should be a testnet or local devnet flow, not mainnet production settlement.
