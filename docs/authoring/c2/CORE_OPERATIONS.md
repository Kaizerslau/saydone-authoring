# Основные операции

Операции записываются отдельными операторами внутри `block`; все named calls
записывайте одной строкой. Полные совместимые сочетания приведены в
[core-flow.c2](examples/core-flow.c2).

- `message.send(content: ...)` отправляет объявленный content.
- `data.set(field: ..., value: ...)` и `data.unset(field: ...)` изменяют DataField.
- `rules.apply(ruleSet: ...)` применяет ruleset.
- `http.request(resource: ..., result: ...)` и `http.save(...)` работают с HTTP-ресурсом.
- `schedule.create(...)` и `schedule.cancel(...)` управляют одноразовыми задачами.
- `subflow.call(subflow: ..., then: ...)` вызывает subflow.

Переходы `next`, `on.command` и другие control-конструкции принадлежат блоку,
а не являются самостоятельным документом.
