# November 2026 Software Engineering Status Update

**Date:** November 20, 2026
**Scope:** Software engineering in the access-controlled PoWV Virtual Lab repository
**Status of the work described:** **Experimental.** Unmerged, unreleased and not production-qualified.

> This update is a public-safe, capability-level summary. It does not disclose:
>
> - source code;
> - event structures or serialization rules;
> - cryptographic profiles;
> - test vectors;
> - internal interfaces;
> - security findings;
> - operational configuration.
>
> Detailed engineering records remain restricted.

## 1. How to read this update

PoWV publishes evidence in separate, dated records. Each record has its own scope, and no record extends the conclusions of another.

| Record | Date | Scope | Where |
| --- | --- | --- | --- |
| Software validation snapshot | September 6, 2026 | 25 selected software test cases on a recorded source snapshot; no physical hardware | [Validation Evidence](https://github.com/powvprotocol-org/powv-governance-and-docs/blob/main/powvprotocol-org/powv-protocol-docs/powv-protocol-documentation/untitled/project-status/validation-evidence.md) |
| Physical integration record | September 18, 2026 | Controlled laboratory acquisition from a real weighing instrument, local transport and ESP32 acknowledgment | [Experimental Record](https://github.com/powvprotocol-org/powv-scale-edge-integration/blob/main/docs/integration-module/experimental-record.md) |
| **Software engineering update** | **November 20, 2026** | **Software-only security hardening, conformance testing, cross-language verification and contract governance** | This document |

The September records are unchanged by this update and remain the historical evidence for their own scopes.

## 2. Historical September validation (unchanged)

- **September 6, 2026:** a controlled local software audit.
  - 25 selected test cases passed.
  - 3 of 3 hardening runs were consistent.
  - 5 simulated tamper and malformed-input scenarios were rejected.
  - No physical hardware was used.
- **September 18, 2026:** a separate laboratory record.
  - It documents acquisition from a real weighing instrument, normalization into a structured event, integrity identification, local network transport, ESP32 acknowledgment and local inspection.
  - It does **not** establish authenticated device identity, hardware-backed signatures, durable audit anchoring or a production chain of custody.

Nothing in the November work re-tests or re-validates the September physical record.

## 3. New software security and conformance testing (November 2026)

Software-only engineering work was carried out in the access-controlled repository as **multiple unmerged experimental change sets**. **None of them is an approved release.**

At capability level, the work addresses:

- **Independent verification at the audit boundary.** The audit component verifies submitted evidence itself.
- **Stricter input handling.** Unsupported or malformed evidence is rejected before it is processed further.
- **Versioned audit integrity.** A new, additive integrity version strengthens tamper detection of the local audit history. Records created under the earlier version are preserved unchanged.
- **Evidence admissibility levels.** Accepted evidence is labelled by what was actually verified. Three properties are kept distinct:
  - **digital evidence admissibility**, which software verification can assess;
  - **physical measurement accuracy**, which it cannot;
  - **secure key custody**, which it cannot either.

  The labels state explicitly that the last two are **not** established.
- **Independent checkpoint verification tooling.** It compares a local audit history against a separately preserved checkpoint.
- **Bench-readiness procedures** for a future controlled physical run. These procedures were rehearsed in software only and were **not** executed on the physical bench.
- **Key-lifecycle procedures** were documented. **No key operation was performed.**

**Verified results** (completed runs only):

| Evidence | Result |
| --- | --- |
| Automated test suite on each change set | Passed in continuous integration on Python 3.12 and 3.13 |
| Test suite at the latest change set | 343 test cases passed, 0 skipped, on Python 3.12 and 3.13 (continuous integration) |
| Physical-hardware tests | **0** (none performed) |

## 4. Cross-language verification

A versioned conformance suite was created for one existing protocol serialization rule. **The rule itself was not changed.**

- The suite is checked against the reference implementation and against **two independent implementations** written in different programming languages.
- Published conformance material is designed to be immutable. New material is added only as new versions.

**Verified results** (completed runs in a controlled software environment):

- The conformance suite passed in both independent implementations.
- Seeded differential testing against the reference implementation found no disagreements.

This is internal engineering verification by the project. It is **not** independent third-party testing, an audit or a certification.

The exercise also identified several points where the rule needs a more explicit written specification before a future version. These points are recorded privately. They did not change existing behaviour.

The independent implementations are **verification tools**. They are not firmware, are not deployed and do not run on embedded hardware. Using them on an embedded target would require separate re-validation.

## 5. Contract governance (under development)

A **provisional** contract register is being developed for the parts of the protocol that independent implementations must reproduce exactly. It provides:

- a version and a status for each contract, with no contract yet approved;
- fixed content identifiers for the files that define each contract;
- checks of the recorded history behind those identifiers;
- rules that require a version change when normative content changes, while previously published conformance material stays immutable;
- reviewed exceptions for changes that do not affect the contract;
- verification of controlled copies against an independently held reference.

**Verified results:** the automated governance checks passed in continuous integration on Python 3.12 and 3.13.

**Status:**

- All contracts remain **provisional**. None has been approved or published as a standard.
- The governance mechanism is itself an unmerged draft and has not been adopted.

## 6. Current experimental status

| Area | Status |
| --- | --- |
| Software change sets (security hardening, bench readiness, commemorative test material, conformance suite, contract governance) | **Experimental: multiple unmerged change sets. Not approved releases.** |
| Contracts | **Provisional.** None approved |
| Cross-language conformance | Verified in a controlled software environment |
| Bench procedures for a future physical run | Prepared and rehearsed in software. **Not executed** |
| Key operations | **None performed** |
| Deployment | **None performed** |

## 7. Production and hardware-validation limitations

This update does **not** establish:

- production readiness, certification or guaranteed security;
- validation of the **physical measurement path**. Software tests do not show that a measurement corresponds to the physical world;
- authenticated device identity, secure-element integration or hardware-backed signatures;
- completed firmware or embedded verification;
- end-to-end operation with running services in a field environment;
- latency, throughput, link quality, energy or recovery performance;
- legal, regulatory or compliance conformity.

The production-readiness gap described in this repository's [README](../README.md#production-readiness-gap) still applies in full.

## 8. Next validation stages

- Independent review of the unmerged experimental change sets before any merge.
- An authorized, controlled physical-bench run under the prepared procedures.
- A decision on approving the provisional contracts.
- Continued separation of public status from restricted implementation material.

---

*Public status document. It grants no license, and should not be used to infer undisclosed architecture, security properties or operational readiness.*
