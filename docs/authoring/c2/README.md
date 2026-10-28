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

принятым product architecture решением принимает Funnel
как независимую versioned единицу и Project composition с отдельным явным
выбором версии. Это не меняет данную C2 source boundary и не делает JSON вторым
исходником. Независимое persistence Funnel/versions и целевой UX пока не
доказаны end-to-end: решение имеет `partial` implementation status.

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
