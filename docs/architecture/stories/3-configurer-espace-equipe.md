<!-- skraft-architecture-brief:v1
{"version":1,"issue":3,"forge":"github","repository":"freecompub/microcks-foundry","bodySha256":"3c0cf3e651b1df01e5175e437e6625a37829a1b8ebe351a2c9aaaf0a9e70af93","components":["microcks-foundry"],"criteria":[{"id":"AC-01","components":["microcks-foundry"],"adrs":[]},{"id":"AC-02","components":["microcks-foundry"],"adrs":[]},{"id":"AC-03","components":["microcks-foundry"],"adrs":[]}],"adrs":[],"documents":["docs/architecture/context-map.md","docs/architecture/space.md","docs/architecture/teamcontext.md","docs/architecture/design.md","docs/domain/event-model/space.md","docs/domain/event-model/teamcontext.md","docs/domain/event-model/design.md"]}
-->
# Architecture brief — EP-001-S03, configure the team space

A team space is the partition every aggregate carries and every read is scoped by; the team's own
`.foundry/` content is read at a revision into one immutable snapshot whose seed makes a generation
reproducible, and a configuration with no seed refuses the generation while naming the file and the
key.

Contexts touched: `space`, `teamcontext`, `design`. No ADR: isolation by `TeamSpaceId` and the
seed as an input are already ratified in `ARCHITECTURE.md` and `CONTRIBUTING.md`, and no candidate
passed the eligibility gate (see the consistency matrix of this pass).

## Criterion to components

| Criterion | Component | Components of the design it exercises |
|---|---|---|
| AC-01 — the content of a space is invisible from another | microcks-foundry | `TeamSpace` entity with its members; `TeamSpaceId` as the shared kernel; `TeamSpaceRepository` scoped by it; `FindSpaceMembership` → `SpaceMembershipView`, the published read every other context calls before it answers; `GetTeamSpace`; `GetTeamContext` with `TeamContextSource`; read models `TeamSpaceView` and `TeamContextView` (`Dictionaries`) |
| AC-02 — the seed of the configuration makes a generation reproducible | microcks-foundry | `TeamContextSource`, `TeamConfiguration`, `Seed`, `TeamContextSnapshot` and its content hash; read model `TeamContextView`; `GenerationManifest`, `DeterministicValueGenerator`, `ApiExamplesDocumentWriter`; read model `DesignedExampleSetView` (`DocumentHash`, `GenerationManifestHash`) |
| AC-03 — no generation without a seed | microcks-foundry | `TeamConfiguration` construction rule and `MissingConfigurationKey`; `GenerateExamples` error result; read models `TeamContextView` and `GenerationReport`, both carrying `Accepted`, `RefusalReason`, `ConfigurationFile` and `MissingConfigurationKey` under those same names; no `ProposalSubmitted` and no proposal |

## Shared documents cited

- `docs/architecture/context-map.md` — the shared kernel and the Customer/Supplier relationship
  that carries the snapshot to Test design
- `docs/architecture/space.md`, `docs/domain/event-model/space.md` — the space, its members and the
  rule every other context applies
- `docs/architecture/teamcontext.md`, `docs/domain/event-model/teamcontext.md` — the configuration,
  the seed, the dictionaries and the snapshot
- `docs/architecture/design.md`, `docs/domain/event-model/design.md` — where the seed enters the
  generation and how byte stability is produced

## Reuse

- Reuse as-is: `TeamSpaceId` as the shared kernel and the rule that every stored identity carries it
  (`ARCHITECTURE.md` § Context map, `CONTRIBUTING.md` § Where code belongs); the
  `GenerationManifest` and the determinism rules (`ARCHITECTURE.md` § Reproducibility, rule `R5`).
- Narrow for this slice set: the ratified `TeamSpace` is built as a self-validating entity with its
  slug, display name and members only, because no story of this pass changes a space after it is
  created; its aggregate invariants, its token and its providers arrive with their stories
  (`docs/architecture/space.md` § Aggregates).
- Create new: the published membership read of `application.space.api`, the Team context value
  objects and its one gateway; no aggregate and no repository in that context.

## Assumptions carried

- The configuration keys are the story's: `seed` and `prLanguage` in `.foundry/config.yaml`. The
  PRD states the file's content without fixing key names, so the adapter reads these and the domain
  knows them as `Seed` and `PrLanguage`.
- The caller's identity is established at the entry point — an OIDC identity when Foundry runs on
  its own, the host's session when it is embedded — and reaches the use case as a `MemberId`. The
  domain reads no token.
- `prLanguage` is read into the snapshot but no behaviour of this story consumes it; the pull
  request description that uses it belongs to the pull-request slice.
- Byte stability needs both the deterministic values of the domain and a fixed serialisation in the
  writer adapter; the proof is a reference-output comparison across separate runs
  (`ARCHITECTURE.md` § Reproducibility).
