# BotAcceptanceSpecV1

Статус: `IMPLEMENTED / VERIFIED (AUTHORING-1S)`

`BotAcceptanceSpecV1` — независимый AI-friendly источник ожидаемого поведения. Он использует logical block identities, business Data Field keys, business events и stable content/resource references, но не generated `stateKey`, `ruleKey`, `effectKey`, callback keys или database IDs.

```text
BotSpecV1 = что делает бот
BotAcceptanceSpecV1 = как независимо проверить поведение
QualificationContractV1 = какие business requirements должны быть доказаны
```

Только canonical `bot-authoring` имеет право lowering:

```text
BotAcceptanceSpecV1 → ScenarioV2 → BotPackageV1.scenarios
```

Loader не создаёт scenarios. Final package checksum вычисляется canonical package inspection только после сборки definition, content и scenarios.

## AUTHORING-1S factual implementation context

`compileBotSpecV1` сохранён совместимым и по-прежнему создаёт package с
`scenarios: []`. Новый additive public seam
`compileBotSpecWithAcceptanceV1(specSource, acceptanceSource, context)` сначала
использует тот же canonical compiler, затем lowering внутри `bot-authoring` и
собирает финальный package с `ScenarioV2[]`; `apps/authoring-tools` не изменяет
`package.scenarios`.

Acceptance JSON строго парсится (включая duplicate keys) и fail-closed принимает
только source-level steps: `start`, `text`, `button`, public `event`,
`scheduledEvent`, effect/clock/restart actions и assertions для block,
transition, context, message, effect, business event, schedule и pending work.
Generated IDs и произвольный ScenarioV2 passthrough запрещены unknown-field
validation. Source references разрешаются по `SourceMapV1` и structural lowering.

Naive `logical block → stateKey` mapping не применяется: `saveState:false`
transient block с target разрешается в final state, а state-preserving transient
не допускает `expectBlock` и требует `expectTransition: { preservesState: true }`.
`MechanicalProofPlanV1` остаётся compiler-owned mechanical evidence и никогда не
создаёт acceptance. Lowering pure/deterministic, без DB/network/LLM; проверка
финального package сохраняет обычное checksum inclusion scenarios.

Controlled authoring-tool envelope теперь принимает `{ spec, acceptance?, context }`;
CLI: `compile <bot-spec.json> <compile-context.json> [acceptance.json]`. Отсутствие
acceptance сохраняет legacy compile semantics и не означает qualification-ready
BotSpec-only artifact.

DATA-1 strict envelope принимает `{ spec, dataFields, acceptance?, context }`.
На этом пути `initialContext` и assertions `partial`/`exists`/`absent` принимают
только declared Data Field keys; present literal values проходят общий
no-coercion validator. Acceptance не содержит DB/profile IDs и не имитирует
persistent user storage: PostgreSQL qualification проверяет её отдельно.
