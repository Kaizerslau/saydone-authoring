# Операции и поток управления SayDone Language

Общие операции не получают provider-prefixed вариантов:

<!-- c2-operation-signatures:start -->
- `analytics.track(event: ..., [properties: ...], [fields: ...])`;
- `billing.checkout(paymentKey: ..., paymentAccountKey: ..., checkoutType: ..., description: ..., amount: ..., currency: ..., [buyerEmail: ...], [receiptItemName: ...], [receiptVatCode: ...], [receiptPaymentMode: ...], [receiptPaymentSubject: ...], [receiptMeasure: ...])`;
- `message.send(text: ... | content: ..., [chatId: ...], [buttonLabel: ...], [buttonAction: ...])`;
- `telegram.sendPhoto(fileId: ...)`;
- `telegram.sendVideo(fileId: ...)`;
- `telegram.sendVideoNote(fileId: ...)`;
- `data.set(field: ..., value: ...)`;
- `data.unset(field: ...)`;
- `rules.apply(ruleSet: ...)`;
- `http.request(resource: ..., result: ...)`;
- `http.save(field: ..., path: ..., result: ...)`;
- `schedule.create(id: ..., after: ... | at: ..., target: ..., [replace: ...])`;
- `schedule.cancel(id: ...)`;
- `subflow.call(subflow: ..., then: ..., [inputMappings: ...], [returnMappings: ...])`;
<!-- c2-operation-signatures:end -->

Квадратные скобки здесь обозначают необязательный аргумент, `|` — выбор ровно
одного аргумента. Это условная запись сигнатуры, а не синтаксис исходника.
Список обновляется командой `pnpm exec tsx scripts/generate-c2-operation-signatures.ts`.

Это изолированные формы statement, а не самостоятельный документ SayDone Language. Каждый
named call записывается на одной строке; полные production-valid examples есть
в [examples](./examples/).

Операции проверяются по exact именам аргументов, типам, references и допустимому
контексту. Compiler принимает `C2ValidatedSemanticRevisionV2`, где raw source text
отсутствует, и переиспользует существующий BotSpec compiler, DataFields,
RuleSets, HTTP, schedules, subflows, effects и `BotPackageV1`.

У `billing.checkout` ключи identity (`paymentKey`, `paymentAccountKey`,
`checkoutType`, `currency`) всегда статичны. `description`, `amount`,
`buyerEmail` и поля одной фискальной позиции принимают literal либо явную ссылку
`$data_field`. Это существующая DataField binding-модель, а не выражения:
арифметика, interpolation и provider-prefixed аргументы не поддерживаются.
Отсутствующее или некорректное значение завершает конкретный checkout до вызова
provider. Пример двух продуктов с одним checkout —
[dynamic-checkout.c2](./examples/dynamic-checkout.c2).

Resolved purchase values материализуются в immutable snapshot exact checkout
operation: retry и recovery этой operation не перечитывают DataFields.
`hosted_checkout` получает URL только из trusted `CHECKOUT_CREATED` lineage, а
`PAYMENT_SUCCEEDED` продолжает Flow только после verified payment event с exact
`paymentKey`. C2 не принимает provider credentials, произвольный checkout URL,
client-controlled payment authority, арифметику или другие денежные выражения.

Публично проверенный end-to-end authoring path использует
`checkoutType: one_time`. One-time checkout не является recurring subscription:
этот набор не заявляет Prodamus recurring contract/callbacks, полноценную
Closed Club purchase composition, Telegram join-request approval/removal,
entitlement-aware reminders либо automatic expiry/grace/revoke. Для таких
требований нельзя придумывать C2 syntax или считать успешную разовую оплату
доказательством subscription lifecycle.

`input.capture(...) => local_block`, `on.text(...) => local_block` и
`message.button(...) => local_block` — структурный ввод, exact text и
модификатор последнего сообщения, а не новые effect-producing операции.
Их сигнатуры и ограничения — в [LANGUAGE](LANGUAGE.md#базовый-диалог).

`telegram.sendPhoto`, `telegram.sendVideo` и `telegram.sendVideoNote` —
единственные provider-specific отправки в этом слое. Каждая принимает literal
Telegram `file_id` длиной 1–1024 без control characters; это не URL, не
SayDone asset и не upload/download mechanism. Compiler помещает значение в
обычный durable `SEND_CONTENT` outbox с уже pinned destination, retry и
idempotency. Необходима server-resolved Telegram capability
`telegram.file_id_media`; отсутствие capability завершается fail-closed.
Пример — [telegram-file-id-media.c2](examples/telegram-file-id-media.c2).

`next local_block` и `on.event`/`on.command` задают локальный control flow.
`route other_funnel.block` — структурный cross-funnel transfer существующей
composition boundary. Если standalone compilation не имеет exact composition,
compiler возвращает `c2.compile.internal_route_requires_composition`, а не
угадывает target или создаёт второй runtime.

`saveState:true` используется по умолчанию и допускает durable position.
`saveState:false` запускает transient chain; если она не достигает stateful
block, CN-1D завершает invocation без искусственного active flow.
