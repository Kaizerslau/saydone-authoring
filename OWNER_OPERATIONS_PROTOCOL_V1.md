# Owner protocol

1. Передай external AI только эти семь файлов, exact task и generated seed binding.
2. Ответ сохрани как raw JSON без изменения. Initial output contract: SayDoneAuthoringOperationsV1.
3. Harness строго parses, plans, applies и запускает canonical preview.
4. При canonical rejection следующий turn получает exact resulting source, фактический RepairReportV1 и тот же compact contract; ответ снова SayDoneAuthoringOperationsV1.
5. Максимум два repair cycles. После второго miss остановись.
6. confirmation_ready не выполняет lifecycle mutation и не заменяет owner confirmation.

Не добавляй сведения о прежних Case C ошибках, manual correction hints или private repository context.
