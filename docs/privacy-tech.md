# Privacy technology options

BlobMail should expose privacy choices to users, but only where those choices do not meaningfully degrade the shared private pool.

## Related docs

- [Master architecture](./master-architecture.md)
- [Posting client](./posting-client.md)
- [Receiving client](./receiving-client.md)
- [Frontend privacy/cost modes](./frontend-modes.md)
- [RemiNet OAuth and identity](./reminet-oauth-identity.md)

## Always-on private-payload baseline

Every payload in the private lane should use:

- recipient-key encryption;
- fresh sender-side ephemeral encryption key per payload or chunk;
- no public recipient identifier in the entry;
- no Bob public key in the entry;
- authenticated encryption;
- standard public size classes;
- local recipient scanning;
- relay-only networking by default.

There should not be a cheaper "private but unencrypted" mode. Public messages or public media belong in a separate lane.

## Recipient privacy

Recipient privacy uses Bob's published messaging public key, not Bob's Ethereum address.

Alice encrypts with an ephemeral key:

```text
Alice ephemeral secret r
Alice ephemeral public key R
Bob messaging public key pk_B
shared secret S = r * pk_B
```

Bob later computes:

```text
S = sk_B * R
```

The public entry contains:

```text
ephemeral_public_key
view_tag
padded_ciphertext
auth_tag
commitment
```

The public entry does not contain:

```text
@bob
bob.gwei
Bob's wallet address
Bob's messaging public key
plaintext content-type, filename, or thread metadata
```

HPKE is the preferred standards-based construction unless implementation constraints force a different audited scheme.[^hpke]

## Crypto agility and post-quantum lanes

Standard HPKE suites such as X25519-based HPKE are not post-quantum secure. BlobMail should preserve crypto agility so future versions can support hybrid or post-quantum KEMs, such as a classical KEM combined with ML-KEM/Kyber.

Post-quantum or hybrid encryption increases public cryptographic material size. If the public entry format exposes different key material lengths, observers may infer which encryption mode a sender used.

The preferred compromise is size/security lanes:

```text
small private lanes
  512 B / 2 KB
  classical HPKE only
  optimized for cheap small messages

larger private lanes
  8 KB / 32 KB
  fixed crypto envelope large enough for hybrid PQ
  HPKE entries pad unused crypto envelope space
  HPKE and hybrid-PQ entries can look identical within the lane
```

This keeps tiny messages cheap while allowing larger privacy/security-capable lanes to hide whether a sender used classical HPKE or hybrid post-quantum encryption.

The tradeoff is that a tiny message using hybrid PQ must move into a larger lane. Observers may see that the sender chose a larger class, but that larger class can also contain:

- ordinary larger text;
- media;
- file attachments;
- obfuscated small messages;
- dummy chunks;
- large-message fragments.

So the public signal should be:

```text
this entry used a larger privacy/security-capable lane
```

not:

```text
this entry definitely used post-quantum encryption
```

## Sender privacy

Standard ERC-4337 exposes the `sender` smart-account address. To avoid linking messages to Alice's known wallet, use ephemeral smart accounts.[^erc4337]

Possible modes:

| Mode | Sender account | Funding | Privacy | Cost |
|---|---|---|---|---|
| Basic | normal account | direct or ordinary paymaster | sender public | lowest |
| Ephemeral session | temp account per session | ordinary paymaster | main wallet hidden, session linkable | low |
| Anonymous session | temp account per session | Semaphore/paymaster proof | main identity hidden from chain | medium/high |
| Shielded balance | temp account | private prepaid pool | strongest funding unlinkability | highest |

The cheapest reasonable sender-privacy default is a temporary smart account per session, sponsored by a shared paymaster.

Future account-abstraction ergonomics may also be affected by EIP-7702-style delegated account behavior, but the current design should not depend on it.[^eip7702]

## Anonymous sponsorship

The ephemeral account should not receive ETH from Alice's known wallet.

Options:

1. Ordinary paymaster
   - simple;
   - cheap;
   - paymaster may know Alice.

2. Semaphore-based proof
   - proves Alice belongs to an authorized group;
   - hides which member Alice is;
   - uses nullifiers to prevent double-use;
   - proof verification is expensive enough that it should usually authorize a session, not a single tiny message.

3. Shielded prepaid balance
   - Alice deposits funds into a private pool;
   - later proves privately that she can spend a credit;
   - strongest funding unlinkability;
   - largest implementation and audit burden.

Semaphore is relevant to anonymous sponsorship, while Privacy Pools-style designs are relevant to stronger shielded funding paths.[^semaphore][^privacy-pools]

Recommended progression:

```text
ordinary paymaster
  -> ephemeral session accounts
  -> Semaphore session sponsorship
  -> shielded balance if needed
```

## IP privacy

Default private messaging should not permit direct WebRTC exposure between users.

Minimum default:

```text
extension -> relay -> gossip network
```

Stronger option:

```text
extension -> entry relay -> second relay -> gossip network
```

Rules:

- disable direct peer dialing in private mode;
- rotate libp2p peer identities;
- keep messaging peer identity separate from AA/generic identity if possible;
- never bind peer ID to X handle, wallet, or `.gwei` name;
- consider bounded forwarding delays;
- support Tor/VPN/other anonymity layers as advanced options.

Relay-only mode hides Alice's IP from ordinary peers. It does not hide Alice's IP from the entry relay.

## Timing and size privacy

Use public size classes:

- 512 B
- 2 KB
- 8 KB
- 32 KB

Exact public entry lengths should not be user-selectable because rare sizes create fingerprints.

Users may choose variable logical chunking inside the encrypted payload:

```text
variable logical chunks -> standard padded public entries
```

This lets Alice spend more for obfuscation without weakening the shared pool.

## Rich content and large-object privacy

BlobMail payloads should be arbitrary encrypted bytes. Text is only one content type. The same envelope should support:

- text;
- images;
- GIFs;
- audio;
- video;
- documents;
- arbitrary file attachments;
- mixed multi-part content;
- compressed archives;
- application-specific binary objects.

Large payloads are split into encrypted chunks. Publicly, each chunk should look like an independent padded private entry.

Encrypted metadata tells Bob:

- stream/message ID;
- private content type or MIME type;
- optional filename or display label;
- inline versus attachment disposition;
- codec/container information, if needed;
- dimensions or duration, if needed;
- multi-part manifest, if needed;
- chunk index;
- total chunks or reconstruction policy;
- hash/commitment;
- whether dummy chunks exist;
- erasure coding parameters;
- reply/thread context.

Public entries should not share a visible message ID.

A private video message is therefore just a large encrypted byte stream with encrypted media metadata. Alice can chunk it into 32 KB-class entries, optionally add redundancy or dummy chunks, and disperse those chunks across batches. Bob only learns the media type, duration, codec, and reconstruction metadata after decrypting enough chunks.

## References

[^hpke]: [RFC 9180: Hybrid Public Key Encryption](https://datatracker.ietf.org/doc/rfc9180/), the main standards reference for HPKE.
[^erc5564]: [ERC-5564: Stealth Addresses](https://eips.ethereum.org/EIPS/eip-5564), relevant to recipient scanning, stealth addressing, and view-tag-like recipient discovery patterns.
[^erc6538]: [ERC-6538: Stealth Meta-Address Registry](https://eips.ethereum.org/EIPS/eip-6538), relevant to public-key/meta-address publication.
[^erc4337]: [ERC-4337: Account Abstraction Using Alt Mempool](https://eips.ethereum.org/EIPS/eip-4337), relevant to ephemeral smart accounts and paymasters.
[^eip7702]: [EIP-7702: Set EOA account code](https://eips.ethereum.org/EIPS/eip-7702), relevant account-abstraction-adjacent work for delegated account behavior.
[^semaphore]: [Semaphore documentation](https://docs.semaphore.pse.dev/), relevant to anonymous group membership proofs and nullifiers.
[^privacy-pools]: [Privacy Pools paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4563364), relevant to shielded funds with association-set proofs.

Optional distribution modes:

- Fast: chunks near each other; low latency; easier traffic analysis.
- Mixed: chunks spread across normal batches.
- Dispersed: randomized release, dummy chunks, redundancy, multiple relay paths; higher cost/latency.
