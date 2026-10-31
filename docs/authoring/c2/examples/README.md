# Проверяемые примеры SayDone Language

Эти файлы являются current copy-ready примерами SayDone Language. Каждый полный документ
проходит production parser и семантическую проверку; examples с provider fact
дополнительно используют текущий зарегистрированный registry seam.

- [start.c2](./start.c2) — минимальный provider-neutral сценарий нормализованного `/start`;
- [minimal.c2](./minimal.c2) — минимальный source и `message.send`;
- [survey.c2](./survey.c2) — имя/возраст, invalid number, подтверждение, Да/Нет, restart;
- [menu.c2](./menu.c2) — несколько обычных кнопок и возврат;
- [lead-form.c2](./lead-form.c2) — name/email/phone, валидация, attempts/timeout;
- [branching.c2](./branching.c2) — exact text choices, свободный текст и DataFields;
- [analytics.c2](./analytics.c2) — два явных события в отдельных переходах: `start`
  фиксирует оценку 10, `continue` — 20. Пример проверяется компиляцией;
  сохранение фактов и повторы проверяются отдельно на PostgreSQL.
- [core-flow.c2](./core-flow.c2) — DataFields, rules, HTTP, schedule и subflow;
- [telegram-route.c2](./telegram-route.c2) — provider-aware external route
  текущим однострочным named call.
- [multi-funnel-routing.c2](./multi-funnel-routing.c2) — два логических раздела Funnel в одном Project C2,
  exact external routes и явный cross-funnel `route` через transient block.
- [hosted-checkout.c2](./hosted-checkout.c2) — `billing.checkout`, одно
  `hosted_checkout` действие и exact `PAYMENT_SUCCEEDED` continuation. Это
  authoring contract; URL и payment data выдаёт trusted runtime lineage.
- [dynamic-checkout.c2](./dynamic-checkout.c2) — две продуктовые ветки BOT-A,
  один общий checkout и explicit DataField bindings для суммы, описания, email
  покупателя и фискальной позиции.
- [telegram-update-capture.c2](./telegram-update-capture.c2) — ожидание
  следующего Telegram Update и безопасный echo сохранённого raw JSON
  через обычный content/DataField путь.
- [telegram-file-id-media.c2](./telegram-file-id-media.c2) — три узких
  Telegram file_id media action без asset/upload/download.
- [telegram-inline-keyboards.c2](./telegram-inline-keyboards.c2) — 50 C2
  choice-кнопок с неодинаковой шириной rows; первые четыре rows образуют
  готовую 16-button раскладку.
- [telegram-direct-chat-id.c2](./telegram-direct-chat-id.c2) — `message.send`
  в literal либо DataField Telegram `chatId`.

Во всех examples named calls находятся на одной строке. Они не содержат
credentials, runtime IDs или provider payloads.
