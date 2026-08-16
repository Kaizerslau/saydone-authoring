# SayDone AI authoring: start here

Статус: `CURRENT / AUTHORING-2A`
Версия набора: `1`

Создайте self-contained JSON `SayDoneAuthoringBundleV1` для бота, а не
`BotPackageV1`, runtime state/rule/effect IDs или Telegram payload. Передавайте
Bundle в canonical preview. Если preview вернул `RepairReportV1`, исправьте только
указанный source path и сформируйте маленький checksum/revision-bound
`SayDoneAuthoringPatchV1`; Patch снова проходит весь preview.

## Bounded reading path

1. Откройте [manifest](AI_AUTHORING_MANIFEST_V1.json) и выберите только нужный
   topic, contract и executable example.
2. Прочитайте [chat protocol](AUTHORING_CHAT_PROTOCOL_V1.md), затем concise
   [primitive guide](AUTHORING_PRIMITIVES_V1.md) для выбранной topic.
3. Для Bundle, Patch, preview и RepairReport используйте
   [Bundle/Patch contract](AUTHORING_BUNDLE_PATCH_V1.md).
4. Для доступных public operations/events смотрите
   [capability index](AI_CAPABILITY_EVENT_OPERATION_INDEX_V1.md); для repair-кодов —
   [error catalog](AI_PUBLIC_ERROR_CATALOG_V1.md).

Откройте дополнительные architecture contracts только если manifest связывает их
с выбранной primitive. Фиксируйте список прочитанных файлов/байтов у внешней
qualification; fresh-AI qualification принадлежит AUTHORING-2C.

## Что нельзя генерировать

- generated state, rule, effect, callback или runtime IDs;
- database IDs, SQL, secrets, credential values, provider payload/JSON или
  Telegram `callback_data`;
- JS, eval, arbitrary expressions/templates, raw HTTP или deployment authority;
- ручную модификацию compiled `BotPackageV1` либо injection scenarios после
  canonical compiler.

Trusted organization/actor identity, lifecycle и confirmation принадлежат
SayDone, а не authoring source. `RepairReportV1` — static repair diagnostic;
он не заменяет runtime `ExecutionTraceV1`.

## Schema boundary

Полная JSON Schema для Bundle/Patch/BotSpec/Acceptance намеренно не публикуется:
canonical strict parser и versioned contracts ниже authoritative. Это исключает
ложную совместимость с unknown fields, duplicate JSON keys и source-level
semantic validation. Manifest явно фиксирует каждый такой статус.
