# Руководство владельца по внешней проверке ИИ V1

Статус: `CURRENT / AUTHORING-2C operational checkpoint`.

Это руководство использует только существующий проверочный контур AUTHORING-2C.
Он не вызывает модель, не создаёт ожидаемые Bundle/Patch и не даёт полномочий на
deployment. `fresh-ai` доступен после build как служебная команда.

## Обычный порядок работы

1. Попросите Codex подготовить следующий шаг.
2. Откройте `OWNER_INSTRUCTIONS.txt` и выполните четыре коротких действия из
   этого файла.
3. Вернитесь в Codex и напишите «готово, продолжай».

`owner-start`, `owner-advance` и `owner-status` автоматически ведут номер шага,
точную основу/отчёт для исправления, хэши и необходимость нового либо прежнего
диалога. Они не вызывают модель и не заменяют фактическое подтверждение владельца.

Для финальной проверки модели не должны быть доступны основной репозиторий
SayDone, Git checkout, `AGENTS.md`, `PROJECT_STATE.md`, планы, исходный код,
тесты, MCP или инструменты с доступом к репозиторию. Доступны только `PROMPT.txt`,
приложенные файлы и закреплённая публичная документация. Каждый Case A/B/C
начинается в отдельном новом диалоге; исправления одного Case выполняются в том
же диалоге.

## Advanced / trusted checkout preparation

На trusted SayDone checkout сначала соберите authoring tools:

```text
pnpm --filter @saydone/authoring-tools build
pnpm --filter @saydone/authoring-tools fresh-ai -- prepare --workspace <outside-repository-workspace> --case case-a-service-menu
```

`<outside-repository-workspace>` обязан быть вне checkout. `prepare` создаёт
только `TASK.md`, `PUBLIC_CONTEXT_MANIFEST.json` и `public/`; последний содержит
точные files, bytes и SHA-256. Available owner-run cases are defined in
[`AUTHORING_2C_FRESH_AI_TASKS_V1.json`](../evidence/AUTHORING_2C_FRESH_AI_TASKS_V1.json):
`case-a-service-menu`, `case-b-bounded-repair`, `case-c-lead-application`.

## A–E: create and vertical cases

1. Run `prepare` for Case A or C on the trusted checkout.
2. Transfer **only** the prepared workspace to a disposable isolated environment,
   or expose only that workspace to an external fresh AI session.
3. Give the AI only `TASK.md` and `public/`. It returns only the requested strict
   JSON Bundle; do not provide a ready Bundle, source, plans or test hints.
4. Copy the raw output file back to the trusted qualification machine.
5. Prepare a trusted context JSON locally (values are owner facts, not AI source):

```json
{
  "organizationId": "owner-approved-organization",
  "actorId": "owner-approved-actor",
  "correlationId": "owner-run-correlation",
  "operationId": "owner-run-operation",
  "botKey": "owner-run-bot-key",
  "botVersion": 1,
  "contentReleaseNumber": 1,
  "generatedBy": "external-fresh-ai"
}
```

6. Capture the unmodified returned artifact through the canonical path:

```text
pnpm --filter @saydone/authoring-tools fresh-ai -- capture-bundle --workspace <workspace> --attempt 1 --origin external_model --output <raw-bundle.json> --context <trusted-context.json>
```

The command records output hash, source/preview/RepairReport evidence and all
canonical gates. Unsafe output is hash-only and never retained as raw artifact.

## F: real RepairReport → Patch

Case B starts from an owner-supplied invalid Bundle. Its failure must be actual,
deterministic and source-level; do not handcraft a RepairReport.

```text
pnpm --filter @saydone/authoring-tools fresh-ai -- prepare --workspace <outside-repository-workspace> --case case-b-bounded-repair
pnpm --filter @saydone/authoring-tools fresh-ai -- prepare-repair --workspace <workspace> --bundle <invalid-bundle.json> --context <trusted-context.json>
```

`prepare-repair` writes the immutable initial pair
`repair-input/base-source-snapshot.json` and `repair-input/repair-report.json`
only after existing canonical preview rejects the Bundle. Transfer these normal
workflow inputs with `TASK.md` and `public/` to a fresh isolated session. The AI
returns only a Patch. Back on the trusted machine:

```text
pnpm --filter @saydone/authoring-tools fresh-ai -- capture-patch --workspace <workspace> --attempt 1 --origin external_model --output <raw-patch.json> --context <trusted-context.json>
```

The default base/report paths are the actual initial `repair-input` files. The
command checks exact base revision/checksum, applies the bounded Patch and reruns
preview. If a later gate rejects, its JSON result includes
`evidence.nextRepairInput` and records the exact canonical current source plus
safe actual report under:

```text
repair-input/after-attempt-1/base-source-snapshot.json
repair-input/after-attempt-1/repair-report.json
```

Prior pairs are immutable. Do not reconstruct a Bundle or edit source manually.
For attempt 2 in the **same external Case B session**, transfer only that new
pair (plus the already staged `TASK.md` and `public/`) and run explicitly:

```text
pnpm --filter @saydone/authoring-tools fresh-ai -- capture-patch --workspace <workspace> --attempt 2 --origin external_model --output <raw-patch-2.json> --context <trusted-context.json> --base <workspace>/repair-input/after-attempt-1/base-source-snapshot.json --repair-report <workspace>/repair-input/after-attempt-1/repair-report.json
```

The next Patch must bind to that new revision/checksum; a Patch bound to the
initial pair fails closed. A `confirmation_ready` preview creates no additional
repair input. If an older workspace contains attempt-1 evidence created before
this checkpoint, replay the same retained raw Patch against the same initial
pair with `capture-patch --attempt 1 --replay-existing-attempt true ...` solely to materialize the deterministic
`after-attempt-1` pair; it does not create a new model attempt or alter the raw
artifact.

## G: owner isolation attestation

After external execution, create an owner facts file. All booleans are mandatory;
model/provider/session fields are optional and must be omitted if unknown. Tokens
must be an exact integer or `NOT_AVAILABLE`.

```json
{
  "attestationVersion": 1,
  "attestedBy": "owner-name-or-role",
  "recordedAt": "2026-08-16T00:00:00.000Z",
  "runner": "external-isolated-runner-description",
  "freshModelSession": true,
  "noRepositoryMount": true,
  "noGitCheckout": true,
  "noAgsOrProjectState": true,
  "noPlansSourceOrTests": true,
  "noPreviousCaseHistory": true,
  "noMcpOrRepositoryTools": true,
  "publicWorkspaceOnly": true,
  "tokenCount": "NOT_AVAILABLE",
  "turns": 1,
  "repairCycles": 0,
  "manualSemanticHints": 0
}
```

```text
pnpm --filter @saydone/authoring-tools fresh-ai -- attest --workspace <workspace> --attestation <owner-facts.json>
```

The result is deliberately `owner_attested_not_verified`; it does not mutate
`prepared_not_attested` in the context manifest and does not turn a VM/container
into proof of model-history isolation. The owner must separately attest that the
model/session was fresh and that no repository mount, Git checkout, AGENTS,
PROJECT_STATE, plans, source/tests, prior Case history, MCP or repository tools
were available.

## H: final review inputs

Run A, B and C in separate fresh contexts. Retain each workspace evidence,
attestation, public file manifest and raw safe artifact/hash. Only after all three
external-model captures are canonical `confirmation_ready` and their isolation
facts are recorded may AUTHORING-2C completion review begin.
