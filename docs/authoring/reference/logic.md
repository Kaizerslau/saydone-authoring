# Логика: `Conditions`, `RuleSet`, `Subflow` и `SafeTransform`

<a id="logic"></a>

## Условия (`Conditions`)

<a id="conditions"></a>

`Condition` — ограниченное чистое JSON-условие над каноническим `DataField`. Оно не
пишет данные, не вызывает сервисы и не содержит языка выражений.

```json
{ "field": "team_size", "op": "greaterOrEqual", "value": 20 }
```

Используйте только опубликованные формы AST; не используйте JS, SQL, eval,
пути к объектам или неявное преобразование типов.

## Таблица решений (`RuleSet`)

<a id="rule-sets"></a>

`RuleSet` имеет упорядоченный `first_match`, одно типизированное записываемое
поле результата и явный `otherwise`.

```json
{
  "ruleSets": {
    "segment": {
      "mode": "first_match",
      "resultField": "segment",
      "rules": [{
        "key": "enterprise",
        "when": { "field": "team_size", "op": "greaterOrEqual", "value": 20 },
        "value": "enterprise"
      }],
      "otherwise": { "type": "value", "value": "standard" }
    }
  }
}
```

`when` использует тот же `Conditions` AST. `RuleSet` не содержит действий или эффектов и
динамических ссылок. Используйте `{ "type": "no_match" }` только когда
результат намеренно должен остаться без изменения.

## Статический именованный подмаршрут (`Subflow`)

<a id="subflows"></a>

```json
{
  "subflows": {
    "collect_contact": {
      "inputs": ["company_name"], "outputs": ["contact_phone"],
      "entry": "ask", "result": "done",
      "blocks": { "ask": { "input": { "type": "phone", "save": "contact_phone", "run": "done" } }, "done": {} }
    }
  }
}
```

Блок вызова достигается напрямую из маршрута и имеет статический `subflow` плюс
обычный блок с сохраняемым состоянием в `then`. Нельзя использовать косвенный
маршрут `next`, рекурсию, динамическую цель, запись поля кнопкой вызова или
прямой вызов/возврат из типизированного ввода.

## Безопасное преобразование (`SafeTransform`)

<a id="safe-transforms"></a>

Используйте только внутри явного `set`:

```json
{ "type": "set", "values": { "display_name": { "$transform": { "operation": "uppercase", "args": [{ "$from": "company_name" }] } } } }
```

Допустимые операции: `concat`, `uppercase`, `lowercase`, `coalesce`, `url_encode`,
`base64`, `round`, `add`, `subtract`, `multiply`, `divide`, `format_date`,
`timestamp`. Аргументы — скалярные литералы или точный `$from`; нельзя
использовать вложенные преобразования, произвольные формулы, часы реального времени или случайность
или побочные действия. `divide` отклоняется при нуле; `coalesce` — явная
операция по умолчанию для `missing`/`null`.
