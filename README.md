# IOA

**IOA** (Internet of Agents) is a **W3C DID method [`did:ioa`](https://www.w3.org/TR/did-core/)** for decentralized identity on the agent internet. It supports agent **registration**, **discovery**, and **interconnection** on blockchain networks through on-chain DID documents.

| Goal | Summary |
|------|---------|
| **Registration** | Agents, organizations, and registration nodes obtain a unique `did:ioa` and register a DID document on chain. |
| **Discovery** | Resolvers read capability descriptions and service endpoints from DID documents (including sidechain resolution). |
| **Interconnection** | Peers connect via `serviceEndpoint` and protocols such as **A2A**. |
| **Governance & security** | Key control, recovery, deactivation, and privacy minimization (see spec §5–§6). |

**DID documents** express who the subject is, what it can do, and how to connect (normative fields in [DID Core](https://www.w3.org/TR/did-core/) plus IOA `extension`). **Verifiable credentials (VCs)** are separate issuer assertions under the [W3C VC Data Model](https://www.w3.org/TR/vc-data-model/); they complement DID documents but are not required to mirror them field-for-field.

Normative DID documents use top-level **`@context`**, **`verificationMethod`**, **`authentication`**, and **`service`** (e.g. `AgentDescription`, `DIDSubResolver`), with IOA-specific metadata in **`extension`**. Standard resolution is the §5.2 **Read** JSON POST API on a registration node.

## Documentation

Version **1.0.0** — authoritative text in this repository:

| Language | Specification |
|----------|----------------|
| 中文 | [doc/cn/IOA身份标识协议规范.md](doc/cn/IOA身份标识协议规范.md) |
| English | [doc/en/IOA Protocol Specification.md](doc/en/IOA%20Protocol%20Specification.md) |

Golden examples and field rules are defined in the specifications §4.3; chain **Create** should also set `extension.ttl` and optionally `extension.type`: `206` where applicable.
