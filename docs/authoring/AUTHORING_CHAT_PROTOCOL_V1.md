# Протокол создания ботов в обычном диалоге V1

Статус: `CURRENT / AUTHORING-2B`
Версия: `1`

Этот протокол предназначен для обычного диалога с ИИ. Режим агента или
инструментов может автоматизировать те же шаги, но не добавляет семантику
исходного описания или полномочия.

## Workflow

1. Преобразуйте требования к боту в стабильные логические ключи, канонические
   DataFields, статические ресурсы/контент и независимые acceptance scenarios.
2. Соберите один корректный JSON `SayDoneAuthoringBundleV1`: `botSpec`,
   `dataFields`, `acceptance`, `qualification`.
3. Пользователь передаёт Bundle в canonical import/preview boundary.
4. При `confirmation_ready` дальнейшее подтверждение/import относится к trusted
   границе SayDone; ИИ не выполняет его сама.
5. При `RepairReportV1` найдите `code` и public `/artifacts/...` `path`, измените
   только соответствующий source и создайте маленький `SayDoneAuthoringPatchV1`.
6. Patch обязан содержать точные `baseRevision` и `baseChecksum` текущего
   описания. Для одного внешнего кейса допускаются initial ответ и не более двух
   циклов Patch-исправления. После второго неуспешного Patch внешний прогон
   останавливается для внутреннего разбора; устаревший Patch отклоняется.

## RepairReport и маршрут исправления

`RepairReportV1` сообщает `code`, public source `path`, `affectedGate` и
`documentationTopic`. Последнее поле — только навигация к уже опубликованной
теме manifest, а не второй язык, новая семантика или готовый ответ. По нему
откройте существующий контракт и исполняемый пример, затем создайте минимальный
Patch указанного типа и exact binding. Матрица примитив → пример → код ошибки →
тип Patch публикуется в
[матрице полноты](PUBLIC_CONTRACT_COMPLETENESS_V1.json).

## Acceptance и coverage без внутренних идентификаторов

Каждый достижимый авторский путь должен иметь независимый acceptance scenario:
для start/command/text/input/fallback описывайте logical block, ожидаемое
сообщение, DataField или наблюдаемое действие. Coverage сверяет скомпилированный
граф, но RepairReport возвращает только authored source path; generated runtime
state/rule/effect IDs не являются частью authoring source и не нужны для
исправления. Standalone примеры в manifest проходят путь source → compile →
inspect → validate → ScenarioV2 → coverage → qualification.

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
