# DataFieldCatalogV1

<a id="data-fields"></a>

`dataFields` — самостоятельная часть `Bundle`. Один статический `key` задаёт
одно каноническое бизнес-значение; `scope` не создаёт альтернативные имена.

```json
{
  "dataFieldCatalogVersion": 1,
  "fields": [{
    "key": "company_name",
    "label": "Название компании",
    "scope": "user",
    "type": "text",
    "visibility": "standard"
  }]
}
```

Допустимые `scope`: `flow`, `user`, `bot`, `system`. Обычные `input`, `set` и
`unset` записывают только `flow` и `user`. Допустимые `type`: `text`, `number`,
`boolean`, `date`, `datetime`; `visibility`: `standard`, `technical`, `hidden`.
`format` допускает `phone`, `email`, `date`, `datetime` только для совместимого
типа. `min`/`max` — только для `number`. `bot` требует совместимый
`defaultValue`; `system` — контролируемый разрешитель только для чтения.

`missing`, `null`, `""`, `0` и `false` различны. Не используйте неявное
преобразование типов, пути к вложенным объектам или динамические имена полей.
Ключи имеют вид `snake_case`, длину от 1 до 64 символов и не могут быть
`__proto__`, `prototype` или `constructor`.

Когда не применять: повторяющиеся бизнес-записи не моделируются динамическими
ключами `DataField`.
