# Telegram capability boundary

Telegram является текущим provider-specific contract только там, где C2 явно
использует зарегистрированный provider fact. Обычные `external.route`,
`input.capture`, `on.text` и `message.button` provider-neutral.

Исключение — `input.capture(type: telegram_update, ...)`: оно принимает один
already-normalized Telegram message update (text/contact/media) и сохраняет
его raw JSON в text DataField в пределах 4096 символов. Это не generic JSON
input и не доступ к callback/update other types. `telegram.sendPhoto`,
`telegram.sendVideo`, `telegram.sendVideoNote` принимают literal file_id
через требуемую `telegram.file_id_media` capability; assets, URL, file bytes и
provider credentials в C2 source не допускаются.

Нормализованный `/start` является ordinary external input, а не специальным
`START` или неявным restart. Для Telegram-specific command fact используйте
только form из [маршрутизации](../c2/ROUTING.md), включая exact registered
`fact`, `factVersion` и `factValue`. Не придумывайте provider facts, callback
payload или credentials.

Этот набор не заявляет поддержку VK, Instagram или иных provider adapters.
