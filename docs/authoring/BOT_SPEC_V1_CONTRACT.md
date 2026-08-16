# Контракт BotSpecV1

Статус: `ACCEPTED / CONTENT-1 IMPLEMENTED / HTTP-1A STATIC CONTRACT IMPLEMENTED`
Дата: 2026-08-15

## 1. Назначение

`BotSpecV1` — основной декларативный формат, который создаёт AI-автор.

```text
человеческое ТЗ
→ AI
→ BotSpecV1
→ bot-authoring
→ BotPackageV1
```

`BotSpecV1` описывает бизнес-поведение. Полный `BotPackageV1` остаётся машинным контрактом SayDone и генерируется кодом.

## 2. Каноническая сериализация

V1 использует **JSON**.

Причины:

- уже используется в SayDone tooling;
- строгая структура без YAML tags/anchors/merge keys;
- простая schema validation;
- легко генерируется современными AI-моделями;
- один transport format уменьшает implementation scope.

YAML может быть добавлен позднее только как adapter `YAML → тот же BotSpec object`. Это не должно создавать вторую семантику.

## 3. Верхний уровень

```json
{
  "specVersion": 1,
  "bot": {
    "name": "Консультант",
    "description": "Опционально",
    "defaultLocale": "ru"
  },
  "entry": "start",
  "assets": {},
  "content": {},
  "httpResources": {},
  "ruleSets": {},
  "blocks": {}
}
```

Правила:

- `specVersion` строго `1`;
- `bot.name` и `bot.defaultLocale` обязательны;
- `botKey`, runtime version и content release number приходят из compile/lifecycle context и не авторятся как логика бота;
- `entry` указывает существующий logical block;
- `blocks` непустой object;
- `subflows` — optional object статически именованных reusable flows;
- `assets` — optional non-empty catalog статических logical asset slots;
- `content` — optional non-empty object статически именованных content entries;
- `httpResources` — optional non-empty catalog статически именованных bounded
  outbound resources; он разрешён только на canonical DataField compile path;
- `ruleSets` — optional non-empty object статически именованных pure decision tables;
- unknown properties fail closed.

## 4. Logical block

Блок — authoring-time единица композиции.

```json
{
  "saveState": true,
  "actions": [],
  "input": null,
  "on": [],
  "fallback": null
}
```

`saveState`:

- `true` — блок представляет пользовательскую позицию, которую runtime должен уметь сохранить/восстановить;
- `false` — блок является временной последовательностью действий и сам не создаёт пользовательскую позицию.

По умолчанию `saveState = true`.

AI задаёт logical block id (`ask_name`, `choose_topic`), но не runtime state key.

### Ограничения transient block (`saveState:false`)

Он может содержать `actions` и `next`, но не может содержать `input`, `on` или `fallback`, потому что пользователь никогда не должен «остаться» в нём ждать событие.

Compiler встраивает его действия в вызывающий transition/handler. Если `next` отсутствует, текущая пользовательская позиция сохраняется. Если `next` задан, compiler продолжает разворачивание до stateful block.

Это является базовой семантикой дожимных/служебных действий без бизнес-типа «дожим».

## 5. Actions

Порядок `actions[]` является значимым и сохраняется при compilation.

V1 action types:

```text
send
set
track
schedule
cancelSchedule
call
unset
applyRuleSet
```

Это универсальные операции, а не типы бизнес-воронок.

## 6. `send`

Canonical source отделяет content identity от action:

```json
{
  "content": {
    "thanks": {
      "format": "plain",
      "text": "Спасибо, {{name}}",
      "variables": {
        "required": ["name"],
        "optional": []
      }
    }
  },
  "blocks": {
    "done": {
      "actions": [{
        "type": "send",
        "content": "thanks",
        "buttons": [
          {
            "text": "Продажи",
            "run": "submitted",
            "set": { "topic": "Продажи" },
            "aliases": ["продажи"]
          }
        ]
      }],
      "on": []
    }
  }
}
```

Допустимые `format`: ровно `plain`, `telegram_html`,
`telegram_markdown_v2`. CONTENT-1B реализует pure rendering: static authored
formatting проходит bounded validation, dynamic scalar values экранируются по
format. Required missing/null fail-closed; optional missing/null заменяется empty
value; итоговый empty text запрещён. Telegram provider limits/composition и
`parse_mode` остаются adapter boundary, а не BotSpec provider payload.

Правила explicit content:

- content key — static logical identity, а не runtime/provider ID;
- `variables.required` и `variables.optional` задаются явно, не пересекаются и
  содержат только canonical DataField keys;
- каждый placeholder `{{field_key}}` должен быть зарегистрирован; каждая required
  variable должна присутствовать в тексте;
- произвольные expressions/template DSL, binary/base64, `file_id` и provider JSON
  запрещены;
- compiler создаёт существующие Content Manifest/artifact entries и `SEND_CONTENT`
  wiring; content key не зависит от текста.

Logical asset V1 объявляется отдельно от binary/provider binding:

```json
{
  "assets": {
    "promo_banner": { "kind": "image", "required": true }
  },
  "content": {
    "thanks": {
      "format": "plain",
      "text": "Спасибо",
      "variables": { "required": [], "optional": [] },
      "asset": "promo_banner"
    }
  }
}
```

Допустимые kinds: `image`, `video`, `audio`, `document`, `animation`.
`required` задаётся явно: unresolved required slot отклоняет lifecycle validation;
unresolved optional slot допустим. Reference статический и обязан указывать
объявленный slot совместимого kind. Binary/base64, storage URL, Telegram
`file_id` и provider JSON в BotSpec запрещены. Internal lifecycle отдельно
разрешает logical slot в organization-owned ready `assets.id`, а
`transport_asset_bindings.provider_file_id` остаётся integration-scoped detail.

Legacy inline shorthand остаётся совместимым:

```json
{
  "type": "send",
  "text": "Спасибо, {{name}}",
  "buttons": [
    {
      "text": "Продажи",
      "run": "submitted",
      "set": { "topic": "Продажи" },
      "aliases": ["продажи"]
    }
  ]
}
```

Правила shorthand:

- текст обязателен;
- `{{name}}` означает ссылку на сохранённый business value `name`;
- compiler deterministically hoist-ит текст как `plain` под stable content key,
  вычисленным из source pointer, и регистрирует найденные placeholders;
- одинаковые source artifacts и CompileContext дают byte-identical package;
- button `run` ссылается на logical block;
- `set` применяется атомарно вместе с выбором;
- `aliases` — необязательные текстовые эквиваленты кнопки;
- actionKey, callback/event key и transition key генерирует compiler;
- если нужен внешний стабильный action contract, он задаётся через отдельный advanced/public event contract, а не случайным callback key.

Для `aliases` V1 использует `trim=true` и `ignoreCase=true` как фиксированную семантику.

Semantic actions — альтернативный strict button mode одного `send`:

```json
{
  "type": "send",
  "text": "Выберите действие",
  "buttons": [
    { "text": "Базовый", "action": { "type": "choice", "key": "basic", "run": "submitted", "save": "selected_offer" }, "aliases": ["базовый"] },
    { "text": "Условия", "action": { "type": "url", "url": "https://club.example/terms" } }
  ]
}
```

- `choice.key` — static business key; `run` — logical block; optional `save`
  записывает key в один canonical writable text DataField;
- `url` допускает только bounded absolute HTTPS URL без credentials/control chars;
- `request_contact` требует `input.type = telegram_contact` на том же block;
- `request_location` — только outbound V1, location capture пока отсутствует;
- inline `choice`/`url` нельзя смешивать с reply-keyboard
  `request_contact`/`request_location`;
- semantic list ограничен `1..8` buttons;
- `callback_data`, `reply_markup`, Telegram/provider JSON, authored action/runtime
  IDs и произвольные expressions запрещены.

Legacy и semantic buttons нельзя смешивать в одном `send`.

## 7. `set`

```json
{
  "type": "set",
  "values": {
    "topic": "Автоматизация",
    "attempt": 1
  }
}
```

Значения — JSON literals, безопасные ссылки `ValueRefV1` либо bounded
`SafeTransformValueV1`. В одном `set` action используется ровно один режим:
literal values, field copies или transforms; это сохраняет однозначный lowering.

```json
{ "$from": "name" }
```

Объект ровно формы `{ "$from": "key" }` означает ссылку на ранее сохранённый business value.

SafeTransform остаётся explicit `set` value:

```json
{
  "type": "set",
  "values": {
    "display_name": {
      "$transform": {
        "operation": "concat",
        "args": [
          { "$from": "first_name" },
          " ",
          { "$from": "last_name" }
        ]
      }
    }
  }
}
```

V1 operation catalog: `concat`, `uppercase`, `lowercase`, `coalesce`,
`url_encode`, `base64`, `round`, `add`, `subtract`, `multiply`, `divide`,
`format_date`, `timestamp`. Args — только scalar literal или exact `ValueRefV1`;
nested transform/object/array запрещён. Compiler проверяет registered operation,
bounded arity, argument/output DataField types и literal options. Runtime читает
pre-transition values, не выполняет coercion и возвращает deterministic error на
missing/null (кроме `coalesce`), divide-by-zero, invalid encoding, non-finite или
invalid target result. Запись появляется только после успешной evaluation всего
transform set.

`timestamp` — Unix seconds из explicit datetime argument; для текущего времени
используется `{ "$from": "current_datetime" }`, то есть event-derived system
DataField, а не wall clock. `format_date` поддерживает `iso_date`, `ru_date` и
`iso_datetime`; locale/timezone templates не входят в V1. `default` представлен
одной canonical operation `coalesce`. HMAC/signature и другие credential-dependent
operations deferred до typed credential/resource contract.

Произвольные expressions/eval/JS/Python/SQL, dynamic names и side effects запрещены.

### `unset`

```json
{ "type": "unset", "fields": ["temporary_value"] }
```

На strict Data Field path каждый key должен быть объявлен writable `flow` или
`user` field. Операция удаляет значение в missing и не подменяется `null`, `""`,
`0` или `false`. Dynamic field names и generic bot/system unset запрещены.

## 8. `track`

```json
{
  "type": "track",
  "event": "consultation_submitted",
  "properties": {
    "topic": { "$from": "topic" }
  }
}
```

AI задаёт business event name и business properties.

Compiler создаёт `RECORD_BUSINESS_EVENT`, effect key, property bindings и analytics capability requirement.

## 9. `schedule`

```json
{
  "type": "schedule",
  "id": "consultation_followup",
  "after": "30m",
  "run": "followup",
  "replace": true
}
```

Правила:

- ровно одно из `after` или `at`;
- `after` V1: положительное целое + `s|m|h|d`, максимум 365 дней;
- `at` — RFC3339 datetime;
- `run` указывает logical block;
- `id` обязателен, если schedule позже отменяется/заменяется; иначе может быть compiler-generated;
- `replace:true` компилируется через существующую cancel + reschedule semantics, а не новый runtime primitive;
- physical `scheduleKey`, internal event и generation не авторятся AI.

## 10. `cancelSchedule`

```json
{
  "type": "cancelSchedule",
  "id": "consultation_followup"
}
```

V1 отменяет только логически именованный schedule текущего flow instance.

`all:true` не вводится в BotSpec V1: controlled repeated START уже использует runtime user-restart semantics и отменяет schedules текущего завершаемого flow.

## 11. `call`

```json
{
  "type": "call",
  "operation": "CREATE_ASSET_ORDER",
  "input": {
    "amount": "100.00",
    "currency": "RUB"
  }
}
```

`operation` должна существовать в публичном capability/effect catalog.

Compiler:

- проверяет публичную operation;
- создаёт typed effect;
- выводит capability requirement;
- преобразует `ValueRefV1` в допустимые bindings.

AI не может объявить новую operation внутри BotSpec.

HTTP-1A регистрирует единственную controlled HTTP operation:

```json
{
  "httpResources": {
    "crm_profile": {
      "endpoint": "https://crm.example.com/v1/profile",
      "method": "GET",
      "idempotency": "none",
      "credential": { "reference": "crm_api_token", "mode": "bearer" },
      "headers": { "accept": "application/json" },
      "request": {
        "query": [{ "name": "phone", "field": "customer_phone" }]
      },
      "response": {
        "contentType": "application/json",
        "mappings": [
          { "path": "/customer/id", "field": "crm_customer_id", "type": "text" }
        ]
      }
    }
  },
  "blocks": {
    "load_profile": {
      "actions": [{
        "type": "call",
        "operation": "HTTP_REQUEST",
        "input": { "resourceKey": "crm_profile" }
      }]
    }
  }
}
```

Resource/call rules:

- resource identity and `credential.reference` are static logical keys;
- endpoint is absolute public `https:` without credentials, fragment, direct IP,
  local hostname, control characters or unsupported port;
- methods are exactly `GET` and `POST`; GET uses `idempotency: "none"`, POST
  requires `idempotency: "provider_key"`;
- static headers are limited to `accept` and `content-type`; authorization,
  cookies, forwarding/proxy/host headers and literal secrets are forbidden;
- request mappings are bounded flat name → canonical DataField reads, not
  templates; response is fixed `application/json` with bounded named-property
  JSON-pointer paths and exact target DataField types;
- response targets must be writable `flow`/`user` fields; duplicate paths or
  writers, unknown fields, coercion, dynamic resource keys, raw request JSON,
  provider IDs and arbitrary expressions fail closed;
- HTTP-1A compiles static contract only. Network/DNS/SecretStore execution is
  not an authoring operation and is not implemented by this checkpoint.

## 12. Input

V1 поддерживает свободный текст:

```json
{
  "type": "text",
  "save": "name",
  "trim": true,
  "required": true,
  "run": "choose_topic"
}
```

Compiler создаёт `TEXT_RECEIVED`, `textInputCondition`, context binding и transition wiring.

`input.type = "text"` является generic text capture. В одном state он может сочетаться с `on.text` и button `aliases`: specific text matches проверяются раньше input. Если specific route совпал, input не сохраняет текст и не выполняется; если не совпал, input обрабатывает событие. Overlap только между specific matchers (`on.text ↔ on.text`, `alias ↔ alias`, `alias ↔ on.text`) fail closed.

V1 не обещает number/date/email parser, пока такие deterministic authoring contracts не будут отдельно приняты.

`trim` и `required` обязательны как явные boolean — compiler не угадывает их из текста ТЗ.

## 13. `on` handlers

Нужны для публичных событий и фиксированного text matching.

### Public/module event

```json
{
  "event": "PAYMENT_SUCCEEDED",
  "run": "delivery_pending"
}
```

### Text match

```json
{
  "text": {
    "equalsAny": ["Консультация"],
    "trim": true,
    "ignoreCase": true
  },
  "run": "consultation"
}
```

Ровно один trigger: `event` или `text`.

Public event должен существовать в canonical `PUBLIC_AUTHORING_EVENT_TYPES_V1` owning-модуля `bot-platform`. В V1 эта surface содержит только реально поддерживаемые module result events.

Low-level `CODE_OWNED_EVENT_TYPES` не является authoring registry. Platform/runtime input events `START`, `TEXT_RECEIVED`, `COMMAND_RECEIVED`, `UNKNOWN_INPUT` запрещены в `on.event`: START выражается через `entry`, свободный текст — через `input`/`on.text`, а технические button/schedule events compiler генерирует сам.

## 14. Fallback

```json
{
  "fallback": {
    "run": "fallback_message"
  }
}
```

`run` может вести в stateful или transient block.

Compiler создаёт low-level fallback rule.

## 15. `next` для transient block

```json
{
  "saveState": false,
  "actions": [
    { "type": "send", "text": "Служебное сообщение" }
  ],
  "next": "another_block"
}
```

`next` разрешён только для `saveState:false`.

Если `next` не указан, после действий пользователь остаётся в текущем runtime state.

Compiler обязан обнаруживать циклы, состоящие только из transient blocks, и отклонять их как неразворачиваемые.

## 16. Entry и START

`entry` — logical block, запускаемый на `START`.

Compiler создаёт необходимые внутренние initial state/START wiring.

Repeated START использует существующую controlled restart semantics SayDone:

- старый active flow завершается как user restart;
- его active schedules отменяются;
- новый flow стартует на текущей deployed version.

BotSpec V1 не переопределяет эту runtime policy.

## 16.1 RuleSets / Decision Tables

`ruleSets` — статический source catalog; `applyRuleSet` — pure data action:

```json
{
  "ruleSets": {
    "lead_segment": {
      "mode": "first_match",
      "resultField": "segment",
      "rules": [
        {
          "key": "vip",
          "when": { "field": "score", "op": "greaterOrEqual", "value": 90 },
          "value": "vip"
        },
        {
          "key": "warm",
          "when": { "field": "score", "op": "greaterOrEqual", "value": 50 },
          "value": "warm"
        }
      ],
      "otherwise": { "type": "value", "value": "cold" }
    }
  },
  "blocks": {
    "classify": {
      "saveState": false,
      "actions": [{ "type": "applyRuleSet", "ruleSet": "lead_segment" }],
      "next": "done"
    }
  }
}
```

Реализованные правила V1:

- RuleSet key и каждый `rules[].key` — static canonical business keys, а не
  runtime/generated IDs; AI source не содержит physical state/rule IDs;
- `mode` строго `first_match`; `rules` содержит `1..64` entries и сохраняет
  source order; top-level catalog ограничен `64` RuleSets;
- `when` использует ровно существующий pure Conditions AST и canonical
  DataFields; второй predicate/evaluation language не создаётся;
- `resultField` — один declared writable `flow`/`user` DataField; каждое
  `value` — scalar literal того же type/constraints, без coercion, `$from`,
  transform, formula или dynamic field name;
- `otherwise` обязателен: `{ "type": "value", "value": ... }` создаёт
  deterministic default, `{ "type": "no_match" }` не изменяет target;
- `applyRuleSet.ruleSet` — static literal reference; transition содержит не
  более `8` уникальных applications. Они читают pre-transition DataField view,
  затем результаты более ранних RuleSet applications;
- другой writer того же result field в transition запрещён; side effects,
  actions, HTTP/DB/module calls и transport operations внутри RuleSet запрещены;
- compiler lowering создаёт immutable package `definition.ruleSets` и static
  `ruleSetApplications`; reducer использует existing Conditions evaluator;
- SourceMap отражает RuleSet, ordered rule и application source paths.

Source acceptance не ссылается на package IDs: ожидаемый результат проверяется
обычным `expectContext` по canonical DataField.

## 16.2 Static named subflows

Опциональный `subflows` композирует reusable source flows в тот же immutable
`BotPackageV1`:

```json
{
  "blocks": {
    "collect": {
      "call": { "subflow": "collect_phone", "then": "after" }
    },
    "after": { "actions": [{ "type": "send", "text": "Сохранено" }] }
  },
  "subflows": {
    "collect_phone": {
      "inputs": ["customer_name"],
      "outputs": ["phone"],
      "entry": "ask",
      "result": "done",
      "blocks": {
        "ask": { "input": { "type": "phone", "save": "phone", "run": "confirm" } },
        "confirm": { "on": [{ "event": "CLOSED_CLUB_PURCHASE_INITIATED", "run": "done" }] },
        "done": {}
      }
    }
  }
}
```

Правила реализованного V1:

- subflow key статический, canonical `snake_case`, не runtime value;
- `inputs` и `outputs` содержат только существующие canonical
  `DataFieldCatalogV1` keys; output может быть только `flow` или `user` scope;
- `entry` указывает обычный stateful block, `result` — отдельный пустой terminal
  marker;
- call block содержит только `call.subflow` и `call.then`; `then` указывает
  обычный stateful block caller flow;
- call marker вызывается напрямую из event/fallback/button/schedule route;
  typed-input capture не совмещает call/return, а call button не совмещает
  field mutation;
- compiler строит полный static call graph, отклоняет unknown target,
  direct/indirect recursion и глубину более `8`;
- каждый subflow lowering создаёт existing `BotDefinitionV1.flows` entry;
  dynamic flow names и второй workflow engine не создаются;
- compiler-owned bounded continuation хранится в существующем flow
  `context_json`, переживает restart и удаляется после return; это не DataField;
- call, return, flow и block сохраняются в SourceMap. Source acceptance может
  адресовать block через optional `subflow`, никогда через generated flow/state id.

Continuation, flow/user mutations и effect outbox сохраняются существующей одной
flow transaction. Migration не требуется: `flow_key`, `state_key` и
`context_json` уже являются durable snapshot seam.

## 17. Данные и ссылки

Business value key:

- непустой `snake_case`;
- не начинается с `_`;
- запрещены `__proto__`, `prototype`, `constructor`.

Compiler обязан статически диагностировать `$from`, для которого нет ни одного возможного writer (`input.save` или `set`), если значение не объявлено внешним compile context.

Ссылки не поддерживают пути вида `user.profile.name` в V1. Вложенные business objects передаются литерально через module operation input, если это разрешает capability contract.

## 18. Fail-closed validation

Отклоняются:

- unknown properties;
- unknown action/input/handler types;
- duplicate block ids;
- missing block references;
- unknown/dynamic subflow target, invalid continuation/result;
- recursive или слишком глубокий static subflow graph;
- undeclared/read-only subflow DataField output;
- unknown/dynamic RuleSet reference, duplicate static identities, empty или
  oversized table/application list;
- missing `otherwise`, invalid result type/constraint и другой writer того же
  result field в transition;
- action/effect/arbitrary expression shape внутри RuleSet;
- transient block с `input/on/fallback`;
- stateful block с `next`;
- transient-only cycle;
- неизвестная public operation/event;
- invalid duration/date;
- schedule cancel reference без известного logical schedule id;
- конфликт reserved fields;
- недопустимые `$from` references;
- произвольные executable expressions.

Каждая ошибка указывает путь в BotSpec.

## 19. Детерминированность

Compilation stages:

```text
parse JSON
→ validate BotSpec
→ normalize
→ resolve logical graph
→ allocate generated keys
→ lower actions/input/handlers
→ derive content/capabilities
→ derive mechanical proof plan
→ assemble BotPackageV1
→ canonical inspect/validate
```

Один и тот же BotSpec + compile context + версия compiler contract дают одинаковый generated package.

## 20. Что не входит в V1

- YAML как канонический source;
- visual editor;
- произвольный JS;
- SQL;
- произвольный HTTP;
- runtime LLM;
- business-specific block types;
- custom backend handlers из текста;
- автоматическое расширение capability catalog;
- ручное управление runtime IDs;
- замена существующего BotPackage lifecycle.

## 21. Независимые доказательства

Compiler может автоматически вывести механические проверки для `trim`, `required`, `save`, `track`, schedules и state-preserving blocks.

Но business acceptance/qualification остаются независимым входом. Generated tests не могут сами по себе доказать, что AI правильно поняла человеческое ТЗ.

## 22. Historical implementation starting point

Этот раздел описывает стартовую последовательность B6.7-3 на 2026-08-09.
Foundation, subsequent lowering и compiler/tools уже реализованы и verified;
текущий product gap находится в unified loader из AUTHORING-1.

Implementation должен начинаться с:

```text
BotSpec parser/schema
→ fail-closed validator
→ normalized representation
→ structured diagnostics
```

а не сразу с полной генерации ботов.
