# Owner protocol

1. Передай external AI только файлы из manifest, exact task и generated seed binding.
2. AI должен вернуть ровно один JSON code block. Нажми Copy code внутри этого блока и сохрани скопированный JSON в ANSWER.json без изменений. Не используй Copy message: в некоторых chat UI он может вернуть Markdown-представление и изменить machine JSON.
3. Если SayDone видит `\_`, `\[` или `[https://...](https://...)`, заново скопируй содержимое JSON code block.
4. Harness строго parses, plans, applies и запускает canonical preview.
5. При rejection следующий turn получает exact resulting source и фактический report; ответ снова SayDoneAuthoringOperationsV1.
6. Максимум два repair cycles. После второго miss остановись.
7. confirmation_ready не выполняет lifecycle mutation и не заменяет owner confirmation.

Не добавляй сведения о qualification task, прежних ошибках, manual correction hints или private repository context.
