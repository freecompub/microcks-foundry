<!-- skraft-architecture-brief:v1
{"version":1,"issue":2,"forge":"github","repository":"freecompub/microcks-foundry","bodySha256":"2b61da9744406ff7f4973fccc10721626e5d22b5dcb768a4515bae105517e456","components":["microcks-foundry"],"criteria":[{"id":"AC-01","components":["microcks-foundry"],"adrs":[]},{"id":"AC-02","components":["microcks-foundry"],"adrs":[]},{"id":"AC-03","components":["microcks-foundry"],"adrs":[]},{"id":"AC-04","components":["microcks-foundry"],"adrs":[]},{"id":"AC-05","components":["microcks-foundry"],"adrs":[]}],"adrs":[],"documents":["docs/architecture/context-map.md","docs/architecture/contract.md","docs/architecture/design.md","docs/architecture/proposal.md","docs/architecture/delivery.md","docs/architecture/space.md","docs/domain/event-model/contract.md","docs/domain/event-model/design.md","docs/domain/event-model/proposal.md","docs/domain/event-model/delivery.md","docs/domain/event-model/space.md"]}
-->
# Architecture brief — EP-001-S01, a ZIP of nominal examples from an OpenAPI

One path through every layer: a REST contract is attached to a space, one nominal example per
operation is designed and submitted for decision, the approval freezes a snapshot, and the archive
exists only after that approval. A document that is not OpenAPI 3.x never gets that far.

Contexts touched: `contract`, `design`, `proposal`, `delivery`, `space`. No ADR: every structural
choice this story needs is already stated in `ARCHITECTURE.md`, `CONTRIBUTING.md` or the context
documents below, and no candidate passed the eligibility gate (see the consistency matrix of this
pass).

## Criterion to components

| Criterion | Component | Components of the design it exercises |
|---|---|---|
| AC-01 — one nominal example per operation | microcks-foundry | `ContractRevisionReader` and `ContractRevision` → `ContractOperationList`; `AlgorithmCatalog`, `ValueResolver`, `DeterministicValueGenerator`, `NominalExampleDesigner` → `DesignedExample`, `DesignedExampleSet`, `DesignTechnique` = `Nominal`; `GenerationManifest`; `ApiExamplesDocumentWriter`; `ExamplesGenerated`; read model `DesignedExampleSetView` |
| AC-02 — the ZIP exists only after approval | microcks-foundry | `Proposal` with `ReviewItem`, `Decision`, `CheckReport`, `ApprovedSnapshot`; `ApiExamplesValidator`; `ProposalApproved`; `Delivery`, `ArchiveName`, `ArchiveEntry`, `ArchiveWriter`, `ArchivePrepared`; read models `ProposalReview` and `DeliveryView` (`Exists`, `DownloadAvailable`) |
| AC-03 — no approval while an item is pending | microcks-foundry | `Proposal` invariant on approval, `Decision` = `Pending`; read model `ProposalReview` (`PendingItemCount`, `ApprovalRefusalReason`); no `ProposalApproved`, no `Delivery` |
| AC-04 — a contract that is not REST is refused | microcks-foundry | `ContractFormat`, `ContractRevision` construction rule, `UnsupportedContractFormat`; `RegisterManagedApi` error result; read model `ContractLoadResult` (`Accepted`, `RefusalReason`, `SupportedFormats`) |
| AC-05 — the journey is proved end to end | microcks-foundry | the whole chain above driven through the application's own entry point in `foundry-app`, in this order: `RegisterManagedApi` → `ContractLoadResult`, `GenerateExamples` → `ExamplesGenerated` and `DesignedExampleSetView`, `GetProposal` → `ProposalReview`, `AcceptItem` per item, `RunPreDeliveryChecks` → `CheckReportView`, `ApproveProposal` → `ProposalApproved`, `PrepareArchive` → `ArchivePrepared`, `GetDelivery` → `DeliveryView` (`DownloadAvailable`, `DownloadLocation`, `Entries`), then the archive fetched at that location. The APIExamples document it carries is compared with the versioned API Pastry 2.0 reference document; a mismatch prints the diff (`ARCHITECTURE.md` § Test organization). No browser, no front-end component and no new gate take part. |

## Shared documents cited

- `docs/architecture/context-map.md` — contexts, subdomain classes and relationships
- `docs/architecture/contract.md`, `docs/domain/event-model/contract.md` — attaching a REST
  contract, refusing another format, publishing the operations
- `docs/architecture/design.md`, `docs/domain/event-model/design.md` — designing the nominal
  examples, the manifest, the document writer
- `docs/architecture/proposal.md`, `docs/domain/event-model/proposal.md` — decision, checks,
  approval and the frozen snapshot
- `docs/architecture/delivery.md`, `docs/domain/event-model/delivery.md` — the archive of an
  approved snapshot and its availability
- `docs/architecture/space.md`, `docs/domain/event-model/space.md` — the space the whole path is
  scoped by

## Reuse

- Reuse as-is: the layer modules and their dependency direction, the ArchUnit guard, the Gherkin
  runner and the build gates (research § Project conventions, § Handoff to DESIGN).
- Extend: `ARCHITECTURE.md`'s ratified `Proposal`, `ManagedApi`, `Delivery` and `TeamSpace`
  aggregates, their commands and their events — this story fills them for the first time, it does
  not redefine them.
- Create new: the Test design value objects and domain services, the four gateways named above, and
  the first real feature files, which retire the temporary `diagnostic` probe as
  `ARCHITECTURE.md` § Test organization requires.
- The structural scan reports no CQRS bus, no event store and no saga in the working copy; none is
  introduced.

## Assumptions carried

- The output path of the APIExamples document is the PRD's `examples/<api>-<version>.examples.yaml`
  (PRD §10), not the archive root the story's domain example shows: the PRD prevails on
  requirements (`ARCHITECTURE.md:10`).
- A document's format is detected by the adapter from what the document declares, and the context
  refuses anything that is not OpenAPI 3.x. Which parser library the adapter uses is left to the
  adapter.
- Nothing is persisted beyond the running instance: the repositories are in-memory
  implementations, and a durable adapter belongs to the story that needs survival across restarts.
- No AI takes part: every value comes from the contract or a deterministic generator.

## Decision the human took on AC-05

The surface of the end-to-end proof was a scope question, carried to the human in the previous
pass. The answer: the journey is proved through the application's own entry point — an HTTP-level
acceptance run over the versioned API Pastry 2.0 reference set that shows the diff on a mismatch —
and the browser run moves to the T5 UI story, where the web component, its stack and its gate are
decided. The product owner rewords AC-05 accordingly; the architecture of this brief is unchanged
by that rewording, and the convention for where such a proof enters and what it compares is written
in `ARCHITECTURE.md` § Test organization.

Consequences carried by this answer:

- the archive of an approved snapshot is fetchable through the entry point: `DeliveryView` carries
  `DownloadAvailable` and `DownloadLocation`, and `ArchiveWriter` hands the stored archive back when
  it is fetched (`docs/architecture/delivery.md` § Gateways);
- the reference set and the expected APIExamples document are versioned test resources, so the
  comparison and its diff need no running UI and no browser tooling;
- nothing in this slice proves the review screen; that proof arrives with the T5 UI story.
