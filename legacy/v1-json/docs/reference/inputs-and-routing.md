# Ввод и маршрутизация

<a id="routing"></a>

<a id="start-payloads"></a>

<a id="typed-inputs"></a>

`input` описывает типизированный ввод в обычном блоке. `save` — объявленный
`DataField` для записи, `run` — статический логический блок после успешного ввода.

```json
{
  "input": {
    "type": "number",
    "save": "team_size",
    "run": "classify",
    "invalid": { "run": "ask_team_size", "maxAttempts": 2, "exhaustedRun": "input_exhausted" },
    "timeout": { "after": "10m", "run": "timed_out" }
  }
}
```

Поддерживаемые `type`: `text`, `number`, `email`, `phone`, `telegram_contact`.
Маршруты проверки, повтора, исчерпания попыток и тайм-аута остаются обычными
блоками; они не могут напрямую вызывать или возвращать `Subflow`.
`telegram_contact` сочетается с `request_contact` на том же логическом пути.

Статическая маршрутизация:

```json
{
  "startPayloads": { "partner": "partner_menu" },
  "on": [
    { "command": "help", "run": "help" },
    { "text": { "equalsAny": ["Помощь"], "trim": true, "ignoreCase": true }, "run": "help" }
  ],
  "fallback": { "run": "unmatched" }
}
```

Параметр запуска — точное статическое значение, а не регулярное выражение, разбор
или захват значения. Ключи команд статичны; суффикс имени пользователя Telegram —
транспортное поведение, а не синтаксис исходного описания. Более точные псевдонимы и
`on.text` маршруты проверяются до общего типизированного ввода; сам общий ввод
не создаёт неоднозначности. `START`, `TEXT_RECEIVED`, `COMMAND_RECEIVED`,
`UNKNOWN_INPUT` никогда не являются публичным `on.event`.

Порядок: контролируемый жизненный цикл `START` → параметр запуска → `entry` →
точная команда → псевдоним кнопки → точный текст → общий ввод → `fallback` блока →
`fallback` потока или всего бота → необработанный случай.
