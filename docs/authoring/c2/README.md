# SayDone Language: производственный авторинг V2

SayDone Language (внутренний контракт: C2) — основной авторский исходник для
каждой новой ревизии. Точный исходник сохраняется неизменяемо вместе с
`languageVersion`, `sourceChecksum`, производным семантическим снимком,
`semanticChecksum` и картой исходника. Обычная цепочка имеет только одно
направление:

```text
Исходник SayDone Language → типизированное AST → проверенная семантическая модель → canonical JSON → compiler → BotPackageV1
```

Canonical JSON остаётся детерминированным машинным представлением, но не
становится вторым независимо редактируемым исходником. Runtime получает только
неизменяемые скомпилированные артефакты и индекс маршрутов; грамматика SayDone
Language в runtime отсутствует.

## Продуктовая композиция

**Для нового Project все Funnel можно писать сразу в одном C2-документе.**
Один Project владеет одним canonical authored source; каждый `funnel` — логический
раздел единой composition. Изменения сохраняются как immutable Project revisions.
[Практический порядок](./LANGUAGE.md#единый-project-и-несколько-funnel) и
[полный copy-ready пример с sales/support](./examples/multi-funnel-routing.c2)
показывают маршруты между разделами без отдельных исходников.

текущей Project-first моделью задаёт текущую модель.
Предварительно создавать Funnel, её независимые версии или attachments для нового
authoring не требуется. Legacy source/version/attachment механизмы сохраняются;
историческая модель DEC-0195/DEC-0198 не является обязательным новым workflow.
Save, проверка, публикация/применение и operational Enable — отдельные действия;
сохранение исходника не включает автоматизацию.

Основные разделы:

- [язык и структура](./LANGUAGE.md);
- [операции и поток управления](./CORE_OPERATIONS.md);
- [внешние маршруты](./ROUTING.md);
- [поля данных, HTTP и расписания](./DATA_AND_INTEGRATIONS.md);
- [приёмку V2 и жизненный цикл](./ACCEPTANCE_AND_LIFECYCLE.md).
- [базовые диалоги и проверяемую матрицу](./COMPLETENESS_MATRIX.json).

Обычные диалоги выражаются `input.capture`, `on.text`, `message.button` и
существующими content/DataField. [Опрос](examples/survey.c2), [меню](examples/menu.c2),
[форма](examples/lead-form.c2) и [ветвление](examples/branching.c2) проверяются через
C2 compiler и runtime. Это доказательство исходников, не production deployment.
Матрицы V1 вне этого каталога относятся к совместимому BotSpec Bundle.

Публичная application boundary `previewC2AuthoringV2` выполняет проверку,
семантический предпросмотр, компиляцию и создаёт confirmation binding, связанный
с контрольной суммой. `C2SourceRevisionServiceV2` отвечает за точное
get/create/latest persistence; injected authoring verifier повторно доказывает,
что семантический снимок получен именно из точного исходника SayDone Language
при записи и reopening. Детали PostgreSQL не являются контрактом будущего frontend.

JSON → SayDone Language не входит в обычный жизненный цикл. Такая операция может
появиться только как явно названный migration, legacy import, recovery или
diagnostic tool и гарантировать лишь семантическую эквивалентность, не
восстановление исходного текста.
