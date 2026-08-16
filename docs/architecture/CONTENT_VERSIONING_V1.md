# Версионирование контента V1

Статус: `ACCEPTED / CONTENT-1 IMPLEMENTED AND VERIFIED`
Дата: 2026-08-06

## 1. Назначение

Контент отделяется от структуры flow, чтобы тексты и медиа можно было обновлять без переписывания бизнес-логики.

При этом опубликованный бот не должен зависеть от изменяемых строк без версии.

```text
Bot Definition
→ объявляет content keys и contract

Content Release
→ содержит конкретные тексты, media и безопасное оформление

Deployment
→ фиксирует bot version + content release
```

В V1 контент создают и публикуют разработчик и AI. Заказчик не получает редактор текстов или структуры.

## 2. Что относится к контенту

Допустимые content data:

- текст сообщения;
- caption;
- фото, видео, документ и другие assets;
- форматирование;
- безопасные URL;
- подписи кнопок;
- локализованные отображаемые названия;
- шаблонные переменные;
- информационные FAQ items, если их контракт не меняет flow.

Не относится к свободно редактируемому контенту:

- event type callback;
- guard;
- target state;
- effect type;
- amount/currency order;
- club offer ID;
- provider secret;
- произвольный HTTP URL effect;
- SQL или JavaScript;
- правила membership;
- idempotency/retry policy.

## 3. Content Manifest

Каждая Bot Definition объявляет требуемый контент.

Концептуальный manifest:

```text
welcome
├── type: message
├── required variables: first_name optional
├── media: optional
├── reply markup contract: welcome_actions
└── fallback: required

payment_success
├── type: message
├── required variables: amount, currency
└── media: none
```

Manifest фиксирует:

- обязательные content keys;
- тип элемента;
- допустимые переменные;
- обязательность media;
- совместимый reply markup contract;
- ограничения длины/формата;
- доступные locales, если они поддерживаются.

Content release нельзя опубликовать, если manifest не выполнен.

## 4. Концептуальные сущности

### 4.1. `content_releases`

Неизменяемая опубликованная версия набора контента.

Предварительные поля:

```text
id
organization_id
bot_project_id
release_number
status
compatible_definition_from
compatible_definition_to optional
created_by
created_at
validated_at optional
published_at optional
retired_at optional
```

Статусы:

```text
draft
validated
testable
published
retired
```

### 4.2. `content_entries`

Версия конкретного content key внутри release.

Предварительные поля:

```text
id
content_release_id
content_key
locale
content_type
text_template optional
asset_id optional
reply_markup_key optional
settings_json optional
created_at
```

Уникальность концептуально:

```text
content_release_id + content_key + locale
```

### 4.3. `assets`

Логический media asset независимо от транспорта.

Предварительные поля:

```text
id
organization_id
asset_type
status
original_filename optional
mime_type optional
size_bytes optional
checksum optional
storage_reference optional
created_at
updated_at
```

Transport-specific Telegram `file_id` хранится в messaging `transport_asset_bindings`, а не в content entry.

### 4.4. `reply_markup_definitions`

Безопасное отображение кнопок, связанное с разрешёнными action keys.

Предварительная структура:

```text
reply_markup_key
rows
button label content
registered action key
optional signed reference template
```

Content может менять подпись кнопки, но не создавать незарегистрированный action/effect.

## 5. Связь с Bot Definition

Bot Definition хранит структуру и ожидаемые события.

```text
button action key
→ принадлежит definition/event catalog

button label
→ принадлежит content release
```

Пример:

```text
Definition:
BUY_CLICKED → creating_order

Content:
label = «Купить доступ»
```

Изменение подписи не требует новой Bot Definition. Изменение callback action или target state требует новой версии definition.

## 6. Deployment pinning

Production deployment фиксирует:

```text
bot_project_id
bot_version
content_release_id
transport_integration_id
environment
```

Runtime не выбирает «последний контент» автоматически.

Новая content release применяется только после validation/test/publish и обновления deployment.

Rollback выполняется переключением deployment на предыдущий совместимый release, а не ручным редактированием опубликованных строк.

## 7. Draft → publish

Базовый процесс:

```text
1. AI/developer создаёт draft content release.
2. Проверяются manifest keys и variables.
3. Проверяются media assets.
4. Генерируется preview.
5. Выполняется test send через test integration.
6. Проходят scenario tests для content-dependent paths.
7. Release получает published.
8. Deployment переключается на release.
```

Опубликованный release неизменяем.

Исправление даже одного текста создаёт новую release revision.

## 8. Переменные

Шаблон использует только зарегистрированные переменные.

Примеры:

```text
first_name
amount
currency
payment_url
access_until
support_contact
```

Для каждой переменной определяются:

- источник;
- тип;
- optional/required;
- escaping;
- допустимость персональных данных;
- fallback policy.

Content не может обращаться к произвольному пути объекта или выполнять выражение.

Отсутствие required variable является validation/render error, а не поводом отправить необработанный placeholder.

## 9. Кнопки и положение элементов

В V1 структура клавиатуры создаётся разработчиком/AI и публикуется с definition/content contract.

Будущая ограниченная функция заказчика может разрешить:

- менять подпись;
- скрывать заранее разрешённую необязательную кнопку;
- менять порядок заранее разрешённых информационных элементов;
- менять текст или media.

Она не должна разрешать:

- менять action key;
- направлять callback в другой state;
- добавлять произвольный effect;
- менять payment amount;
- обходить validation/publish.

Пользовательское редактирование считается отдельным будущим scope и не входит в V1.

## 10. FAQ

FAQ можно реализовать двумя способами:

### Статический flow content

Небольшой фиксированный FAQ хранится в content keys и кнопках конкретного bot version.

### Data-driven FAQ collection

Позднее может появиться organization-scoped коллекция:

```text
faq_items
├── question
├── answer content
├── sort order
├── status
└── content release
```

Data-driven FAQ допустим только после отдельного контракта поиска, навигации и versioning. V1 не требует универсальной CMS.

## 11. Media lifecycle

Asset проходит состояния:

```text
draft
validated
ready
archived
failed
```

Перед production:

- проверяется тип и размер;
- выполняется test upload/send;
- создаётся transport binding;
- подтверждается доступность через нужную integration.

Удаление asset, используемого опубликованной release, запрещено. Его можно архивировать после снятия всех активных deployment references.

## 12. Совместимость releases

Content release совместима с bot version, если:

- присутствуют все required keys;
- тип каждого key совпадает;
- required variables поддержаны;
- reply markup action keys зарегистрированы;
- media requirements выполнены;
- не нарушены transport limits;
- validation suite пройдена.

Нельзя подключать release только по совпадению названий файлов или ключей без schema validation.

## 13. Аналитика контента

Business analytics использует стабильные event/action keys, а не отображаемый текст.

Например:

```text
PLAN_SELECTED
```

остаётся тем же событием, даже если подпись кнопки изменилась с «Выбрать» на «Купить».

Это сохраняет сопоставимость аналитики между content releases.

## 14. Клиентская поверхность

В V1 заказчик не управляет content releases.

Будущая редактура, если будет подтверждена, должна иметь:

- отдельное permission;
- draft;
- preview;
- test send;
- validation;
- publish/approval;
- rollback;
- audit trail.

Прямое редактирование production content через Adminer запрещено.

## 15. Связи с модулями

### Bot Platform

Definition объявляет manifest; deployment pin-ит release.

### Messaging

Messaging рендерит entry и разрешает transport asset binding.

### Flow Runtime

Flow ссылается только на `content_key`, не хранит текст в state transition.

### Tenancy

Content release и asset принадлежат organization и bot project.

### Analytics

События используют стабильные keys; release ID может добавляться как dimension.

## 16. Инварианты

1. Опубликованный content release неизменяем.
2. Production deployment pin-ит конкретный release.
3. Content не меняет state transitions и effect types.
4. Все keys и variables проходят manifest validation.
5. Telegram file ID не хранится как универсальное поле content.
6. Удаление используемого asset запрещено.
7. Изменение текста не должно ломать аналитические event keys.
8. Клиент V1 не редактирует content/runtime.
9. Rollback использует предыдущую release.
10. Произвольный код в content не исполняется.

## 17. Что не входит в V1

- клиентский visual editor;
- совместное редактирование;
- полноценная headless CMS;
- сложная локализация;
- A/B-тестирование контента;
- dynamic expressions;
- AI-генерация текста заказчиком;
- произвольный HTML;
- data-driven каталоги любого типа;
- автоматическая публикация без review/test.

## 18. Уточнения реализации B5

- validated content artifact хранится в `content_releases.content_json`, metadata/checksums/report — в существующих колонках;
- compatibility связывает exact definition/content pair общим package checksum в `validation_report_json`;
- testable требует passing scenario evidence;
- publish выполняет internal platform operator после active test deployment;
- promotion и rollback создают historical pinned deployments, не меняя content JSON;
- customer content editing/publish остаётся вне V1.

## 19. Открытые вопросы после B5

1. Нужен ли отдельный content release для каждого bot version или допускается диапазон совместимости?
2. Какие форматы Markdown/HTML разрешены Telegram V1?
3. Хранится ли исходный media binary после получения Telegram file_id?
4. Нужен ли один locale `ru` в первом пилоте?
5. Какие кнопки считаются безопасно редактируемыми в будущем?
6. Нужен ли data-driven FAQ в первой версии closed-club?

Документ фиксирует версионирование, безопасность и storage implementation B5; будущая customer editing policy остаётся отдельным решением.

## 20. Accepted CONTENT-1 future direction

- `content_key` — stable editable content identity;
- compatible content-only change создаёт новый Content Release без rebuild Bot Definition;
- target formats: `plain`, safe Telegram subset `telegram_html`, `telegram_markdown_v2`;
- arbitrary browser HTML не допускается;
- validation transport-aware и до runtime проверяет formatting/escaping,
  media/button combinations, provider limits и Telegram composition restrictions;
- inline text допустим только как convenience/shorthand, если canonical
  compiler deterministic поднимает его в stable content identity;
- binary assets имеют separate lifecycle и logical references; Telegram `file_id` остаётся transport binding;
- AI объявляет logical asset slot (`promo_banner`, `kind=image`,
  `required=true`), а binary загружается отдельно через storage lifecycle;
- ordinary Authoring Bundle не хранит binary/base64;
- `SayDoneAuthoringBundleV1` не равен future `SayDoneProjectArchiveV1`,
  который может содержать manifest + JSON + binaries;
- future bounded business UI может редактировать safe content/media, но не technical flow identity/actions;
- customer content editor и эти formatting capabilities сейчас не реализованы.

## 21. Фактическая граница CONTENT-1A

`bot-authoring` поддерживает optional canonical `BotSpecV1.content`: static
logical `content_key`, один из `plain` / `telegram_html` /
`telegram_markdown_v2`, text и explicit required/optional variable registry.
Compiler остаётся единственным владельцем lowering/package assembly и формирует
существующие Definition Content Manifest и immutable content artifact без второго
lifecycle или persistence model.

Legacy `send.text` сохранён как shorthand. Его stable `content_*` key зависит от
source pointer, а не от текста или runtime ID: одинаковый source компилируется
byte-identically, а text-only edit меняет content artifact/checksum без изменения
Definition checksum. Explicit content key также не зависит от текста.

Strict source validation fail-closed отклоняет неизвестные content references,
unsupported format, malformed/overlapping variable declarations,
unregistered/unused required placeholders, binary/base64, provider IDs и
provider JSON. При наличии DATA-1 catalog variables дополнительно проверяются как
canonical DataFields. SourceMap сохраняет entry, format, variable declarations и
send/button source attribution. Direct legacy `BotPackageV1` без optional format
metadata остаётся совместимым.

CONTENT-1A не реализует renderer/escaping, arbitrary markup или Telegram
format/composition limits — это граница CONTENT-1B. Asset slots, binary lifecycle
и расширенная release/pinning qualification не начаты и относятся к CONTENT-1C.
Existing published Content Release immutability, checksums, deployment pinning и
rollback не изменялись.

## 22. Фактическая граница CONTENT-1B

Canonical renderer находится в existing `messaging` seam и является pure typed
operation: content entry + manifest variable contract + flat scalar values дают
rendered content либо bounded failure. Object paths, nested values и expression
evaluation отсутствуют. Required missing/null — failure; optional missing/null —
empty string; `0`, `false` и empty string не определяются через truthiness.
Если итоговый message empty, rendering fail-closed.

Static authored markup сначала validates как trusted bounded template, а dynamic
values экранируются по format:

- `plain` — без markup semantics;
- `telegram_html` — properly nested attribute-free subset `b/strong`, `i/em`,
  `u/ins`, `s/strike/del`, `code`, `pre`, `blockquote`, `tg-spoiler`; только
  named entities `amp/lt/gt/quot`;
- `telegram_markdown_v2` — escaped reserved literals и sequential non-nested
  `*`, `_`, `__`, `~`, `||`, backtick spans; dynamic values внутри code span
  запрещены.

Authoring и package validation используют тот же pure format analyzer, поэтому
unsupported tags/attributes/entities, malformed/crossing spans, object-path
placeholder и control characters отклоняются до runtime. Existing Telegram
boundary дополнительно проверяет rendered non-empty text, `4096` code-unit bound,
button label `1..64`, keyboard total `<=100`, semantic total `1..8`, exact action
shape и inline/reply incompatibility до provider call. Mapping в `HTML` /
`MarkdownV2` `parse_mode` существует только в Telegram adapter.

Outbox SEND_CONTENT берёт variable contract из pinned immutable Definition.
Render failure terminal и происходит до `outgoing_messages`, delivery attempt и
transport call, поэтому raw placeholder не отправляется и не сохраняется как
rendered outgoing payload. Legacy direct entries без format используют `plain`;
legacy helper сохраняет flat case-insensitive placeholder convenience, но не
разрешает object paths или non-scalar values.

CONTENT-1B не добавляет asset lifecycle, upload/storage, `file_id` binding или
release/pinning qualification; это остаётся CONTENT-1C.

## 23. Фактическая граница CONTENT-1C

`BotSpecV1.assets` объявляет static logical slots вида `key + kind + required`;
content entry ссылается только на `asset` key. Canonical compiler переносит
контракт в `ContentManifestV1.assetSlots`, reference — в manifest/content
artifact `assetKey`, а SourceMap сохраняет обе source identities. Поддерживаются
ровно DB-aligned kinds `image`, `video`, `audio`, `document`, `animation`.

Lifecycle validation принимает internal operator mapping logical slot →
provider-neutral `assets.id`, проверяет organization, `ready` и exact kind.
Unresolved required slot fail-closed; unresolved optional slot допустим. Этот
mapping не является ordinary authoring source. Binary/base64, provider JSON,
storage URL и Telegram `file_id` там запрещены. `provider_file_id` остаётся
только integration-scoped записью existing `transport_asset_bindings`; CONTENT-1
не реализует upload UI, binary storage redesign или Telegram media transfer.

Existing lifecycle теперь допускает compatible content-only release при
неизменном Definition checksum: published Bot Version переиспользуется, новый
Content Release независимо проходит validate → Scenario evidence → testable →
publish. Published JSON/checksum не изменяются. Test/production deployments и
new flow instances pin-ят exact content release; runtime не выбирает maximum/
latest release. Rollback создаёт новую production deployment на предыдущую
published compatible release и не переписывает historical flow pins.

Migration не добавлялась: `content_releases.content_json`, existing lifecycle
metadata, `bot_deployments`, `flow_instances.content_release_id`, `assets` и
`transport_asset_bindings` достаточны для этой bounded границы.
