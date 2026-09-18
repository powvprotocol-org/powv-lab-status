# PoWV Virtual Lab — Technical Status

A local reference implementation and testbed for a compact signed-event pipeline designed for constrained, auditable event ingestion.

The lab focuses on preserving provenance between an originating measurement, its cryptographic attestation, and the audit representations derived from it.

This repository is a research and engineering prototype focused on demonstrating how measurements or observations associated with physical and operational events can be represented as minimal signed evidence, validated at the edge, protected against replay, and anchored through a Merkle-based audit trail.

> **Important**
>
> This project is a local proof-of-concept and laboratory environment. It is not a production deployment, not a distributed blockchain network, and not a hardened operational system for live critical infrastructure.

## Executive summary

The PoWV Virtual Lab demonstrates a controlled local pipeline for turning an originating measurement or observation into auditable cryptographic evidence:

```text
Measurement or observation
        ↓
Compact serialized representation
        ↓
ECDSA P-256 cryptographic attestation
        ↓
Edge validation
        ↓
Replay protection
        ↓
Event hash
        ↓
Merkle-root audit anchoring
        ↓
Signed local evidence
```

The system attests to the bytes signed by a provisioned device key. It does **not** independently prove that the physical world, sensor, operator, or measurement source is truthful.

```text
Verify(σ, M) = true  does not imply  M = E
```

It means that the holder of the corresponding private key signed the represented bytes. Physical-source trust requires additional controls outside this local prototype, such as calibrated sensors, secure elements, trusted execution, device lifecycle management, and operational governance.

## Scope and terminology

| Symbol | Meaning |
| --- | --- |
| `E` | Physical or operational event being observed. |
| `M` | Measurement or observation associated with `E`. |
| `P` | Serialized packet carrying the compact evidence. |
| `σ` | Cryptographic signature over the signed packet bytes. |
| `V` | Structural, integrity, authenticity, and replay validation. |
| `H` | Cryptographic hash used to identify accepted evidence. |
| `A` | Audit anchoring, including Merkle-root calculation and evidence persistence. |
| `I` | Later interpretation, classification, or business decision. |

The implemented binary path can be represented as:

```text
E → M → P_unsigned → σ → P → V → H → A → I
```

The distinction between these stages is intentional:

- **attestation** establishes that a key signed specific bytes;
- **validation** determines whether those bytes satisfy the laboratory's acceptance rules;
- **provenance** preserves the relationship between the originating representation and its derived audit artifacts;
- **anchoring** records a digest or Merkle root in an application-managed audit trail;
- **interpretation** assigns domain meaning and is not implemented as a complete decision engine here.

## What is implemented

### 1. Compact binary event format

The encoding and decoding logic lives in [`powv_compact_event.py`](https://github.com/GabsDevX/PoWV_Virtual_Lab/blob/main/powv_compact_event.py).

The current PoWV-SBD v1 packet is fixed at **132 bytes**:

| Field | Size | Description |
| --- | ---: | --- |
| `version` | 1 byte | Protocol version. |
| `device_id` | 8 bytes | Unsigned 64-bit device identifier. |
| `event_id` | 8 bytes | Unsigned 64-bit event identifier. |
| `timestamp` | 4 bytes | Unix timestamp in seconds. |
| `geo` | 6 bytes | Quantized latitude and longitude. |
| `asset_id` | 8 bytes | Unsigned 64-bit asset identifier. |
| `status` | 1 byte | Event status. |
| `measurement_hash` | 32 bytes | SHA-256 digest of the measurement representation. |
| `signature` | 64 bytes | Raw ECDSA `r || s`, 32 bytes per component. |
| **Total** | **132 bytes** | 68-byte unsigned body plus 64-byte signature. |

The gateway verifies the signature over the original 68-byte unsigned body. It does not reconstruct the signed bytes from decoded fields, avoiding serialization discrepancies between producer and verifier.

### 2. Edge validation and replay protection

[`edge_gateway.py`](https://github.com/GabsDevX/PoWV_Virtual_Lab/blob/main/edge_gateway.py) provides the primary binary ingestion route, `POST /gateway_bin`.

It currently performs:

- exact packet-size validation;
- packet decoding;
- public-key lookup by `device_id`;
- ECDSA P-256 verification over the original unsigned bytes;
- duplicate detection using `(device_id, event_id)` in SQLite;
- forwarding of the minimal event evidence to the local audit service.

For the binary path, the event hash forwarded downstream is calculated as:

```text
H = SHA-256(P_unsigned)
```

The current gateway does not independently recompute `measurement_hash` from the underlying physical measurement, because the raw measurement is not present in the compact packet. `measurement_hash` is carried as a signed field and should therefore be described as an attested measurement digest, not as an independently verified measurement claim.

Expected binary-ingestion outcomes include:

| Condition | Result |
| --- | --- |
| Valid packet | HTTP `200` |
| Malformed or incorrectly sized packet | HTTP `400` |
| Unknown device | HTTP `401` |
| Invalid signature | HTTP `400` |
| Repeated `(device_id, event_id)` | HTTP `409` |

### 3. Local audit-chain service

[`blockchain_audit.py`](https://github.com/GabsDevX/PoWV_Virtual_Lab/blob/main/blockchain_audit.py) acts as a local audit-chain service. The filename is retained for compatibility, but the implementation should not be described as a distributed blockchain.

It provides:

- local block creation;
- in-memory event-chain state;
- Merkle-root calculation;
- device public-key registration;
- block inspection endpoints;
- Server-Sent Events (SSE) streaming through `/stream`.

The current chain is **in memory**. Restarting the service loses the chain unless another local process has already exported or anchored the relevant evidence.

### 4. Anchoring and evidence generation

[`anchor_service.py`](https://github.com/GabsDevX/PoWV_Virtual_Lab/blob/main/anchor_service.py) periodically reads the current audit state and creates local evidence artifacts:

- `immutable_anchor_ledger.log`: historical filename for an application-managed append-only audit ledger;
- `latest_anchor_evidence.json`: latest signed evidence document;
- locally generated audit signing key material used by the prototype.

The append-only behavior is enforced by the application during normal operation. A local file is not immutable against an administrator or process with filesystem access. These artifacts do not constitute publication to a public blockchain, legal certification, or tamper-proof external storage.

### 5. Simulated surrounding services

| Component | File | Role | Status |
| --- | --- | --- | --- |
| Satellite link | `satellite_link.py` | Simulates latency, loss, and forwarding. | Simulation |
| Geolocation | `geolocation_service.py` | External lookup with local fallback. | Local service/simulation |
| Hardware oracle | `hardware_oracle.py` | Placeholder signing interface. | Stub; not real hardware |
| Event feeder | `event_feeder.py` | Produces synthetic JSON or binary events. | Test utility |

## Repository layout

The implementation repository is [`GabsDevX/PoWV_Virtual_Lab`](https://github.com/GabsDevX/PoWV_Virtual_Lab):

```text
powv_compact_event.py       compact packet format and signature helpers
edge_gateway.py             edge validation and replay protection
blockchain_audit.py         local audit-chain API and Merkle roots
anchor_service.py           local evidence anchoring and signing
event_feeder.py             synthetic event generator
satellite_link.py           simulated upstream satellite path
geolocation_service.py      geolocation lookup/fallback service
hardware_oracle.py          placeholder oracle interface
tests/                      automated hardening tests
tools/                      provisioning, integration, and reporting utilities
logs/                       runtime artifacts; not a source of truth
```

## Validated capabilities

The local project includes tests and tools covering:

- fixed-size packet encoding;
- rejection of trailing bytes;
- ECDSA P-256 signing and verification;
- rejection of tampered unsigned data;
- rejection of corrupted signatures;
- device provisioning with an administrative token;
- replay protection through a uniqueness constraint;
- Merkle-root exposure through the audit API;
- generation and independent verification of signed evidence;
- local SSE event streaming.

These validations demonstrate behavior in the current test environment. They are not equivalent to an external security audit, formal verification, certification, or production acceptance test.

## Local quick start

Clone the implementation repository and create a Python environment:

```powershell
git clone https://github.com/GabsDevX/PoWV_Virtual_Lab.git
Set-Location PoWV_Virtual_Lab
py -3.12 -m venv .venv
& .\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 1. Run the hardening tests

```powershell
python -m pytest -q tests/test_hardening.py -p no:cacheprovider
```

### 2. Start the local services

Start the audit-chain service and gateway in separate terminals:

```powershell
python blockchain_audit.py
python edge_gateway.py
```

Optional simulated services:

```powershell
python geolocation_service.py
python satellite_link.py
python hardware_oracle.py
python anchor_service.py
```

### 3. Create and register a test device

Set a local administrative token **before starting** `blockchain_audit.py`:

```powershell
$env:POWV_ADMIN_TOKEN = "local-dev-token"
```

Generate a local test key pair:

```powershell
python tools/device_keygen.py `
  --device-name sim-0 `
  --private-out device_priv.pem `
  --public-out device_pub.pem
```

Use the device ID printed by `device_keygen.py` to register the public key:

```powershell
python tools/register_device.py `
  --device-id <DEVICE_ID_PRINTED_BY_KEYGEN> `
  --public device_pub.pem `
  --url http://127.0.0.1:5003/register_device
```

Do not commit `device_priv.pem`, `device_pub.pem`, tokens, logs, or generated packet files.

### 4. Send signed test traffic

```powershell
python event_feeder.py `
  --mode binary `
  --count 1 `
  --private device_priv.pem `
  --gateway http://127.0.0.1:5002/gateway_bin
```

The exact command and additional integration utilities are documented in the implementation repository's [`README.md`](https://github.com/GabsDevX/PoWV_Virtual_Lab/blob/main/README.md).

## Security posture and trust boundaries

This repository demonstrates **selected security controls** for a local concept prototype:

- signature verification before binary event acceptance;
- rejection of malformed and tampered packets;
- duplicate detection for `(device_id, event_id)`;
- public-key registration protected by an administrative token;
- application-managed append-only audit logging;
- explicit packet-size and format checks.

These controls establish integrity and provenance properties within the laboratory assumptions; they do not establish the trustworthiness of the physical measurement source itself.

The implementation still requires substantial hardening before production use. In particular, it does not currently provide all of the following:

- secure-element, HSM, or TEE-backed signing;
- production key generation, storage, rotation, revocation, and recovery;
- TLS or mutual TLS between services;
- a durable, transactional, recoverable audit-chain store;
- distributed availability and consistency guarantees;
- comprehensive schema validation and policy enforcement;
- complete rate limiting and abuse protection;
- production-grade secret management;
- operational monitoring, alerting, backup, and disaster recovery;
- field validation with real satellite, RS-485, or industrial hardware.

Do not use generated private keys, local tokens, debug logs, or simulated signatures in a production environment.

## Production-readiness assessment

| Area | Current assessment |
| --- | --- |
| Compact binary format | Implemented and locally tested |
| ECDSA P-256 verification | Implemented and locally tested |
| Tamper rejection | Demonstrated in local tests |
| Replay protection | Implemented with local SQLite uniqueness |
| Merkle-root audit | Implemented locally |
| Signed evidence | Implemented locally |
| Durable audit persistence | Not production-ready |
| Hardware-backed trust | Not implemented in the lab |
| Network security | Requires TLS/mTLS and service authentication |
| Distributed operation | Not implemented |
| Industrial deployment | Not validated |
| Certification or compliance | Not claimed |

### Honest classification

The correct current classification is:

> **Functional local proof of concept and technical reference implementation for compact signed-event validation, provenance preservation, and audit anchoring.**

It is not accurate to classify the lab as a production-ready platform, a public blockchain, or a deployed cyber-physical infrastructure.

## Roadmap

Recommended next phases are:

1. replace the placeholder oracle with real hardware-backed signing;
2. define and implement a durable audit-chain storage model;
3. add authenticated and encrypted service-to-service communication;
4. formalize packet, API, error, and key-registry schemas;
5. define key rotation, revocation, recovery, and device lifecycle procedures;
6. add rate limiting, observability, alerting, backup, and disaster recovery;
7. validate realistic transport behavior, including satellite and RS-485 scenarios;
8. perform independent security review and threat-model validation;
9. separate laboratory reference code from production deployment code;
10. define explicit acceptance criteria for any future field deployment.

## Contribution guidelines

Contributions are welcome when they improve:

- reproducibility of the local setup;
- test coverage and failure handling;
- protocol and API clarity;
- security boundaries and threat modeling;
- documentation accuracy;
- separation between simulated and production-intended components.

Please label experimental, simulated, and production-intended behavior explicitly. Avoid describing local test results as operational guarantees.

## License and usage

No license has been declared in the implementation repository at the time of writing. No production or commercial-use rights are implied by this prototype repository unless explicitly stated in an applicable repository license or other written authorization.

## Source of truth

This repository documents status and maturity. The implementation source of truth is [`GabsDevX/PoWV_Virtual_Lab`](https://github.com/GabsDevX/PoWV_Virtual_Lab). When this status document and the implementation diverge, treat the code and current test results as authoritative until the discrepancy is resolved and documented.
