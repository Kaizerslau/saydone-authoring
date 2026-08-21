# SayDoneAuthoringOperationsV1

Верни только один JSON-документ SayDoneAuthoringOperationsV1 без Markdown и пояснений. Точная форма принадлежит schemas/saydone-authoring-operations-v1.schema.json. Краткое назначение операций и code-owned registries находятся в AI_OPERATIONS_CATALOG_V1.json.

Используй exact baseRevision/baseChecksum, предоставленные owner harness. Initial turn создаёт Case C поверх system seed. Repair turn меняет только фактическую ошибку поверх новой exact base и также возвращает Operations, а не Patch или Bundle.

Не передавай credentials, runtime/database IDs, provider payload, generated IDs, raw BotSpec AST, Patch changes, Bundle или deployment actions. Canonical preview после apply остаётся окончательной проверкой.
