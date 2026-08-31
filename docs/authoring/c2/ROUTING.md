# Маршрутизация

Маршрут выбирается однозначно: один match ведёт в exact target; при нуле match
и активном блоке обработка остаётся в текущем блоке; при нуле match без
активного блока ввод не обработан; два и более match завершаются fail closed.
Нет неявных priority, порядка, regex, fuzzy matching или скрытой специфичности.

`/start` — обычный Telegram input, а не restart, обязательная точка входа или
универсальный lifecycle.

`external.route` — top-level declaration. В текущей грамматике его named call
полностью находится на одной строке:

```c2
external.route telegram_gift(kind: text, value: gift, provider: telegram, fact: telegram.bot_command, factVersion: 1, factValue: "{\"arguments\":\"gift\",\"command\":\"start\",\"startParameterCandidate\":\"gift\"}", target: sales.open_gift)
```

См. полный [telegram-route.c2](examples/telegram-route.c2).
