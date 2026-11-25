<!-- skraft-architecture-brief:v1
{"version":1,"issue":4,"forge":"github","repository":"freecompub/microcks-foundry","bodySha256":"a2e65047ba7e0851b84bc19c2a5065614e80e2d5031b76b94f4745f665c388e9","components":["microcks-foundry"],"criteria":[{"id":"AC-01","components":["microcks-foundry"],"adrs":[]},{"id":"AC-02","components":["microcks-foundry"],"adrs":[]},{"id":"AC-03","components":["microcks-foundry"],"adrs":[]}],"adrs":[],"documents":["docs/architecture/context-map.md","docs/architecture/contract.md","docs/architecture/design.md","docs/architecture/proposal.md","docs/domain/event-model/contract.md","docs/domain/event-model/design.md","docs/domain/event-model/proposal.md"]}
-->
<!-- markdownlint-disable-file -->
# Architecture brief — story #4, EP-001-S05

One example per declared error response and one negative example per declared constraint; an
operation that declares no error response is reported, never given an invented one.

Contexts: `contract`, `design`, `proposal` ([context-map.md](../context-map.md)).

## Criteria

| Criterion | Components | Slices and elements | ADRs |
| --- | --- | --- | --- |
| AC-01 — one example per declared error response | microcks-foundry | Slice TD-2, on the `DeclaredResponse` values the published contract model of slice CG-1 exposes. `DeclaredErrorExampleDesigner` produces one `GeneratedExample` per declared error response, with `Technique: DeclaredError`, a value absent from the contract, and the declared response it reproduces. Observed on `ExampleGenerationReport` and on the `ProposalReview` items of slice PR-1. | none |
| AC-02 — one negative example per constraint | microcks-foundry | Slice TD-2. `NegativeConstraintExampleDesigner` produces one `GeneratedExample` per `Constraint` of the request schema — `Type`, `Format`, `Pattern`, `Required` — each carrying `Technique: NegativePerConstraint`, its `ViolatedConstraint` and the declared validation error response of its operation. | none |
| AC-03 — no invented error | microcks-foundry | Slice TD-2. An operation whose `DeclaredResponses` holds no error response yields no `DeclaredError` and no `NegativePerConstraint` example, and `ExampleDesigner` adds a `GenerationGap` of kind `NoDeclaredErrorResponse` for it, which `ExampleGenerationReport.Gaps` shows. | none |

## Shared documents cited

- [docs/architecture/context-map.md](../context-map.md) — contexts of this story and their relationships.
- [docs/architecture/contract.md](../contract.md) — `DeclaredResponse`, `Constraint`, `Schema` and the evolution rule of the published contract model.
- [docs/architecture/design.md](../design.md) — `Technique`, `GeneratedExample`, `GenerationGap`, `DeclaredErrorExampleDesigner`, `NegativeConstraintExampleDesigner`.
- [docs/architecture/proposal.md](../proposal.md) — one `ReviewItem` per produced example, with its technique and provenance.
- [docs/domain/event-model/contract.md](../../domain/event-model/contract.md), [design.md](../../domain/event-model/design.md), [proposal.md](../../domain/event-model/proposal.md) — the slices above with their fields.

## Assumptions carried

- Q4 (open): the absence of a declared error response is reported in the generation report of
  this slice, as a `GenerationGap`. The Coverage context is not opened by this story.
- A declared error response is a response whose status code is in the client-error or
  server-error range; `DeclaredResponse.ResponseKind` derives that from the status code.
- Q3 (open): each example carries its technique label and the `AlgorithmRef` of the design
  algorithm that produced it.
