# MeshCore Trust Layer Protocol Specification v0.1

## Overview

MeshCore Trust Layer is a decentralized security layer for MeshCore networks that provides:

- Self-sovereign identity using Ed25519 keys
- Web-of-Trust based public key verification
- Forward-secret session establishment using X25519
- End-to-end encrypted messaging using AES-256-GCM

This document defines the minimum viable protocol required for interoperable implementations.

---

## Design Goals

- No central authority
- No certificate infrastructure
- Secure identity verification via social trust
- Forward secrecy for all sessions
- Lightweight operation for mesh environments
- Human-verifiable identity fingerprints

---

## Cryptographic Primitives

| Purpose           | Algorithm     |
|------------------|--------------|
| Identity signing  | Ed25519      |
| Key exchange      | X25519       |
| Key derivation    | HKDF-SHA256  |
| Symmetric crypto  | AES-256-GCM  |

---

## Identity Model

Each node has a permanent identity key pair:

- Ed25519 Private Key
- Ed25519 Public Key

The public key acts as the Node ID.

### Node Identity Structure

```json
{
  "node_id": "alice",
  "public_key": "<ed25519_public_key>",
  "fingerprint": "<sha256(public_key)>"
}
````

### Fingerprint Display

Fingerprints MUST be human-verifiable:

```
SHA256(public key) → hex string
```

Used for offline identity verification.

---

## Trust Model (Web of Trust)

Nodes may sign other nodes' identity public keys.

### Trust Signature Format

```json
{
  "issuer": "alice",
  "subject": "bob",
  "subject_public_key": "<ed25519_public_key>",
  "signature": "<ed25519_signature>"
}
```

### Trust Rule (v0.1)

A node is considered trusted if:

* It is directly signed by a trusted node, OR
* It has at least one valid trust path from a trusted node

No scoring system is required in v0.1.

---

## Session Establishment

MeshCore Trust Layer uses ephemeral X25519 keys for each session.

### Step 1: Alice initiates session

Alice generates an ephemeral X25519 key pair:

* AliceEphemeralPrivate
* AliceEphemeralPublic

Alice signs the ephemeral public key with her identity key:

```
Signature = Sign(AliceIdentityPrivate, AliceEphemeralPublic)
```

Alice sends:

```json
{
  "type": "session_init",
  "identity": "alice",
  "ephemeral_public": "<alice_ephemeral_public>",
  "signature": "<ed25519_signature>"
}
```

---

### Step 2: Bob responds

Bob verifies Alice’s identity and signature.

Bob generates his own ephemeral key pair:

* BobEphemeralPrivate
* BobEphemeralPublic

Bob signs his ephemeral public key:

```
Signature = Sign(BobIdentityPrivate, BobEphemeralPublic)
```

Bob sends:

```json
{
  "type": "session_reply",
  "identity": "bob",
  "ephemeral_public": "<bob_ephemeral_public>",
  "signature": "<ed25519_signature>"
}
```

---

## Shared Secret Derivation

Both parties compute:

```
SharedSecret =
X25519(
    own_ephemeral_private,
    other_party_ephemeral_public
)
```

This value is never transmitted.

---

## Session Key Derivation

The shared secret is processed using HKDF-SHA256:

```
SessionKey = HKDF(SharedSecret, context = "MeshCoreTrustLayer-v0.1")
```

Result:

* AES-256-GCM key (32 bytes)

---

## Message Encryption

All messages use AES-256-GCM.

### Message Format

```json
{
  "type": "message",
  "nonce": "<12_byte_nonce>",
  "ciphertext": "<encrypted_data>",
  "tag": "<auth_tag>"
}
```

### Encryption Rule

Each message MUST use a unique nonce per session.

---

## Message Flow

```
Alice → session_init → Bob
Bob   → session_reply → Alice
Alice ↔ encrypted messages ↔ Bob
```

---

## Security Properties

MeshCore Trust Layer v0.1 provides:

* Confidentiality (AES-256-GCM)
* Integrity (AEAD authentication)
* Authentication (Ed25519 signatures)
* Forward secrecy (X25519 ephemeral keys)
* Decentralized trust (Web of Trust)

---

## Threat Model (v0.1)

### Protected against:

* Passive network eavesdropping
* Message tampering
* Replay attacks (if nonce tracking is implemented)
* Key substitution attacks (with valid trust graph)

### Not protected against:

* Traffic analysis
* Compromised endpoints
* User mis-trust decisions
* Trust graph manipulation in edge cases

---

## Trust Limitations

Web of Trust is inherently subjective.

Each node decides:

* Which signers it trusts
* How many trust paths are required
* Whether to accept unknown signatures

There is no global authority or global truth.

---

## Versioning

```
MeshCoreTrustLayer-protocol: v0.1
```

All implementations MUST reject incompatible protocol versions unless explicitly configured.

---

## Future Extensions (not in v0.1)

* Trust scoring system
* Key revocation
* Key expiration
* Group messaging
* Post-quantum cryptography
* Mesh routing integration
* Identity rotation

---

## Security Warning

This protocol is experimental and has not been audited.

Do not use it for high-risk or production security applications without independent cryptographic review.
