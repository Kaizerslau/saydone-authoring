# Структура BotSpecV1

<a id="structure"></a>

<a id="capabilities"></a>

Используйте этот раздел для `specVersion`, `bot`, `entry`, `blocks` и `actions`.
`entry` — статический логический ключ начального блока. Ключ каждого `block`
должен быть статическим.

```json
{
  "specVersion": 1,
  "bot": { "key": "service_bot", "name": "Сервис" },
  "entry": "menu",
  "blocks": {
    "menu": {
      "actions": [{ "type": "send", "text": "Выберите вариант" }]
    }
  }
}
```

Блок с сохраняемым состоянием (`saveState` отсутствует или `true`) может
ожидать ввод, обработчики и `fallback`. Переходный блок обязан иметь
`"saveState": false`; он выполняет действия по порядку и может иметь только
статический `next`. Не создавайте цикл только из переходных блоков.

Поддерживаются следующие значения `type` для действий: `send`, `set`, `unset`, `track`, `schedule`,
`cancelSchedule`, `call`, `applyRuleSet`. Не добавляйте неизвестные поля или
`actions`: строгая проверка отклоняет неизвестные поля.

`set` записывает только объявленный `DataField` со `scope` `flow` или `user`.
Значение — литерал JSON, `{ "$from": "field_key" }` или ограниченный
`$transform`. `unset` явно переводит поле в отсутствующее значение, а не в
`null` или пустую строку.

Минимальный отдельный пример `examples/minimal/structure-entry.bundle.json`
будет добавлен в DOCS-AI-3. До него используйте полный
`examples/simple-menu.bundle.json`.
