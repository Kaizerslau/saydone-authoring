# Authoring primitives V1

Статус: `CURRENT / AUTHORING-2B`

Use this page to choose a relevant contract, not as a replacement for one. Each
item below is already implemented and has a bounded public source form.

## Structure and data

- `entry` is the ordinary logical START target. `blocks` have stable logical
  keys; `saveState:true` is a persistent user position, while `saveState:false`
  is a transient action sequence and cannot wait for input/handlers/fallback.
- Transitions use static `run`; actions are ordered `send`, `set`, `unset`,
  `track`, `schedule`, `cancelSchedule`, `call` and `applyRuleSet` only.
- `DataFieldV1` is the one canonical value identity. `flow` is temporary,
  `user` is persistent, `bot` is immutable configuration, `system` is read-only.
  Missing differs from `null`, `""`, `0` and `false`; use explicit `unset`.
  Keys never shadow by scope. Types/formats/constraints are in the
  [DataFields contract](../architecture/BUSINESS_DATA_FIELDS_V1.md).

## Logic and inputs

- Conditions are pure structured predicates; they cannot mutate or call services.
- A `ruleSets` table is ordered `first_match`, writes one typed writable field and
  has explicit `otherwise` value or `no_match` semantics.
- SafeTransforms occur only inside explicit `set`, accept flat literal/`$from`
  arguments, are typed/deterministic and have no side effects.
- Static named `subflows` have declared inputs/outputs, entry/result and a static
  continuation; recursion and dynamic targets are forbidden.
- Inputs support `text`, `number`, `email`, `phone` and `telegram_contact`.
  Validation/retry/exhaustion/timeout are bounded. Specific alias/`on.text`
  routing precedes generic input (D-054).

Exact shapes: [BotSpec](BOT_SPEC_V1_CONTRACT.md) and
[bounded language](../architecture/AUTHORING_LANGUAGE_V1.md).

## Content, messages and schedules

- Content uses stable logical keys and `plain`, `telegram_html` safe subset or
  `telegram_markdown_v2`; registered DataFields render deterministically.
- Assets are logical slots. Binary, base64, storage URL and Telegram `file_id`
  are not BotSpec source.
- Buttons are either legacy `run`/`set`/`aliases` or semantic `choice`, `url`,
  `request_contact`, `request_location`; do not mix the modes in one `send`.
  `request_location` is outbound-only V1.
- `schedule` and `cancelSchedule` use static IDs/targets. Generated schedule
  event identities are compiler-owned.

See [content contract](../architecture/CONTENT_VERSIONING_V1.md) and
[BotSpec](BOT_SPEC_V1_CONTRACT.md).

## HTTP, triggers and modules

- `httpResources` are static controlled HTTPS resources. Credential references
  are configured separately; request/response mappings use DataFields. No raw
  fetch, arbitrary headers or credential value is authoring source. Outcomes are
  typed (`network`, `timeout`, `http_status`, `response_invalid`, `mapping_failed`).
- `startPayloads` are exact static routes; `on.command` and `on.text` are static.
  Precedence is start payload → ordinary entry; then alias/specific text → generic
  input → fallback → unhandled. Raw runtime input events are forbidden in
  `on.event` (D-053).
- BotSpec describes an interactive flow. It may call a registered public operation
  or handle a public module result event, but does not configure Closed Club,
  Billing or Broadcasts domain internals.

Use [HTTP contract](../architecture/HTTP_EGRESS_V1.md),
[capability index](AI_CAPABILITY_EVENT_OPERATION_INDEX_V1.md) and
[BotSpec](BOT_SPEC_V1_CONTRACT.md).
