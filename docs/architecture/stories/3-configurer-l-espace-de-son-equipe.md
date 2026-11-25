<!-- skraft-architecture-brief:v1
{"version":1,"issue":3,"forge":"github","repository":"freecompub/microcks-foundry","bodySha256":"3c0cf3e651b1df01e5175e437e6625a37829a1b8ebe351a2c9aaaf0a9e70af93","components":["microcks-foundry"],"criteria":[{"id":"AC-01","components":["microcks-foundry"],"adrs":[]},{"id":"AC-02","components":["microcks-foundry"],"adrs":[]},{"id":"AC-03","components":["microcks-foundry"],"adrs":[]}],"adrs":[],"documents":["docs/architecture/context-map.md","docs/architecture/space.md","docs/architecture/teamcontext.md","docs/architecture/design.md","docs/domain/event-model/teamcontext.md","docs/domain/event-model/design.md"]}
-->
<!-- markdownlint-disable-file -->
# Architecture brief — story #3, EP-001-S03

An isolated team space whose seed and pull-request language come from `.foundry/config.yaml`.

Contexts: `space`, `teamcontext`, `design` ([context-map.md](../context-map.md)).

## Criteria

| Criterion | Components | Slices and elements | ADRs |
| --- | --- | --- | --- |
| AC-01 — a space's content is invisible to the others | microcks-foundry | Slice TC-1. `ListTeamDictionaries` carries the caller's `TeamSpaceId` and `MemberId`; the use case checks `TeamSpaceMembership` and `TeamContextSource` answers for that space only, so `TeamDictionaryList` cannot hold another space's dictionary. The scope rule that makes this hold for every read is [space.md](../space.md) § Team-space scope. | none |
| AC-02 — the configured seed makes a generation reproducible | microcks-foundry | Slice TD-1. `TeamContextSource` yields a `TeamContextSnapshot` holding `Seed` and `PrLanguage`; `GenerationManifestFactory` pins the contract hash, the snapshot hash, the techniques, the seed and the algorithm references into a `GenerationManifest`; `FormatValueGenerator` derives each value from the seed, the field path and the algorithm reference; `ExampleSet` fixes the order and `ExampleDocumentWriter` the bytes. Two runs of one manifest therefore produce identical files. | none |
| AC-03 — no generation without a seed | microcks-foundry | Slice TD-1. A `TeamContextSnapshot` cannot exist without a seed, so `TeamContextSource` yields a `MissingConfigurationKey` naming the configuration file and the missing key; the `GenerateExamples` use case refuses with that reason, changes no state and raises no event. | none |

## Shared documents cited

- [docs/architecture/context-map.md](../context-map.md) — contexts of this story and their relationships.
- [docs/architecture/space.md](../space.md) — `TeamSpaceId`, `MemberId`, `TeamSpaceMembership`, the team-space scope rule.
- [docs/architecture/teamcontext.md](../teamcontext.md) — `TeamContextSnapshot`, `PrLanguage`, `Dictionary`, `MissingConfigurationKey`, `TeamContextSource`.
- [docs/architecture/design.md](../design.md) — `GenerationManifest`, `GenerationManifestFactory`, `FormatValueGenerator`, `ExampleSet`, `ExampleDocumentWriter`.
- [docs/domain/event-model/teamcontext.md](../../domain/event-model/teamcontext.md), [design.md](../../domain/event-model/design.md) — the slices above with their fields.

## Assumptions carried

- The configuration key names of the story (`seed`, `prLanguage`) are the ones the adapter reads;
  they live in the `TeamContextSource` adapter, never in the domain, so renaming one touches one
  class and the refusal message it reports.
- The caller's identity comes from OIDC in standalone mode and from the host session when
  embedded, as the story's answered question states; `TeamSpaceMembership` is the only thing the
  rest of the design knows about it.
- The team context is read-only in this slice: no command of these stories writes `.foundry/`.
