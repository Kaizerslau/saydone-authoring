# Executable public examples V1

Статус: `CURRENT / AUTHORING-2B`

Each `.bundle.json` is a complete `SayDoneAuthoringBundleV1` and is read directly
by the public-kit drift test through strict parse → compile → inspect → validate
→ ScenarioV2 → coverage → qualification. They contain no runtime IDs, credentials
or live provider access.

| Example | Covers |
|---|---|
| `simple-menu.bundle.json` | stable keys, semantic choice/url, aliases, acceptance and qualification |
| `lead-application.bundle.json` | Telegram lead qualification: DataFields, typed input/retry/timeout, Conditions, RuleSet, SafeTransform, static Subflow, content/assets, contact, HTTP and triggers |
| `lead-application.patch.template.json` | non-standalone Patch repair fragment; the drift test binds its placeholders to the exact snapshot revision/checksum before validation |
| `schedule-cancel.fragment.json` | non-standalone static schedule/cancel pattern; the drift test compiles it in a minimal valid Bundle |

Do not copy the placeholder bindings from a Patch template. Obtain revision and
checksum only from the current preview/snapshot as described in the
[chat protocol](../AUTHORING_CHAT_PROTOCOL_V1.md).
