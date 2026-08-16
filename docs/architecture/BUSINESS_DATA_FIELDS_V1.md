# Business Data Fields V1

Статус: `IMPLEMENTED / VERIFIED (DATA-1)`

## 0. Factual DATA-1 ownership audit

`DataFieldCatalogV1` is an independent versioned source artifact beside BotSpec
(the future Authoring Bundle owner), not an implicit BotSpec extension. The
strict canonical compile seam accepts BotSpec, catalog, optional acceptance and
controlled compile context; only `bot-authoring` assembles the final package.
Its immutable `BotDefinitionV1.dataFields` manifest contains runtime metadata,
constraints, bot configuration and bounded system resolver identity, so runtime
does not need the source catalog.

Current source contract is `{ dataFieldCatalogVersion: 1, fields: [...] }`.
Keys use `^[a-z][a-z0-9_]{0,63}$`, reject prototype-polluting names and are
globally unique per catalog, so same-key scope shadowing is impossible. Supported
types are `text`, `number`, `boolean`, `date`, `datetime`; visibility is
`standard`, `technical`, `hidden`. `null`, `""`, `0`, and `false` validate as
distinct values; `undefined` is not a value and represents missing. No implicit
coercion is performed. `format: phone` uses bounded `+?[1-9][0-9]{6,14}` syntax;
`email` uses deterministic bounded syntax validation; `date` is `YYYY-MM-DD` and
`datetime` is canonical RFC3339/ISO datetime. `min`/`max` are number-only;
`allowedValues` and bot `defaultValue` must match type and constraints.

Physical owners remain distinct:

- `flow` — `flow_instances.context_json`;
- `user` — `organization_user_bot_data_field_values`, owned through
  `organization_user_bot_profiles`;
- `bot` — version-pinned immutable manifest `defaultValue`;
- `system` — code-owned resolver; V1 exposes only `current_datetime` through
  `event_occurred_at`.

Reducer receives a deterministic scoped snapshot and remains pure. It returns
separate flow/user patch and unset mutations; PostgreSQL persists user mutations
inside the same transaction as inbound event, flow revision, transition and
effect outbox. Absence of a user-value row is missing; JSONB `null` is present
null. Deleted manifest fields keep their durable rows but are not visible to a
version that no longer declares them; no destructive cleanup occurs.

## 1. Identity and namespace

`DataFieldV1` — canonical definition места для business value; input — только
один writer. Stable machine key (`customer_phone`, `crm_lead_id`) не
переименовывается косметически; Unicode label и optional description могут
меняться без изменения runtime identity.

```text
ONE MACHINE KEY → ONE CANONICAL FIELD DEFINITION → ONE SCOPE
```

Implicit `flow.phone`/`user.phone`/`bot.phone` shadowing и runtime identities
вроде `field_#{i}` запрещены: они ломают static validation, usage graph,
safe delete/Patch, impact analysis и AI repair. Collections принадлежат
`DataSetV1`/DATA-2, а не dynamic field names.

## 2. Types, formats and value states

Initial storage types: `text`, `number`, `boolean`, `date`, `datetime`. Storage
type не равен capture/validation format: `type=text, format=phone` допустим.
DATA-1 может задать bounded `format`, `allowedValues`, `min`, `max`, но
не раздувает type system и не вводит arbitrary expression language.

Contract обязан deterministic различать field missing, `null`, `""`, `0`,
`false`. Clear/unset — explicit operation/semantic; empty string не является
universal deletion. Pure conditions и mappings используют те же semantics.

## 3. Scopes

- `flow`: read/write temporary flow-instance state;
- `user`: read/write persistent per-user business state;
- `bot`: configuration-owned; normal running flow по умолчанию не
  изменяет его generic `set`;
- `system`: controlled read-only values/resolvers.

Generic `input`, `set` and `unset` may write only `flow` and `user`. Bot/system
writes fail at strict authoring validation. An existing key may retain scope/type
across versions; label/description/visibility changes and non-destructive removal
are compatible, while scope/type mutation fails lifecycle import.

Global counters, stock, capacity, queues и available slots не маскируются
под `bot` field. Они принадлежат Dataset или domain service/module с
explicit concurrency semantics.

## 4. Writers, readers and presentation

Writers: input, set/flow operation, HTTP response, module result, controlled
provider/system source и future bounded editing. Readers: content, HTTP mapping,
module calls, analytics, acceptance и safe conditions.

Presentation visibility: `standard`, `technical`, `hidden`.

```text
visibility != authorization
hidden != secret
```

Secrets никогда не Data Fields и остаются в SecretStore. Normal UX показывает
label/business meaning; technical mode — key, scope, type, writers/readers,
provenance и usage. Optional groups/categories — presentation-only metadata.
AL-1E SafeTransforms вычисляют bounded scalar только внутри explicit BotSpec
`set`. Registered catalog statically связывает argument/output types с canonical
DataFields; runtime читает pre-transition view и отдельно проверяет target
constraints. `current_datetime` по-прежнему берётся только из
`event_occurred_at`, поэтому timestamp/replay не зависят от wall clock. Transform
не создаёт computed-field storage, hidden mutation или второй namespace. Nested
expressions, JS/eval/general formula language и side effects запрещены.
