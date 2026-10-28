# SayDone Language (внутренний контракт: C2)

SayDone Language использует точный заголовок `language "saydone-r2/1"` и выбранную по
`DEC-0132` построчную грамматику. Parser сохраняет source spans, строки,
колонки, declaration structure и logical references. Семантическая модель не зависит
от пробелов, комментариев и форматирования.

Верхний уровень поддерживает `registry`, `project`, `field`, `content`, `http`,
`ruleset`, `funnel`, `subflow`, `provider` и `external.route`. Внутри `funnel`
и `subflow` находятся `block`; внутри блока — зарегистрированные операции,
`saveState`, `next`, `route`, `on.event`, `on.command`, `on.text`,
`input.capture` и модификаторы `message.button`.

Минимальный исходник:

```text
language "saydone-r2/1"
registry "r2-oracle-1"
project hello(name: "Приветствие", locale: "ru")
content greeting(text: "Здравствуйте", required: [])

funnel main {
  block greeting {
    message.send(content: greeting)
  }
}
```

К сообщению можно добавить одну provider-neutral кнопку hosted checkout:

```text
message.send(content: checkout_ready, buttonLabel: "Оплатить", buttonAction: hosted_checkout)
```

Optional `chatId` направляет только этот `message.send` в Telegram private/group/channel
ID через подключение текущего Project. ID записывается строкой, включая отрицательные
group/supergroup ID и значения за пределами безопасного JavaScript `number`:

```text
message.send(text: "Менеджеру", chatId: "123456789")
message.send(text: "В настроенный чат", chatId: $manager_chat_id)
```

DataField должен существовать и в момент создания эффекта содержать decimal signed
64-bit ID строкой. Получатель фиксируется до outbox/retry; изменение DataField влияет
только на следующую отправку. Missing/invalid значение завершает переход fail closed,
без fallback на текущего пользователя. Параметр не поддерживается у
`telegram.sendPhoto` / `telegram.sendVideo` / `telegram.sendVideoNote`.

`buttonLabel` и `buttonAction` задаются вместе. Для `buttonAction` допустимо
только точное действие `hosted_checkout`; обычные кнопки используют отдельную
форму `message.button`, описанную ниже. URL, checkout identifier, provider и
платёжные параметры автор не задаёт. Runtime разрешает действие в обычную
HTTPS URL-кнопку только из exact trusted `CHECKOUT_CREATED` lineage и иначе
fail-closed не отправляет сообщение.

В текущей грамматике исходника каждый named declaration, operation и control call
должен быть синтаксически завершён на одной строке. Многострочное оформление
аргументов пока не поддерживается parser или formatter; это текущая граница
реализации, а не неизменяемое свойство языка. Дополнительные готовые для копирования примеры
лежат в [examples](./examples/).

Запрещены произвольные выражения, `let`, `var`, `const`, loops, `while`, `for`,
generic programming `if/else`, user functions, `eval`, macros и произвольное
вложенное исполнение. Unknown construct, operation, argument или reference
даёт стабильную диагностику со source span и не проходит lowering.

`formatC2SourceV2` форматирует AST исходника SayDone Language в читаемый
SayDone Language. Он не генерирует исходник из
JSON и не переписывает persisted source автоматически.

## Базовый диалог

```text
field email(type: text, format: email, scope: user, label: "Почта")
// Внутри stateful блока:
message.send(text: "Введите email")
input.capture(type: email, field: email, invalid: retry, maxAttempts: 3, exhausted: restart, after: "5m", timeout: expired) => confirm
```

`input.capture` принимает `text`, `number`, `email`, `phone`, `telegram_update`; пишет в объявленное
поле `flow` или `user`. Number требует `type: number`; email/phone — `type: text`
с одноимённым `format`; обычный text — text без format. Сохранение, coercion,
области жизни и очистка значений совпадают с существующим DataField runtime.

`telegram_update` — узкая Telegram-specific форма для ожидания **следующего
Telegram Update**. Она принимает text, contact или media-message и сохраняет
exact JSON нормализованного входящего Update в text DataField без
преобразования в произвольный expression. Поле должно иметь `type: text` без
format. Длина JSON ограничена 4096 символами, чтобы его можно было безопасно
передать далее через обычный `content`/`message.send`; Update больше лимита
отвергается, а не обрезается. Это не захват callback, location, edited message
или любого не-Telegram event. Для одной capture не доступны
`invalid`/`maxAttempts`/`exhausted` и `after`/`timeout`: вход уже прошёл
trusted Telegram normalization и не должен создавать несколько competing
timers. Полный пример — [telegram-update-capture.c2](examples/telegram-update-capture.c2).
`data.set` по-прежнему задаёт authored literal, не текущий вход.

Text имеет `trim: true, required: true` по умолчанию. `required` проверяет
непустой после trim ввод даже при `trim: false`; последнее сохраняет пробелы.
Number использует строгую числовую запись с точкой, без пробелов и запятой.
Email и phone используют существующий bounded validator; phone — формат E.164.
Без invalid policy невалидный ввод не сохраняется и не выполняет success.

`invalid`, `maxAttempts` (1–10), `exhausted` необязательны, но задаются вместе.
`after`/`timeout` тоже вместе; after — положительная длительность с суффиксом
`s`, `m`, `h`, `d`, не более 365 дней. Цели — локальные блоки; exhausted должен
разрешаться в stateful block. Один ввод на блок, без `next`, `route` или
`subflow.call`; правила вызова/возврата subflow проверяет существующий compiler.
Timeout отменяется при success/invalid по прежней runtime semantics; повторное
вхождение в блок ввода создаёт его заново. Retry без смены состояния сохраняет
счётчик попыток; exhaustion очищает его. Успешный ввод сам по себе не обнуляет
счётчик прежних ошибок этого блока в текущем flow: это существующая BotSpec
semantics, не новая политика C2.

```text
on.text(equals: "Да", trim: true, ignoreCase: true) => accepted
on.text(equalsAny: ["Нет", "Позже"]) => restart
```

Ровно один `equals`/`equalsAny`; trim/ignoreCase по умолчанию false. Это exact
text текущего блока, не external route и не регулярное выражение. Пересекающиеся
маршруты отвергаются. Специфичный text проверяется раньше generic input.capture.

```text
message.send(content: confirmation)
message.button(key: yes, label: "Да") => accepted
message.button(key: no, label: "Нет") => restart
```

Кнопки идут непосредственно за сообщением или предыдущей его кнопкой.
Ключ уникален в сообщении и не зависит от отображаемой надписи; target — локальный
блок. Это BotSpec choice, а не отдельный effect. Нажатие продолжает только
авторизованный callback существующей версии; текст «Да» сам по себе не нажимает
кнопку. Generated eventType и provider payload не являются C2 source.
Смешение этих кнопок с `buttonAction` в одном сообщении не поддерживается.

По умолчанию каждая `message.button` получает собственную Telegram inline-строку:
это сохраняет rendering старых C2 source. Для явной раскладки добавьте одинаково
явный положительный integer `row` каждой кнопке данного сообщения:

```text
message.send(text: "Выберите один вариант")
message.button(key: basic, label: "Базовый", row: 1) => selected
message.button(key: pro, label: "Pro", row: 1) => selected
message.button(key: business, label: "Business", row: 2) => selected
```

Строки начинаются с `1`, идут без пропусков в declaration order и имеют от 1 до
8 кнопок; всего в одном inline message — не более 100 кнопок. Это SayDone
practical compatibility limit, а не заявленный Telegram public API guarantee.
Нельзя смешивать кнопки с `row` и без него в одном `message.send`. `row` не
создаёт новый callback format: key, generated callback semantic event, target,
проверка pinned version/integration и authorization остаются прежними. Полный
copy-ready пример с 16 первыми и всеми 50 кнопками: [telegram-inline-keyboards.c2](examples/telegram-inline-keyboards.c2).

Сохранённые поля выводятся через прежние `content(text: "{{name}}", required: [name])`
и `message.send(content: ...)`. Полные проверяемые исходники: [опрос](examples/survey.c2),
[меню](examples/menu.c2), [форма](examples/lead-form.c2), [ветвление](examples/branching.c2).

Не поддерживаются в C2: `telegram_contact` capture, request_contact/location,
произвольные URL-кнопки, aliases кнопок, dynamic expressions, regex/fuzzy.
Их наличие в BotSpec не расширяет C2. Старые C2 без новых форм сохраняют
semantic shape/checksum; миграция persisted source не требуется. Это граница
исходников и тестов; production rollout устанавливается отдельным evidence.
