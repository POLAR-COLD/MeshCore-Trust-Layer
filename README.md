# MeshCore Trust Layer

A decentralized Web-of-Trust security layer for MeshCore networks.

MeshCore Trust Layer enables nodes to establish authenticated, end-to-end encrypted communication without relying on centralized certificate authorities or servers. Identity is established through a Web of Trust model, while session security is provided through modern cryptographic primitives and forward-secret key exchange.

## Why?

Mesh networks are decentralized by design. Traditional Public Key Infrastructure (PKI) depends on centralized Certificate Authorities, which may not be available or desirable in disconnected or community-operated networks.

This project explores an alternative approach:

* Users own their identities.
* Users verify each other directly.
* Trust is distributed through signed public keys.
* Communication remains secure even if long-term identity keys are compromised in the future.

## Features

### Decentralized Identity

Each node generates a permanent Ed25519 identity key pair.

* No central authority
* No account registration
* No certificate authority

A public key becomes the node's cryptographic identity.

### Web of Trust

Users may verify and sign other users' public keys.

Trust can be established through:

* In-person fingerprint verification
* Existing trusted relationships
* Community trust networks

Signed keys propagate through the mesh, allowing nodes to evaluate trust paths.

### Forward Secrecy

Every communication session uses temporary X25519 key pairs.

Session keys are never transmitted directly.

If a node's long-term identity key is compromised in the future, previously recorded conversations remain protected.

### End-to-End Encryption

Messages are encrypted using AES-256-GCM.

Provides:

* Confidentiality
* Integrity verification
* Authenticated encryption

## Protocol Overview

### Identity Creation

Each node generates:

```text
Ed25519 Private Key
Ed25519 Public Key
```

The public key acts as the node's identity.

### Trust Establishment

Alice verifies Bob's fingerprint.

Alice signs Bob's public key:

```text
Alice → Bob
```

This signed trust relationship may be shared throughout the mesh.

### Session Establishment

Alice generates a temporary X25519 key pair.

Alice signs the ephemeral public key using her Ed25519 identity key.

Bob verifies the signature and responds with his own signed ephemeral key.

### Shared Secret Derivation

Both nodes derive the same shared secret using X25519.

```text
Alice Ephemeral Private Key
           +
Bob Ephemeral Public Key

           =

Shared Secret
```

No secret material is transmitted across the network.

### Session Encryption

The shared secret is processed through HKDF to derive an AES-256 session key.

Messages are encrypted using AES-GCM.

## Example Trust Graph

```text
You
 │
 └── Alice
       │
       └── Bob
              │
              └── Charlie
```

Nodes may choose their own trust policies.

Examples:

* Trust direct signatures only
* Trust signatures within two hops
* Require multiple independent trust paths

## Security Goals

* Decentralized identity verification
* Public key authentication
* Forward secrecy
* Message confidentiality
* Message integrity
* Resistance to man-in-the-middle attacks

## Non-Goals

* Anonymous communication
* Traffic analysis resistance
* Metadata protection
* Routing protocols
* Mesh transport implementation

This project focuses exclusively on cryptographic identity and secure communication.

## Cryptography

| Purpose            | Algorithm   |
| ------------------ | ----------- |
| Identity           | Ed25519     |
| Trust Signatures   | Ed25519     |
| Key Exchange       | X25519      |
| Key Derivation     | HKDF-SHA256 |
| Message Encryption | AES-256-GCM |

## Development Roadmap

### v0.1

* Identity generation
* Fingerprint generation
* Key signing
* Signature verification
* X25519 session establishment
* AES-GCM encrypted messaging

### v0.2

* Trust database
* Trust graph storage
* Trust path discovery
* Trust validation engine

### v0.3

* MeshCore integration
* Signed trust propagation
* Peer discovery integration

### v1.0

* Stable protocol specification
* Interoperability testing
* Security review
* Production-ready release

## Warning

This project is experimental.

Do not rely on it to protect sensitive information until the protocol and implementation have undergone extensive testing and independent security review.

## License

Licensed under the GNU General Pubic License 3.0
