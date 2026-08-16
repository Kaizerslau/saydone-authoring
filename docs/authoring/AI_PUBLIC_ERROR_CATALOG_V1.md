# Public repair/error catalog V1

Статус: `CURRENT / stable bounded codes only`

`RepairReportV1` returns code, source path, severity, affected preview gate and
redacted bounded context. Do not copy raw exceptions into a Patch. The exact
source rule remains the linked canonical contract.

| Code | Domain/path | Meaning and safe repair | Contract |
|---|---|---|---|
| `authoring.bundle_version` | Bundle root | `bundleVersion` is unsupported; use version `1`. | [Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md) |
| `authoring.bundle_unknown_artifact` | `/artifacts` | Bundle contains an unsupported artifact; keep only the documented canonical artifacts. | [Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md) |
| `authoring.source_forbidden_field` | source path | Source attempted authority or forbidden material; remove it and let SayDone supply trusted context. | [Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md) |
| `authoring.source_plaintext_secret` | source path | Source contains a credential-like value; replace it with the allowed static resource reference where applicable. | [Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md) |
| `authoring.patch_stale_base` | `/baseRevision` | Patch does not bind the current snapshot; obtain its current revision/checksum and regenerate the small Patch. | [Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md) |
| `authoring.patch_duplicate_target` | `/changes` | More than one change targets the same source item; emit one unambiguous change. | [Bundle/Patch](AUTHORING_BUNDLE_PATCH_V1.md) |
| `botspec.schema_unknown_property` | BotSpec path | Unknown source property; remove it or use the exact documented primitive. | [BotSpec](BOT_SPEC_V1_CONTRACT.md) |
| `botspec.capability_unknown_event` | `/blocks/.../on` | `on.event` is not a public module result event; use a supported event/capability or another source primitive. | [Capability index](AI_CAPABILITY_EVENT_OPERATION_INDEX_V1.md) |
| `datafield.value_type` | DataField value path | Literal/reference type does not match the declared canonical DataField; correct the declared source value without coercion. | [DataFields](../architecture/BUSINESS_DATA_FIELDS_V1.md) |
| `input.field_type_mismatch` | typed input path | Input capture type is incompatible with the destination canonical DataField; use the matching declared input/value type. | [BotSpec](BOT_SPEC_V1_CONTRACT.md) |
| `botspec.condition_shape` | condition path | Condition is not a bounded documented Conditions AST shape; use the published typed form. | [Language](../architecture/AUTHORING_LANGUAGE_V1.md) |
| `subflow.call_indirect_reference` | subflow call path | Subflow target must be one static logical key, not a dynamic reference. | [Language](../architecture/AUTHORING_LANGUAGE_V1.md) |
| `acceptance.unknown_block` | acceptance path | Acceptance refers to no logical block; use an existing authored logical block id. | [Acceptance](BOT_ACCEPTANCE_SPEC_V1_DIRECTION.md) |
| `compiler.lowering_ambiguous_text` | source text route | Multiple same-precedence text routes overlap; make exact routes unambiguous. | [Compiler](BOT_AUTHORING_COMPILER_CONTRACT_V1.md) |
| `content.variable_unregistered` | content variable path | Placeholder is not a registered canonical DataField; register it or remove the placeholder. | [BotSpec](BOT_SPEC_V1_CONTRACT.md) |
| `transform.unknown_operation` | transform path | Transform is outside the bounded V1 catalog; choose a documented SafeTransform. | [Language](../architecture/AUTHORING_LANGUAGE_V1.md) |
| `transform.input_type` | transform path | Transform input does not match the documented typed operation; change the explicit source value or operation. | [Language](../architecture/AUTHORING_LANGUAGE_V1.md) |
| `ruleset.unknown_reference` | RuleSet path | RuleSet reference is missing from the static catalog; use a declared static key. | [Language](../architecture/AUTHORING_LANGUAGE_V1.md) |
| `trigger.start_payload` | start payload path | Start payload is not a bounded static exact value; use one valid static exact route. | [BotSpec](BOT_SPEC_V1_CONTRACT.md) |
| `http.endpoint` | HTTP resource path | Endpoint violates the bounded static HTTPS resource contract; use a valid declared HTTPS endpoint. | [HTTP egress](../architecture/HTTP_EGRESS_V1.md) |
