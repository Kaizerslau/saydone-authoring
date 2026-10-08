# Telegram

Текущий публичный provider-факт для маршрута — `telegram.bot_command` версии
`1`. Для `/start gift` применимы `command: "start"`, `arguments: "gift"` и
`startParameterCandidate: "gift"`. Это provider-aware route, а `/start` при
этом остаётся обычным входом Telegram.

Не публикуйте provider payload, credentials или каталог будущих возможностей:
CN-1F ещё не начат.
