<!-- skraft-architecture-brief:v1
{"version":1,"issue":2,"forge":"github","repository":"freecompub/microcks-foundry","bodySha256":"2b61da9744406ff7f4973fccc10721626e5d22b5dcb768a4515bae105517e456","components":["microcks-foundry"],"criteria":[{"id":"AC-01","components":["microcks-foundry"],"adrs":[]},{"id":"AC-02","components":["microcks-foundry"],"adrs":[]},{"id":"AC-03","components":["microcks-foundry"],"adrs":[]},{"id":"AC-04","components":["microcks-foundry"],"adrs":[]},{"id":"AC-05","components":["microcks-foundry"],"adrs":[]}],"adrs":[],"documents":["docs/architecture/context-map.md","docs/architecture/contract.md","docs/architecture/design.md","docs/architecture/proposal.md","docs/architecture/delivery.md","docs/architecture/space.md","docs/domain/event-model/contract.md","docs/domain/event-model/design.md","docs/domain/event-model/proposal.md","docs/domain/event-model/delivery.md"]}
-->
<!-- markdownlint-disable-file -->
# Architecture brief — story #2, EP-001-S01

Nominal examples from an OpenAPI contract, reviewed, approved and downloaded as an archive; an
unsupported contract format is refused.

Contexts: `contract`, `design`, `proposal`, `delivery`, `space`
([context-map.md](../context-map.md)).

## Criteria

| Criterion | Components | Slices and elements | ADRs |
| --- | --- | --- | --- |
| AC-01 — one nominal example per operation | microcks-foundry | Slice CG-1 then TD-1. `ContractDocumentReader` yields the `ContractRevision`; `NominalExampleDesigner` produces one `GeneratedExample` per `Operation`, conforming to its declared response schema and carrying `Technique: Nominal`, its `AlgorithmRef` and the `ValueOrigin` of each value. Observed on `ExampleGenerationReport` and on the `ProposalReview` items of slice PR-1. | none |
| AC-02 — the archive exists only after approval | microcks-foundry | Slices PR-2, PR-3 then DL-1. `Proposal` freezes an `ApprovedSnapshot` once every `ReviewItem` is decided and `CheckReport` passed on the current revision, including `ApiExamplesSchemaValid` through `ApiExamplesDocumentValidator`; `Delivery` is created only from that snapshot and `ArchiveWriter` packs it. Observed on `ProposalReview.DownloadAvailable` (no before approval) and on `ArchiveDownload.Entries`. | none |
| AC-03 — no approval with a pending item | microcks-foundry | Slice PR-3. The `Proposal` invariant "a decision on every item" refuses `ApproveProposal` and names the pending count; the refusal changes no state, raises no event and creates no `Delivery`. | none |
| AC-04 — an unsupported contract format is refused | microcks-foundry | Slice CG-1. `ContractDocumentReader` reports the `ContractFormat` it detected; the `SupportedContractFormats` domain service refuses and its reason names the supported REST contract format. No `ManagedApi` is registered and no event is raised. | none |
| AC-05 — the journey is proved end to end | microcks-foundry | The whole chain of slices CG-1, TD-1, PR-1 to PR-3 and DL-1, exercised against the versioned reference contract with the real document reader, document writer and archive writer, and a diff shown when the produced file differs from the expected one. Open question Q1 decides whether the proof runs in a browser or at the use-case and HTTP level; the design is the same either way, only the entry point of the test changes. | none |

## Shared documents cited

- [docs/architecture/context-map.md](../context-map.md) — contexts of this story and their relationships.
- [docs/architecture/contract.md](../contract.md) — `ManagedApi`, the published contract model, `SupportedContractFormats`, `ContractDocumentReader`.
- [docs/architecture/design.md](../design.md) — `ExampleDesigner`, `NominalExampleDesigner`, `GenerationManifest`, `ExampleDocumentWriter`.
- [docs/architecture/proposal.md](../proposal.md) — `Proposal`, `ReviewItem`, `CheckReport`, `ApprovedSnapshot`, `ApiExamplesDocumentValidator`.
- [docs/architecture/delivery.md](../delivery.md) — `Delivery`, `Channel`, `ArchiveEntry`, `ArchiveWriter`.
- [docs/architecture/space.md](../space.md) — the team-space scope every command and query of this story carries.
- [docs/domain/event-model/contract.md](../../domain/event-model/contract.md), [design.md](../../domain/event-model/design.md), [proposal.md](../../domain/event-model/proposal.md), [delivery.md](../../domain/event-model/delivery.md) — the slices above with their fields.

## Assumptions carried

- Q1 (open): the end-to-end proof of AC-05 runs at the use-case and HTTP level in the existing
  Java build, and the browser run arrives with the review screen. No front-end component, gate or
  CI browser job is designed here.
- Q2 (open): the archive holds the examples document under `examples/`, as the PRD's output
  layout states.
- Q3 (open): each example carries its technique label and the `AlgorithmRef` of the design
  algorithm that produced it; each value keeps the origin of its own source.
- The contract document is uploaded in this slice: `ContractSourceRef` gains an `Upload` kind, and
  no Microcks or git integration is designed here.
- No store is chosen: proposal and delivery state stays behind their repositories
  ([proposal.md](../proposal.md) § Repositories).
