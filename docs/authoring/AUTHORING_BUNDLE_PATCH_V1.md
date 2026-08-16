# Authoring Bundle, Patch and Repair V1

Статус: `IMPLEMENTED / AUTHORING-1 COMPLETION VERIFIED`

JSON — canonical machine serialization. YAML возможен только как adapter в те же objects без второй semantics.

`SayDoneAuthoringBundleV1` — portable initial import/export/snapshot container для BotSpec, Data Fields, content, acceptance, qualification, resources/integrations и asset references. Bundle не является единственной mutation unit: после import SayDone versioning разделяет definition, content и другие artifacts.

Compatible content-only edit создаёт independent Content Release без
rebuild flow semantics. Future CONTENT-1 formats: `plain`, safe Telegram subset
`telegram_html`, `telegram_markdown_v2`. Assets имеют separate lifecycle;
Telegram `file_id` — transport binding, не universal asset identity.

AI может объявить logical asset slot с kind/required metadata; binary
загружается отдельно. `SayDoneAuthoringBundleV1` не равен future
`SayDoneProjectArchiveV1`: archive может позже содержать manifest, JSON и
binaries, тогда как ordinary Bundle хранит logical references.

`SayDoneAuthoringPatchV1` — bounded domain-level change, revision/checksum-bound и stale-base fail-closed. Он не обязан быть raw RFC JSON Patch и не требует full-bundle regeneration для local edit. `RepairReportV1` — machine-readable AI repair packet с codes, paths и bounded context.

Chat mode является first-class:

```text
ordinary AI chat → Bundle → copy/paste → validate/preview
failure → RepairReport → same chat → Patch → validate/preview
```

Agent mode автоматизирует те же contracts, но не является requirement. Embedded AI chat, MCP/tool access и customer deployment authority этим решением не вводятся. Binary media не хранится base64 как main model; bundle содержит logical asset references. Local request вроде «поменяй текст после оплаты» должен возвращать small semantic Patch, а не full Bundle regeneration.

## Реализованный AUTHORING-1B contract

Canonical implementation находится в `apps/authoring-tools/src/loader.ts` и
переиспользует существующий `executeAuthoringOperation`; второй compiler,
validator, Scenario lowerer или lifecycle service не создан.

### Bundle и source snapshot

`SayDoneAuthoringBundleV1` — strict bounded JSON:

```text
bundleVersion: 1
artifacts.botSpec
artifacts.dataFields
artifacts.acceptance
artifacts.qualification
```

Content catalog, HTTP Resources/credential references и logical asset slots
остаются внутри canonical `BotSpecV1`, как требует current compiler contract.
Bundle не дублирует их отдельной semantics. Strict parser отклоняет duplicate и
unknown container keys, plaintext credentials, binary/base64/provider payload,
generated runtime/package/checksum fields и actor/organization/lifecycle
authority.

Loader создаёт immutable in-memory `AuthoringSourceSnapshotV1` с canonical
source checksum, per-artifact checksums и loader-issued deterministic revision.
Checksum/revision не принимаются из initial Bundle. Snapshot integrity повторно
проверяется перед Patch и preview.

### Patch

`SayDoneAuthoringPatchV1` содержит `patchVersion: 1`, exact
`baseRevision`/`baseChecksum` и `1..64` domain changes. V1 разрешает:

- replacement одного canonical source artifact;
- upsert/remove static entry в `assets`, `content`, `httpResources`, `blocks`,
  `subflows` или `ruleSets`;
- изменение logical `entry` block.

Patch не является RFC JSON Patch, не принимает arbitrary pointer/package
fragment и не задаёт resulting revision/checksum. Duplicate target и stale base
fail closed. Result получает новый loader-issued revision/checksum и обязан снова
пройти весь preview; Patch сам не создаёт confirmation-ready status.

### Trusted preview и RepairReport

`TrustedAuthoringContextV1` приходит отдельно от source и содержит exact
organization/actor/correlation/operation identity плюс controlled
`CompileContextV1`. HTTP/tool payload не может переопределить trusted validation
organization или checked-at boundary.

Preview выполняет только такую последовательность:

```text
strict source/snapshot integrity
→ canonical compile
→ inspect
→ validate
→ scenarios
→ coverage
→ qualify
```

Compiler остаётся единственным владельцем package assembly, SourceMap,
MechanicalProofPlan и ScenarioV2 lowering. Preview не изменяет BotPackage после
compile. Любой `failed`/`blocked` gate прекращает цепочку, отмечает remaining
gates `not_run`, не создаёт confirmation и возвращает `RepairReportV1` со stable
code, public `/artifacts/...` path, severity, redacted message, bounded context,
affected gate, `documentationTopic` и exact source revision/checksum.
`documentationTopic` — один ключ существующей темы manifest для выбора already
published contract/example; он не задаёт новую семантику, не содержит correct
value и не заменяет code/path. Stack, internal absolute path, SQL, secret и
environment dump не возвращаются.

Успешный result содержит canonical package/artifact checksums, SourceMap,
MechanicalProofPlan, стабильные summaries всех gates, compatibility issues,
`lifecycleMutationOccurred: false` и confirmation request. Нестабильные runtime
timestamps/results не входят в normalized preview evidence, поэтому одинаковый
Bundle + trusted context даёт byte-equivalent machine result.

### Explicit confirmation handoff

Confirmation — не boolean. Request и resulting artifact привязаны к exact
preview version, source revision/checksum, final package checksum,
organization, trusted actor, correlation и operation. Challenge и artifact
checksums защищают canonical binding от случайной/partial mutation.
`TrustedAuthoringPreviewSessionV1` сохраняет exact compiler-produced
`BotPackageV1` только внутри trusted application boundary; AUTHORING-1C повторно
сверяет artifact, trusted context и package checksums и не recompiles source
после confirmation. Modified source/package, stale revision, другой actor/org
или изменённый request fail closed до `importDraft`.

AUTHORING-1B не импортирует lifecycle artifact. AUTHORING-1C добавляет только
trusted orchestration над existing `BotLifecycleApplicationService`: exact
import/status, persisted validation, package-owned Scenario V1/V2 execution и
evidence, create/activate/status test deployment. Migration и отдельная loader
persistence не добавлены. AUTHORING-1C не вызывает publish/promote/rollback;
existing lifecycle остаётся их единственным mutation owner в AUTHORING-1D.

### Controlled production and rollback

AUTHORING-1D не расширяет authoring source. Отдельный trusted operator boundary
принимает exact confirmation/session и server-issued AUTHORING-1C result,
повторно проверяет package/import/test-deployment evidence и только затем
вызывает lifecycle publish и production promotion. Rollback принимает internal
production evidence, проверяет persisted target deployment, imported package
pair и current active production; arbitrary/foreign/stale selector fail closed.
Restart и exact retry используют lifecycle `existing` semantics. Customer/AI
source не получает lifecycle IDs или deployment authority; automatic promotion
не существует. A → B → A сохраняет historical immutable deployments и не
переписывает pinned artifacts/flows. Migration и второй lifecycle не добавлены.
