# Ordinary-chat authoring protocol V1

Статус: `CURRENT / AUTHORING-2B`
Версия: `1`

Этот protocol предназначен для ordinary capable AI chat. Agent/tools mode может
автоматизировать те же шаги, но не добавляет source semantics или authority.

## Workflow

1. Преобразуйте business requirements в stable logical keys, canonical
   DataFields, static resources/content и independent acceptance scenarios.
2. Соберите один strict JSON `SayDoneAuthoringBundleV1`: `botSpec`,
   `dataFields`, `acceptance`, `qualification`.
3. Пользователь передаёт Bundle в canonical import/preview boundary.
4. При `confirmation_ready` дальнейшее confirmation/import принадлежит trusted
   SayDone boundary; AI не выполняет его сама.
5. При `RepairReportV1` найдите `code` и public `/artifacts/...` `path`, измените
   только соответствующий source и создайте маленький `SayDoneAuthoringPatchV1`.
6. Patch обязан содержать exact `baseRevision` и `baseChecksum` текущего
   snapshot. Повторяйте preview до success; stale Patch fail-closed.

## Required source separation

| Need | Canonical source artifact |
|---|---|
| Interactive behavior, content, static resources | `botSpec` |
| Typed business values | `dataFields` |
| Independently observable scenarios | `acceptance` |
| Business requirements over scenarios | `qualification` |

Bundle/Patch shape, bounds and trusted handoff are exact only in
[Bundle/Patch contract](AUTHORING_BUNDLE_PATCH_V1.md). Examples are available
through the [manifest](AI_AUTHORING_MANIFEST_V1.json).

## AI must not generate

- runtime/package/database IDs, generated state/rule/effect/callback keys,
  checksums or revisions;
- SQL, credentials, secrets, provider payloads or private endpoints;
- direct deployment/lifecycle operations;
- raw `START`, `TEXT_RECEIVED`, `COMMAND_RECEIVED` or `UNKNOWN_INPUT` as public
  `on.event` DSL.

Use static logical keys and only registered public operations/events from the
[capability index](AI_CAPABILITY_EVENT_OPERATION_INDEX_V1.md). `RepairReportV1`
is static source repair; runtime `ExecutionTraceV1` remains a separate diagnostic.
