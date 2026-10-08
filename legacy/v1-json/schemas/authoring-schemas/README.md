# Структурные JSON Schema SayDone V1

Здесь находятся пять версионированных схем для предварительной проверки формы
исходного JSON. Каждая схема использует JSON Schema Draft 2020-12.

| Артефакт | Схема |
|---|---|
| `SayDoneAuthoringBundleV1` | [saydone-authoring-bundle-v1.schema.json](saydone-authoring-bundle-v1.schema.json) |
| `SayDoneAuthoringPatchV1` | [saydone-authoring-patch-v1.schema.json](saydone-authoring-patch-v1.schema.json) |
| `BotSpecV1` | [bot-spec-v1.schema.json](bot-spec-v1.schema.json) |
| `BotAcceptanceSpecV1` | [bot-acceptance-spec-v1.schema.json](bot-acceptance-spec-v1.schema.json) |
| `DataFieldCatalogV1` | [data-field-catalog-v1.schema.json](data-field-catalog-v1.schema.json) |

Прохождение схемы подтверждает только структуру. Окончательное решение всегда
принимают строгий анализатор исходного JSON, каноническая проверка, компилятор и
предварительная проверка SayDone. В частности, схема не проверяет повторяющиеся JSON-ключи, ссылки между
артефактами, покрытие, квалификацию или точную привязку `Patch` к текущему
снимку исходного описания.

`saydone-authoring-bundle-v1.schema.json` использует стабильные внешние URN
ссылки на схемы `BotSpecV1`, `DataFieldCatalogV1` и
`BotAcceptanceSpecV1`. Потребитель Bundle должен зарегистрировать все четыре
соответствующие schema-файла в JSON Schema validator; это проверяется с `Ajv`
в canonical parity corpus. Схемы остаются доступными в public export, но
автоматически прикладываются к отдельному заданию ИИ только когда описывают
его текущий ожидаемый ответ.
