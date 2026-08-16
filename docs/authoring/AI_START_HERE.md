# SayDone AI authoring: start here

Статус: `ТЕКУЩИЙ / AUTHORING-2A`
Версия набора: `1`

Подготовьте самостоятельный JSON `SayDoneAuthoringBundleV1` для бота, а не
`BotPackageV1`, идентификаторы состояния/правила/effect времени выполнения или
Telegram payload. Передайте Bundle в каноническую проверку. Если она вернула
`RepairReportV1`, исправьте только указанный путь исходного описания и создайте
небольшой `SayDoneAuthoringPatchV1`, привязанный к checksum/revision; Patch снова
проходит всю проверку.

## Ограниченный путь чтения

1. Откройте [manifest](AI_AUTHORING_MANIFEST_V1.json) и выберите только нужную
   тему, контракт и исполняемый пример.
2. Прочитайте [протокол диалога](AUTHORING_CHAT_PROTOCOL_V1.md), затем краткое
   [описание примитивов](AUTHORING_PRIMITIVES_V1.md) для выбранной темы.
3. Для Bundle, Patch, проверки и RepairReport используйте
   [контракт Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md).
4. Для доступных операций и событий смотрите
   [индекс возможностей](AI_CAPABILITY_EVENT_OPERATION_INDEX_V1.md); для кодов
   исправления — [каталог ошибок](AI_PUBLIC_ERROR_CATALOG_V1.md).

Открывайте дополнительные архитектурные контракты, только если manifest связывает
их с выбранным примитивом. Для внешней проверки фиксируйте список прочитанных
файлов и байтов; fresh-AI qualification относится к AUTHORING-2C.

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
