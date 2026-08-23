# SayDoneAuthoringOperationsV1

Верни ровно один fenced JSON block: строка ```json, затем один документ SayDoneAuthoringOperationsV1, затем ```. Не добавляй текст до или после блока. Точная форма принадлежит schemas/saydone-authoring-operations-v1.schema.json. Краткое назначение операций и code-owned registries находятся в AI_OPERATIONS_CATALOG_V1.json. Точные arity, типы аргументов, result type, null/missing behavior и запрет coercion для каждого `set_safe_transform` опубликованы в AI_SAFE_TRANSFORM_SIGNATURES_V1.json; сверяй operation с этой матрицей до ответа.

Initial prompt SayDone уже содержит exact baseRevision/baseChecksum для system seed. В каждом repair turn SayDone автоматически передаёт точные текущие literal baseRevision и baseChecksum. Перенеси оба значения в новый SayDoneAuthoringOperationsV1 дословно: не вычисляй, не угадывай, не восстанавливай их из предыдущего turn и не используй binding прошлого ответа. Repair turn меняет только фактическую ошибку поверх новой exact base и также возвращает Operations, а не Patch или Bundle.

## Choice → Subflow

Если choice сохраняет business value, запускает reusable Subflow и после его return продолжает путь в logical block, используй `connect_choice_to_subflow`. В `set_choice_block` у этой choice не указывай `nextBlockKey`. Передай только source block/choice, target Subflow, continuation и необходимые business field mappings. Не создавай call block, bridge, return wiring или generated identity: ими полностью владеет SayDone. `set_subflow_call` и choice с `nextBlockKey` остаются поддерживаемым низкоуровневым маршрутом.

Приёмочные сценарии покрывают каждый самостоятельный авторский бизнес-путь. `Button` и его text aliases одной `choice` — эквивалентные представления одного пути; достаточно одного корректного сценария выбора. Явно объявленный command route, включая `/start`, — самостоятельный путь и требует собственного сценария. Не используйте generated runtime IDs для покрытия: canonical preview сам сопоставляет исходную семантику.

Пользователь просто копирует generated prompt в AI, получает JSON code block через Copy code и вставляет JSON обратно в SayDone. Не нужно вручную искать revision/checksum или понимать их происхождение.

Примеры показывают изолированный синтаксис отдельных операций; они не являются готовой схемой продукта, задачей qualification или последовательностью для копирования. Canonical preview после apply остаётся окончательной проверкой.

Не передавай credentials, runtime/database IDs, provider payload, generated IDs, raw BotSpec AST, Patch changes, Bundle или deployment actions.
