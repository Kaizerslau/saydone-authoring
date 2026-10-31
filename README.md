# SayDone authoring

SayDone authoring выполняется на SayDone Language (внутренний контракт: `C2`).
Автор выдаёт полный исходный текст C2; платформа разбирает и проверяет его,
строит семантическую модель и детерминированную машинную проекцию для compiler.
JSON не является исходным форматом новой ревизии.

Начните с [AI_START_HERE.md](AI_START_HERE.md), затем прочитайте
[язык](docs/authoring/c2/LANGUAGE.md), [операции](docs/authoring/c2/CORE_OPERATIONS.md),
[маршрутизацию](docs/authoring/c2/ROUTING.md) и [проверяемые examples](docs/authoring/c2/examples/README.md).

Материалы в `legacy/v1-json` сохранены только для history и migration. Не
используйте V1 JSON для новых C2-сценариев.

## Один Project — один полный C2

Для нового Project все Funnel можно писать сразу в одном C2-документе.
Один project и несколько funnel образуют общую composition; не создавайте
обязательные независимые исходники или версии каждой Funnel.
Практический workflow и отличие от legacy: [LANGUAGE.md](docs/authoring/c2/LANGUAGE.md#единый-project-и-несколько-funnel).
[Полный пример sales/support](docs/authoring/c2/examples/multi-funnel-routing.c2)
показывает локальные переходы и cross-funnel route через transient блоки.
Сохранение, проверка exact revision, публикация и operational Enable раздельны.
Legacy данные и механизмы совместимости сохраняются.
