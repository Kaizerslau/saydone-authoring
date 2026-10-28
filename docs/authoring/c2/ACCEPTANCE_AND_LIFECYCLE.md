# Приёмка V2 и lifecycle SayDone Language

`BotAcceptanceSpecV2` привязан к exact `sourceRevision`, `sourceChecksum` и
`semanticChecksum`. Он использует semantic input, optional registered provider
facts, logical block/DataField/content references и business events. Generated
`stateKey`, rule/effect keys, runtime UUID, DB ID и transport ID в authored
Acceptance запрещены.

Обычная кнопка задаётся шагом `{"type":"button","funnel":"main","block":"start","key":"begin"}`.
Это logical key из `message.button`; он должен однозначно определять кнопку в
блоке. Compiler связывает его с generated событием в ScenarioV2, как в прежнем
BotSpec acceptance, без callback payload в авторском сценарии. Это проверка
reducer, не обход runtime callback authorization. Обычный semantic action input
по-прежнему является командой/внешним входом и не выдаётся за callback.

Canonical authoring lowering создаёт `ScenarioV2`; loader не добавляет сценарии
и не вставляет implicit `START`. Несовпадение source/semantic/package binding
даёт `acceptance.v2.source_binding_mismatch`.

Confirmation binding включает source revision/checksum, semantic checksum и
compiled package checksum. Persisted source revision неизменяема; edit создаёт
следующую revision. Persistence принимает injected `verifyC2SourceSnapshotV2`,
который повторно parses source и сравнивает canonical semantic meaning;
отдельно корректные, но не связанные source/snapshot отклоняются.
`applyC2SourceReplacementV2` принимает только exact current
revision/checksum и отклоняет stale source как `c2.patch_stale_source`. Fuzzy
patch и requirement-repair model loop не входят в CN-1E.
