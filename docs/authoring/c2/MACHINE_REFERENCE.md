# Машинная справка SayDone Language C2

Версия языка: `saydone-r2/1`.

Верхний уровень: `registry`, `project`, `field`, `content`, `http`, `ruleset`,
`funnel`, `subflow`, `provider`, `external.route`.

Внутри `block`: зарегистрированные операции, `saveState`, `next`, `route`,
`on.event`, `on.command`, `on.text`, `input.capture`, `message.button`.

Каждый named declaration, operation и control call занимает одну строку.
Unknown construct, argument или reference должен быть отвергнут diagnostic; не
используйте fallback, regex, fuzzy matching или произвольный код.

`input.capture` поддерживает `text`, `number`, `email`, `phone`, `telegram_update`.
`telegram_update` сохраняет raw JSON одного trusted Telegram message update
(text/contact/media) в text DataField не длиннее 4096 символов; callback и
остальные update types не становятся таким input. `on.text`
сопоставляет exact text текущего блока. `message.button` использует stable key,
не отображаемую надпись и не provider callback payload. `external.route`
сопоставляет нормализованный внешний input точно; provider/integration — только
явные optional restrictions.

`message.send(... buttonAction: hosted_checkout)` допускает только
`hosted_checkout`; URL и payment data не являются C2 source. Операция требует
реально доступную capability и иначе должна fail closed.

`telegram.sendPhoto(fileId: ...)`, `telegram.sendVideo(fileId: ...)` и
`telegram.sendVideoNote(fileId: ...)` принимают только literal Telegram
`file_id`. Это compiler-owned durable outbound action, не asset URL и не
generic provider call; она требует exact Telegram capability
`telegram.file_id_media` и pinned server-resolved destination.
