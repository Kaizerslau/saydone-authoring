# SayDoneAuthoringOperationsV1

Верни только один JSON-документ SayDoneAuthoringOperationsV1 без Markdown и пояснений. Точная форма принадлежит schemas/saydone-authoring-operations-v1.schema.json. Краткое назначение операций и code-owned registries находятся в AI_OPERATIONS_CATALOG_V1.json.

Используй exact baseRevision/baseChecksum, предоставленные owner harness. Initial turn создаёт решение поверх system seed. Repair turn меняет только фактическую ошибку поверх новой exact base и также возвращает Operations, а не Patch или Bundle.

Примеры показывают изолированный синтаксис отдельных операций; они не являются готовой схемой продукта, задачей qualification или последовательностью для копирования. Canonical preview после apply остаётся окончательной проверкой.

Не передавай credentials, runtime/database IDs, provider payload, generated IDs, raw BotSpec AST, Patch changes, Bundle или deployment actions.
