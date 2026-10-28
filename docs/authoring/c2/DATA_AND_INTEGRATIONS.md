# Поля данных, правила, HTTP и расписания SayDone Language

`field` объявляет logical DataField с `type`, `scope` и `label`; для text
допустим `format: email` или `format: phone`. `input.capture` сохраняет
проверенный inbound scalar в writable flow/user поле; type/format должны
совпадать с выбранным видом ввода. Подробнее — [LANGUAGE](LANGUAGE.md#базовый-диалог).
Операции
`data.set`/`data.unset`, RuleSet `resultField` и HTTP response mappings ссылаются
только на logical field key. Unknown field отклоняется при семантической проверке.

`ruleset` использует bounded Conditions и `first_match` semantics существующего
compiler. Его `resultField` записывает сам `rules.apply`: тот же field нельзя
заранее записывать вручную в одной transient transition chain. SayDone Language не добавляет
произвольный expression evaluator.

Внутри блока data-producing операции `data.set`, `data.unset`, `rules.apply` и
`schedule.cancel` записываются до effect-producing `message.send`,
`schedule.create`, `http.request` и `analytics.track`. Это сохраняет deterministic action phases
существующего runtime contract.

`http` объявляет method, endpoint, idempotency, optional credential reference и
body mappings. `http.request` создаёт существующий typed effect, а `http.save`
связывает JSON pointer response с DataField. Secrets и raw provider payload не
помещаются в diagnostics SayDone Language или route index.

`schedule.create` использует `after` либо `at` и logical target. Compiler
переиспользует существующий schedule effect; CN-1D runtime сохраняет exact
stable target и independent timer lineage. `schedule.cancel` использует тот же
logical schedule ID. Автоматического DataField snapshot/capture нет.

Provider overlay применяется лишь к действительно уникальному формату или
поведению. Полный provider capability catalog, Telegram pack и другие provider
descriptors относятся к CN-1F и здесь не определяются.

## Явные аналитические события

Разработчик размещает `analytics.track` в выбранных блоках. Автоматического
учёта посещений каждого блока нет:

```text
analytics.track(event: "lead_progress", properties: [step:"qualified"], fields: [lead_score:score])
```

`event` — непустое имя. Необязательный `properties` задаёт пары
`имя_свойства:"строковое значение"`; `fields` — пары `имя_свойства:ключ_поля`.
Имена свойств используют `snake_case` без зарезервированных имён; повторение
имени, в том числе между двумя списками, запрещено. Поля должны быть объявлены.
Текущие списки не представляют произвольный JSON: вложенные объекты и массивы,
числа и логические константы в `properties` недоступны. Для числового или
логического значения используйте соответствующее поле через `fields`.

Значения полей фиксируются после изменений данных текущего перехода перед
постановкой эффекта; отсутствующее значение становится `null`. Worker берёт
сохранённые значения, последующие изменения поля событие не меняют. В цепочке
временных блоков применяются общие фазы перехода; две отдельные точки со своими
значениями следует разделять обычным переходом, как в
[полном примере](./examples/analytics.c2).

Для нового C2-события `occurred_at` — серверное время применения операции и
постановки эффекта в транзакции исполнения. `recorded_at` — время записи события
worker. При задержке с 23:59 до 00:02 момент события остаётся 23:59. Откат
транзакции не создаёт успешного факта. Технический повтор сохраняет факт и его
время, новое выполнение создаёт новое событие.

Сервер проверяет исходную ревизию C2, её контрольные суммы и точную операцию;
при исполнении закрепляет фактический `botVersionId`. Подтверждённые сведения
хранятся с событием независимо от очереди и дальнейших обновлений воронки.
Автор не задаёт эти сведения или время через аргументы либо свойства.
Старые события сохраняют прежнюю семантику времени и неизвестное происхождение.
Библиотека включает объявления выбранных полей; просмотр блока показывает
только объявленные имена, постоянные строки и ссылки на поля.
