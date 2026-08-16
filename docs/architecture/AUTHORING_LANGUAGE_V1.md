# Bounded Authoring Language V1

Статус: `IMPLEMENTED / VERIFIED / CHECKPOINTED`

## 1. Boundary

`AUTHORING-LANGUAGE-1` расширяет declarative core маленькими universal
primitives. Он не вводит business-specific block taxonomy, expression engine,
visual canvas или executable user code.

## 2. Pure conditions

Initial predicates: `equals`, `notEquals`, `greaterThan`, `greaterOrEqual`,
`lessThan`, `lessOrEqual`, `contains`, `in`, `exists`, `isEmpty`; composition:
`all`, `any`, `not`.

```text
CONDITION IS PURE
```

Condition не выполняет HTTP, DB/module mutation, external side effect,
JS/eval. External decision path: capability call → DataField write → pure condition.

## 3. Typed input

Input — capture mechanism; DataField — canonical storage/business definition.
Один field может получать value из text input, Telegram contact,
HTTP/module result или controlled operator/provider source.

Initial capture/validation formats: `text`, `number`, `email`, `phone`,
`telegram_contact`. Future: date/time, location, file, image. Standard email/phone
не требуют AI-authored regex/IF. Реализованный contract bounded описывает
validation, retry, max attempts, invalid-input path и timeout `{ after, run }`.
`after` — positive duration не более 365 дней; compiler генерирует технический
schedule identity, отменяет его при valid reply и exhausted route, а timeout route
привязывает к исходному runtime state. Поэтому canceled, stale и duplicate timeout
не могут повторно перевести уже покинувший input state.

Valid typed rule проверяется до generic invalid catcher. При достижении
`maxAttempts` compiler-owned exhausted branch исполняет actions/mutations именно
`exhaustedRun`, а не retry branch; старый BotPackage shape без branch сохраняет
прежний совместимый `exhaustedTarget` behavior. Source acceptance выражает normalized contact
как `{ "type": "contact", "phone": "+79991234567" }` и lowering создаёт только
provider-neutral `CONTACT_RECEIVED` payload, без Telegram update/JSON в source.

## 4. Static named subflows

Large bot может статически компоновать named subflows с bounded
`inputs`, `outputs`, `entry`, `result`. Recursive subflows, arbitrary dynamic names
и arbitrary call stack в V1 не входят.

AL-1C реализует это как optional source `subflows` и structural call marker
`{ "call": { "subflow": "static_key", "then": "continuation" } }`. Каждый
subflow детерминированно lowering в existing `BotDefinitionV1.flows`; compiler
проверяет весь call graph, запрещает recursion и глубину более `8`. Explicit empty
`result` block возвращает в statically compiled continuation. Bounded continuation
metadata хранится в existing `context_json`, не является DataField и удаляется
после return. `inputs`/`outputs` ссылаются только на canonical Data Fields;
dynamic target, typed-input call/return fusion и второй runtime engine не введены.

## 5. Decision tables / Rule Sets

AL-1D реализует mandatory primitive под именем `RuleSetV1`: top-level source
`ruleSets` содержит статически именованные decision tables, а pure data action
`applyRuleSet` ссылается на них только literal key. Каждая таблица имеет `1..64`
ordered rules, static rule keys, existing Conditions AST и один literal result,
типизированный writable canonical `flow`/`user` DataField. Каталог ограничен
`64` таблицами, transition — `8` уникальными applications.

Единственный V1 mode — `first_match`: проверка прекращается на первой true
Condition. Обязательный `otherwise` задаёт либо typed `value`, либо explicit
`no_match`; `no_match` сохраняет текущее target value/missing без mutation.
Applications выполняются в authored order, читают pre-transition canonical
DataField view и результаты предшествующих applications. Другой writer того же
result field в transition запрещён как ambiguous. Reducer переиспользует один
`evaluateDeclarativeConditionV1`; второго Conditions engine нет.

RuleSet не содержит actions/effects, runtime IDs, dynamic names, ValueRef,
transforms или executable expressions. Это не SQL, JS/eval, arbitrary formula
DSL или side-effect engine. SourceMap сохраняет static RuleSet/rule/application
identities; acceptance остаётся source-level и проверяет результат через
canonical DataField expectations.

## 6. Safe transforms

AL-1E реализует SafeTransforms как значение только внутри explicit `set`:
`{ "$transform": { "operation": "...", "args": [...] } }`. Вызов плоский:
`args` содержат только scalar literals или exact `{ "$from": "field" }`.
Transform сам ничего не записывает; compiler lowering создаёт отдельный typed
binding, а reducer применяет вычисленный scalar к target DataField только как
результат исходной `set` operation.

V1 catalog: `concat`, `uppercase`, `lowercase`, `coalesce`, `url_encode`,
`base64`, `round`, `add`, `subtract`, `multiply`, `divide`, `format_date` и
`timestamp`. Arity ограничена `1..8` в зависимости от operation, `round`
precision — literal integer `0..6`, `format_date` format — literal `iso_date`,
`ru_date` или `iso_datetime`. `timestamp` возвращает Unix seconds и получает
datetime только из typed literal/DataField; обычный current-time path читает
system `current_datetime`, который DATA-1 выводит из `event.occurredAt`.

Все transforms читают один pre-transition canonical DataField view и не видят
результаты соседних `set`/transform. Input/output types проверяются compiler и
package validator без coercion; target constraints повторно проверяются runtime.
`missing` и `null` различны: только `coalesce` пропускает missing/null, но считает
`""`, `0` и `false` полноценными значениями; если значения нет, результат —
explicit `null`. Остальные operations fail closed на missing/null. Деление на
ноль, invalid Unicode для encoding, non-finite numeric result и target constraint
failure дают deterministic error до возврата transition result; частичный patch
не выдаётся.

Nested transforms, dynamic operation/field names, JS/eval/Python/SQL/formula DSL,
DB/network/provider/secret access, wall clock и random отсутствуют. `default`
не является вторым alias в V1: его semantics покрывает `coalesce`. HMAC/signature,
locale/timezone patterns и credential-dependent transforms отложены до отдельной
typed credential/resource boundary.

## 7. Message actions

AL-1F добавляет к existing `send.buttons` strict semantic `action` union:
`choice`, `url`, `request_contact`, `request_location`. Presentation `text`
остаётся в Content, immutable behavior объявляется Definition manifest и только
зеркалируется в content delivery projection. Legacy `{ text, run, set, aliases }`
сохранён без изменения; смешивать legacy и semantic buttons в одном `send`
нельзя.

`choice` содержит static business `key`, static `run` и optional `save` одного
canonical text DataField. Compiler генерирует transport event и existing runtime
route; source не содержит `actionKey`, callback data или runtime ID. `url`
принимает только absolute HTTPS URL длиной до 2048 символов, без credentials и
control characters. Tracked Links не входят в AL-1F.

`request_contact` разрешён только на block с existing `telegram_contact` input:
Telegram adapter создаёт provider keyboard, а existing `CONTACT_RECEIVED` path
записывает phone в canonical DataField. `request_location` в AL-1F — только
outbound request: Telegram keyboard поддержан, но location input/event/DataField
capture отсутствует, и inbound location update fail-closed. Future: `web_app`,
`share`, `payment`. Raw Telegram Bot API и provider JSON не являются authoring
contract.

## 8. Triggers and static usage graph

`TRIGGERS-1A` добавляет bounded high-level entrypoints без открытия raw runtime
events: existing `entry` задаёт ordinary `START`; optional `startPayloads` —
static exact tokens (`1..64`, alphanumeric/`_`/`-`) → block; `on.command` —
static lowercase logical command; keyword/text использует existing canonical
`on.text`. Start payload не захватывает DataField и не является parser/regex DSL;
command arguments не являются частью V1 source contract. `BotAcceptanceSpecV1`
выражает `{ type: "start", payload? }` и `{ type: "command", command }`, а
compiler единолично понижает их до internal events.

Порядок: payload `START` → ordinary `entry` `START`; для stateful text —
button alias/specific `on.text` → generic typed input → existing state fallback
→ flow/global fallback → unhandled. `fallback` уже является explicit unmatched
semantics; второй router/fallback primitive не вводится. `START`,
`TEXT_RECEIVED`, `COMMAND_RECEIVED`, `UNKNOWN_INPUT` остаются internal и
fail-closed в public `on.event`. SourceMap привязывает `/entry`, каждый
`/startPayloads/<token>`, command/text/fallback к отдельному generated rule.
Deep-link payload поддерживает campaign/referral/source/lead-magnet/analytics
semantics. Group/channel/member triggers остаются provider-specific capabilities.

Static references должны позволять SayDone строить usage graph для
field, content, HTTP resource, subflow, RuleSet, asset и Dataset/resource:
writers, readers, references, provenance, usage. Used resource нельзя
безопасно удалить; graph нужен safe delete, Patch validation, impact
analysis, technical UI и AI repair.

## 9. Final qualification boundary

AL-1G canonical fixture проходит source artifacts → compiler → BotPackageV1 →
inspect → validate → ScenarioV2 → coverage → qualification. Multi-flow coverage
использует flow-qualified state/rule identities и включает static subflow entry,
result/return, retry/exhaustion и timeout semantics; single-flow coverage keys
сохранены для compatibility. Representative proof покрывает `14/14` states,
`13/13` rules, `4/4` scenarios и `4/4` qualification requirements. Одинаковые
source artifacts и CompileContext дают byte-stable canonical package, а SourceMap
сохраняет source paths всех реализованных primitives.

## 10. Anti-patterns

Normal authoring primitives не включают giant Calculator/formula DSL,
arbitrary JS/Python/SQL, dynamic field names, scope shadowing, side effects in
conditions, raw provider IDs, database/runtime IDs в AI contracts, direct client
DB editing и visual canvas как canonical source.
