<!-- markdownlint-disable-file -->
# Event model — Contract governance (`contract`)

Scope of this slice set: attaching a REST contract to a team space, refusing a document that is not
OpenAPI 3.x, and publishing the operations, their declared responses and their field constraints as
the published language the Test design and the Proposal contexts read.

State changed by these slices: the registration of one API inside a space. It is a write-once
entity rather than an aggregate root, because no story of this slice set changes it afterwards
([../../architecture/contract.md](../../architecture/contract.md) § Aggregates); it still records
the fact of its registration, which is the successful outcome of the command that created it.
A `ContractRevision` is a value object read from the source, never state this context owns.

The names below are the ones the skeleton and the code use. Rules are not restated here: the
acceptance scenarios hold them.

```mermaid
timeline
    title Contract governance
    section Command
        Register a managed API : RegisterManagedApi
    section Event
        The API is attached to the space : ManagedApiRegistered
    section Read Model
        Outcome of the load : ContractLoadResult
        Operations of the contract : ContractOperationList
```

## Slice CG-1 — Attach a REST contract to the space

| Step | Kind | Name | Fields |
|---|---|---|---|
| 1 | Command | `RegisterManagedApi` | `TeamSpaceId: TeamSpaceId`, `RequestedBy: MemberId`, `ApiName: text`, `ApiVersion: text`, `ContractSourceRef: ContractSourceRef` |
| 2 | Event | `ManagedApiRegistered` | `TeamSpaceId: TeamSpaceId`, `ManagedApiId: ManagedApiId`, `ContractRevisionHash: text`, `RegisteredBy: MemberId` |
| 3 | Read Model | `ContractLoadResult` | `Accepted: yes/no`, `RefusalReason: text?`, `ManagedApiId: ManagedApiId?`, `ApiName: text?`, `ApiVersion: text?`, `ContractRevisionHash: text?`, `DetectedFormat: ContractFormat`, `SupportedFormats: list of ContractFormat` |

Observed: `ContractLoadResult.Accepted`, `ContractLoadResult.RefusalReason` and
`ContractLoadResult.SupportedFormats` when the loaded document is not OpenAPI 3.x, with no
`ManagedApiRegistered` recorded; `ContractLoadResult.ManagedApiId` and
`ContractLoadResult.ContractRevisionHash` when it is.

## Slice CG-2 — Publish the operations a generation designs from

| Step | Kind | Name | Fields |
|---|---|---|---|
| 1 | Query | `FindContractRevision` | `TeamSpaceId: TeamSpaceId`, `ManagedApiId: ManagedApiId`, `RequestedBy: MemberId` |
| 2 | Read Model | `ContractOperationList` | `ApiName: text`, `ApiVersion: text`, `ContractRevisionHash: text`, `Operations: list of OperationView` |
| 2a | Read Model | `OperationView` | `OperationKey: OperationKey`, `Summary: text?`, `SuccessResponse: ResponseView`, `DeclaredErrorResponses: list of ResponseView`, `FieldConstraints: list of FieldConstraintView` |
| 2b | Read Model | `ResponseView` | `StatusCode: whole number`, `Kind: one of Success \| ClientError \| ServerError`, `PayloadShape: text?` |
| 2c | Read Model | `FieldConstraintView` | `FieldPath: text`, `Kind: one of Type \| Format \| Pattern \| Required`, `Expectation: text` |

Observed: `ContractOperationList.Operations` — one entry per operation of the contract, each with
its `SuccessResponse`, its `DeclaredErrorResponses` (empty when the contract declares none) and its
`FieldConstraints`.

`FindContractRevision` is a pure read: it raises no event. `ContractOperationList` and the views it
carries are the published language of this context; they are extended by addition only, and a
reader ignores a field it does not know.

## Facts this slice set does not model as events

| Fact | Why not here |
|---|---|
| `PrimarySourceDesignated`, `ContractUpdateModeChanged`, `ContractAudited` | Their commands (PRD E2.4, E2.13, E1.9) are out of this slice set. |

`ManagedApiRegistered` has no consumer in this slice set — its ratified consumers are Traffic
observation, Divergences and Coverage (`ARCHITECTURE.md` § Domain events) — and no acceptance
criterion observes it: what the stories observe is `ContractLoadResult`. It is modelled all the
same, because it is the recorded successful outcome of `RegisterManagedApi`.
