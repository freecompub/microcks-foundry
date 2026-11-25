<!-- markdownlint-disable-file -->
# Event model — Team spaces (`space`)

Scope of this slice set: the minimal isolated space of one team on a shared instance, and the
membership every other context checks before it answers. Export tokens, AI providers and retention
are described in `ARCHITECTURE.md` and are not modelled until their own slice.

State changed by these slices: the space itself, written once with its slug, its display name and
its members. It is a self-validating entity, not an aggregate root, so this context raises no
domain event here ([../../architecture/space.md](../../architecture/space.md) § Aggregates,
§ Domain events).

```mermaid
timeline
    title Team spaces
    section Command
        The platform creates a team space : CreateTeamSpace
    section Event
        None in this slice set : no aggregate root of this context
    section Read Model
        The space and its members : TeamSpaceView
        Whether a person belongs to a space : SpaceMembershipView
```

## Slice SP-1 — Create an isolated team space

| Step | Kind | Name | Fields |
|---|---|---|---|
| 1 | Command | `CreateTeamSpace` | `Slug: SpaceSlug`, `DisplayName: text`, `Members: list of MemberId`, `CreatedBy: MemberId` |
| 2 | Read Model | `TeamSpaceView` | `TeamSpaceId: TeamSpaceId`, `Slug: SpaceSlug`, `DisplayName: text`, `Members: list of MemberId` |

Observed: a `TeamSpaceView` per space, two spaces of the same instance sharing no member unless the
platform placed the same person in both; a second space with the same slug is refused and no second
`TeamSpaceView` exists for that slug.

## Slice SP-2 — Read the space a person belongs to

| Step | Kind | Name | Fields |
|---|---|---|---|
| 1 | Query | `GetTeamSpace` | `RequestedBy: MemberId`, `Slug: SpaceSlug?` |
| 2 | Read Model | `TeamSpaceView` | as in SP-1 |

Observed: `TeamSpaceView` for a space the caller belongs to; nothing for a space they do not.

## Slice SP-3 — Answer every other context's membership question

| Step | Kind | Name | Fields |
|---|---|---|---|
| 1 | Query | `FindSpaceMembership` | `TeamSpaceId: TeamSpaceId`, `MemberId: MemberId` |
| 2 | Read Model | `SpaceMembershipView` | `TeamSpaceId: TeamSpaceId`, `MemberId: MemberId`, `Member: yes/no` |

Observed: `SpaceMembershipView.Member` yes for a member of that space and no for anyone else,
including when the space does not exist — which is what makes every command and every read of
another context refusable before it runs, and what keeps one team's dictionaries, examples and
proposals out of another team's answers.

The trigger of this slice comes from outside the use case: another context asks before it answers
its own caller. `GetTeamSpace` and `FindSpaceMembership` are pure reads: they raise no event. The
caller's identity is established outside the model, at the entry point.

## Facts this slice set does not model as events

| Fact | Why not here |
|---|---|
| `TeamSpaceCreated` | The ratified event (`ARCHITECTURE.md` § Domain events) is consumed by Traffic observation and Test design for state neither holds in this slice set, and no acceptance criterion observes it. This context has no aggregate root here, and a fact recorded outside an aggregate is not a domain event. The space is observed in `TeamSpaceView`. |
| `TrafficTokenIssued`, `TrafficTokenRevoked`, `AiProviderSelected`, `AiProviderEnabled`, `AiProviderDisabled` | Their commands are out of this slice set (PRD E2.7, E4.5). |
