# Контракт преобразования BotSpecV1 → BotPackageV1

Статус: `ACCEPTED / IMPLEMENTED AND VERIFIED BY B6.7 COMPILER`
Дата: 2026-08-09

## 1. Назначение

Этот документ фиксирует низкоуровневую семантику модуля `bot-authoring`.

```text
BotSpecV1
+ CompileContextV1
→ deterministic compilation
→ BotPackageV1
+ SourceMapV1
+ MechanicalProofPlanV1
```

Compiler не является LLM и не принимает бизнес-решений.

DATA-1 добавляет additive strict seam:

```text
BotSpecV1 + DataFieldCatalogV1 + BotAcceptanceSpecV1? + CompileContextV1
→ compileBotArtifactsWithDataFieldsV1
→ canonical BotPackageV1 with BotDefinitionV1.dataFields
```

Legacy `compileBotSpecV1` и `compileBotSpecWithAcceptanceV1` сохраняют историческую
implicit-context compatibility. Только explicit DataField-aware seam запрещает
undeclared keys и валидирует `input.save`, `set`/`unset`, `$from`, buttons,
track/call references, content placeholders и acceptance context assertions.
`apps/authoring-tools` только передаёт raw artifacts и не дописывает manifest.
Manifest участвует в существующем canonical definition/package checksum.

## 2. Compile context

Identity/lifecycle данные не должны случайно генерироваться AI.

Концептуальный `CompileContextV1` содержит:

```text
botKey
botVersion
contentReleaseNumber
generatedBy
sourceCommitSha? (если доступен operator tooling)
```

`botKey` поступает из project/organization context.

`botVersion` и `contentReleaseNumber` назначаются controlled authoring/lifecycle process.

BotSpec не может переопределить эти значения.

## 3. Generated flow

BotSpecV1 компилируется минимум в один generated flow.

Для V1 основной logical graph получает flow key:

```text
main
```

Compiler создаёт внутренний initial state, необходимый для обработки `START`. AI его не видит.

Если позднее BotSpec получит явную multi-flow authoring форму, это добавляется совместимым расширением contract. B6.7-3 не должен придумывать её скрыто.

## 4. Stateful и transient blocks

### `saveState:true`

Compiler создаёт runtime state.

`input`, `on`, `fallback` становятся handlers этого state.

Actions блока выполняются **при входе в блок**: compiler добавляет их в каждый generated transition/handler, который приводит к этому блоку. AI не обязано использовать `entryEffects` напрямую.

### `saveState:false`

Runtime state не создаётся.

При `run` такого блока compiler:

1. встраивает его data changes/actions в текущий generated transition;
2. если есть `next`, продолжает разворачивание следующего блока;
3. останавливается на первом stateful block либо на transient block без `next`;
4. без итогового stateful target создаёт targetless transition, поэтому текущая пользовательская позиция сохраняется.

Transient-only cycle является ошибкой BotSpec.

## 5. Порядок data changes и effects

Текущий `TransitionRuleV1` применяет context/cancel semantics отдельно от ordered effects. Поэтому BotSpec V1 не имеет права изображать произвольное чередование этих фаз.

Нормативный порядок generated transition:

```text
1. set / unset / input binding
2. cancelSchedule + implicit replace cancellation
3. ordered effects:
   send / track / schedule / call
4. target state
```

В `actions[]`:

- `set` и явные `cancelSchedule` обязаны находиться до первого effect-producing action;
- после `send|track|schedule|call` новый `set`/`cancelSchedule` запрещён validator;
- порядок между `send|track|schedule|call` сохраняется точно.

`replace:true` у `schedule` является специальной атомарной shorthand: compiler добавляет отмену того же logical schedule slot в фазу 2 и новый schedule effect в позицию этой action в фазе 3.

## 6. Generated keys

AI не авторит технические keys. Они генерируются из source path детерминированно.

### Source identity

Для каждой generated сущности compiler имеет canonical source pointer, например:

```text
/blocks/choose_topic/actions/0/buttons/1
```

Хеш:

```text
SHA-256(UTF-8 sourcePointer)
```

Для короткого suffix используются первые 10 hex символов. Это правило является частью compiler contract V1.

### State key

```text
state_<normalizedBlockId>_<hash10>
```

### Rule key

```text
rule_<kind>_<hash10>
```

### Effect key

```text
effect_<kind>_<hash10>
```

### Content key

```text
content_<hash10>
```

### Transport action key

Должен соответствовать существующему transport-safe contract:

```text
^[A-Z][A-Z0-9_]{0,63}$
```

Форма V1:

```text
A_<NORMALIZED_BLOCK_PREFIX>_<HASH10>
```

`NORMALIZED_BLOCK_PREFIX` — uppercase `[A-Z0-9_]`, максимум 40 символов. Если после нормализации первый символ не буква, добавляется `B_`.

### Scheduled internal event

```text
S_<NORMALIZED_SCHEDULE_PREFIX>_<HASH10>
```

с тем же transport-safe ограничением.

### Collision rule

После генерации всех keys compiler проверяет уникальность. Любая collision — hard compilation error; compiler не добавляет случайный suffix.

## 7. Content lowering

Каждый `send` создаёт content entry.

Compiler:

1. создаёт deterministic `contentKey`;
2. переносит text;
3. сканирует `{{variable}}` placeholders;
4. создаёт manifest variable declarations;
5. создаёт `variableBindings` для сохранённых business values;
6. для legacy/`choice` создаёт buttons/action keys; `url` и request actions не
   создают flow event;
7. создаёт соответствующие handlers на state, в котором legacy/`choice` кнопка
   будет показана;
8. добавляет `messaging.send_content` capability.

Один и тот же source `send` может быть встроен в несколько incoming transitions, но source map связывает все generated effects с одним BotSpec path.

## 8. Input lowering

`input.type = text` создаёт `TEXT_RECEIVED` transition.

- `required:true` → `textInputCondition { transform: trim, condition: non_empty }`;
- `trim:true` → context binding с transform `trim`;
- `trim:false` → binding без transform;
- `save` → context binding target key;
- `run` → resolved transient/stateful chain.

Важно: `required:true` означает «после trim значение не пустое». Это правило проверки одинаково даже при `trim:false`; `trim` управляет сохранённым значением, а `required` — критерием содержательности.

По D-054 compiler после всех presentation/transient expansion passes детерминированно располагает specific `TEXT_RECEIVED` rules (`on.text`, generated aliases) перед generic input rule. Public events, button action event types и fallback не участвуют в этой сортировке. Specific matcher overlap отклоняется canonical `doPayloadMatchesOverlap`; input не считается ambiguity с specific matcher.

## 9. Button lowering

Button создаёт:

```text
content button
→ generated actionKey
→ generated event handler
→ optional contextPatch from set
→ resolved run chain
```

`aliases[]` дополнительно создают `TEXT_RECEIVED` rules с:

```text
payloadMatch.field = text
trim = true
ignoreCase = true
```

Semantic `choice` использует тот же generated event/handler path, но definition
manifest хранит immutable `{ type, choiceKey, eventType }`, а optional source
`save` lowering в atomic context/DataField patch. `url`, `request_contact` и
`request_location` остаются provider-neutral content controls без flow handler.
SourceMap содержит отдельную `semantic_message_action` identity; только `choice`
дополнительно получает `transport_action`. Telegram `callback_data`, inline/reply
keyboard JSON и provider calls возникают только в telegram-adapter.

Alias overlap должен обнаруживаться до package output и дополнительно остаётся защищён canonical package validator.

## 10. Analytics lowering

`track` создаёт `RECORD_BUSINESS_EVENT`.

- literal properties → `properties`;
- `{ "$from": "key" }` → `propertyBindings`;
- capability `analytics.record_event` выводится автоматически.

## 11. Schedule lowering

`after` преобразуется в `delayMs` строго по единицам `s|m|h|d`.

`at` передаётся как normalized RFC3339 `runAt`.

Compiler создаёт:

- physical schedule key;
- internal scheduled event type;
- handler, который `run`-ит указанный logical block;
- `jobs.schedule_event` capability.

`replace:true` использует существующую cancel/reschedule semantics.

Событие schedule по умолчанию является compiler-owned internal event и не показывается AI.

## 12. Public event lowering

`on.event` не генерирует новый event identity.

Указанный event обязан существовать в canonical `PUBLIC_AUTHORING_EVENT_TYPES_V1` owning-модуля `bot-platform` и проходит обычную provenance validation. Этот authoring contract V1 содержит поддерживаемые module result events и не равен low-level `CODE_OWNED_EVENT_TYPES`.

`START`, `TEXT_RECEIVED`, `COMMAND_RECEIVED`, `UNKNOWN_INPUT` не принимаются из `on.event`: их semantics принадлежат compiler/runtime и выражаются только принятыми BotSpec-конструкциями.

Compiler создаёт только handler rule и resolved run chain.

## 13. Generic `call`

`call.operation` в V1 соответствует существующему public effect operation из capability catalog.

Compiler:

- проверяет operation;
- проверяет структуру input через canonical public effect contract;
- преобразует `$from` references в поддерживаемые bindings только если конкретный effect contract это позволяет;
- создаёт effect key;
- добавляет required capability.

Если существующий effect payload не поддерживает требуемую dynamic binding, compiler возвращает `UNSUPPORTED_PLATFORM_CAPABILITY`; он не внедряет шаблоны/JS.

## 14. Capability derivation

Итоговый `definition.capabilities` — sorted unique set, выводимый из generated effects.

AI не поддерживает список вручную.

Если effect не имеет зарегистрированного capability mapping, compilation/validation fail closed.

## 15. SourceMapV1

Source map — отдельное authoring evidence, не runtime dependency.

Для каждого BotSpec path хранится список generated references:

```text
sourcePath
→ generatedKind
→ generatedPath/key
```

Одна source entity может иметь несколько generated references.

Диагностика canonical package validator по возможности переводится обратно к ближайшему BotSpec source path.

Исходный BotPackage error при этом не скрывается из engineering evidence.

## 16. MechanicalProofPlanV1

Compiler может вывести обязательные механические доказательства:

- input `required`;
- input `trim`;
- `save`;
- button alias matching;
- `track`;
- schedule creation/replacement/cancel;
- transient/state-preserving execution.

Proof plan является входом для генерации Scenario V2 skeleton, но не заменяет independent qualification contract.

## 17. Compile result

Логический результат compilation:

```text
status: passed | failed | blocked
issues[]
package?             # только при passed
sourceMap?           # только при passed
mechanicalProofPlan? # только при passed
```

`failed` — неверный BotSpec/unsupported declarative combination.

`blocked` — tool/environment не способен выполнить operation корректно.

## 18. Диагностические классы

Минимальные namespaces:

```text
botspec.schema_*
botspec.reference_*
botspec.graph_*
botspec.action_*
botspec.input_*
botspec.schedule_*
botspec.capability_*
botspec.value_ref_*
compiler.key_*
compiler.lowering_*
compiler.source_map_*
```

Human message не является стабильным API. Стабильнее `code + path`.

## 19. Запрещённые зависимости

Compiler не использует:

- PostgreSQL;
- Telegram adapter;
- worker;
- provider SDK;
- production environment secrets;
- arbitrary network;
- LLM;
- runtime state.

Он зависит только от BotSpec contract и публичных package/capability/event/effect contracts.

## 20. Acceptance implementation

B6.7 compiler implementation считается соответствующей этому contract, если:

1. одинаковый input даёт byte-equivalent canonical package output;
2. generated keys соответствуют V1 algorithm;
3. transient/stateful semantics доказана tests;
4. existing package validator принимает generated package;
5. source mapping позволяет вернуть low-level issue к BotSpec path;
6. generated capabilities совпадают с реально используемыми effects;
7. direct BotPackage path продолжает работать;
8. runtime не импортирует bot-authoring.
