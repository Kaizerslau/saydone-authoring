# Fresh-AI qualification owner runbook V1

Статус: `CURRENT / AUTHORING-2C operational checkpoint`.

Этот runbook запускает только existing AUTHORING-2C harness. Он не вызывает
модель, не создаёт expected Bundle/Patch и не даёт deployment authority.
`fresh-ai` доступен после build как owner CLI.

## Trusted checkout preparation

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

`prepare-repair` writes `repair-input/base-source-snapshot.json` and
`repair-input/repair-report.json` only after existing canonical preview rejects
the Bundle. Transfer these normal workflow inputs with `TASK.md` and `public/` to
a **new** isolated fresh session. The AI returns only a Patch. Back on the trusted
machine:

```text
pnpm --filter @saydone/authoring-tools fresh-ai -- capture-patch --workspace <workspace> --attempt 1 --origin external_model --output <raw-patch.json> --context <trusted-context.json>
```

The default base/report paths are the actual `repair-input` files. The command
checks exact base revision/checksum, applies the bounded Patch and reruns preview.
For another repair cycle, transfer only the new actual RepairReport/current base
binding to another new fresh session.

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
