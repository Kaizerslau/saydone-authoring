# SayDoneAuthoringOperationsV1

Верни ровно один fenced JSON block: строка ```json, затем один документ SayDoneAuthoringOperationsV1, затем ```. Не добавляй текст до или после блока. Точная форма принадлежит schemas/saydone-authoring-operations-v1.schema.json. Краткое назначение операций и code-owned registries находятся в AI_OPERATIONS_CATALOG_V1.json.

Initial prompt SayDone уже содержит exact baseRevision/baseChecksum для system seed. В каждом repair turn SayDone автоматически передаёт точные текущие literal baseRevision и baseChecksum. Перенеси оба значения в новый SayDoneAuthoringOperationsV1 дословно: не вычисляй, не угадывай, не восстанавливай их из предыдущего turn и не используй binding прошлого ответа. Repair turn меняет только фактическую ошибку поверх новой exact base и также возвращает Operations, а не Patch или Bundle.

Пользователь просто копирует generated prompt в AI, получает JSON code block через Copy code и вставляет JSON обратно в SayDone. Не нужно вручную искать revision/checksum или понимать их происхождение.

Примеры показывают изолированный синтаксис отдельных операций; они не являются готовой схемой продукта, задачей qualification или последовательностью для копирования. Canonical preview после apply остаётся окончательной проверкой.

Не передавай credentials, runtime/database IDs, provider payload, generated IDs, raw BotSpec AST, Patch changes, Bundle или deployment actions.
