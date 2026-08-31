# `Bundle`, `Patch` и `RepairReportV1`

<a id="bundle-patch-repair"></a>

Первый ответ — один самостоятельный `SayDoneAuthoringBundleV1`:

```json
{
  "bundleVersion": 1,
  "artifacts": {
    "botSpec": {}, "dataFields": {}, "acceptance": {}, "qualification": {}
  }
}
```

Каждая часть исходного описания обязательна и затем структурно и семантически
проверяется SayDone. Первоначальный `Bundle` не содержит версии или контрольной суммы,
доверенных сведений об участнике, скомпилированного пакета, сгенерированного идентификатора,
значения учётных данных или полномочия жизненного цикла.

После неуспешной проверки используйте код/путь из полученного `RepairReportV1`
и его точные `sourceRevision`/`sourceChecksum`. Верните один минимальный
`SayDoneAuthoringPatchV1`, но никогда не создавайте заново `Bundle`:

```json
{
  "patchVersion": 1,
  "baseRevision": "<from RepairReport>",
  "baseChecksum": "<from RepairReport>",
  "changes": [{ "type": "set_bot_spec_entry", "entry": "menu" }]
}
```

Допустимые элементы `changes`: `replace_artifact`, `upsert_bot_spec_entry`,
`remove_bot_spec_entry`, `set_bot_spec_entry`. Для `upsert`/`remove` разрешены
только разделы `assets`, `content`, `httpResources`, `blocks`, `subflows`,
`ruleSets`. `Patch` содержит от 1 до 64 однозначных элементов `changes` и не
может дважды менять один логический элемент. Не придумывайте версию и контрольную сумму:
устаревшая базовая версия отклоняется. Успешный `Patch` снова полностью
проверяется; он не является автоматическим подтверждением.

После получения отчёта переходите по точному `documentationRef` из
`RepairReportV1`; ссылка ведёт только к публичному правилу и не раскрывает
готовое исправление.
