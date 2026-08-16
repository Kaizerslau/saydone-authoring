# Исполняемые публичные примеры V1

Статус: `CURRENT / AUTHORING-2B`

Каждый `.bundle.json` — полный `SayDoneAuthoringBundleV1`; публичная проверка
дрейфа читает его напрямую через строгий разбор → compile → inspect → validate
→ ScenarioV2 → coverage → qualification. В примерах нет идентификаторов времени
выполнения, учётных данных или доступа к живому provider.

| Пример | Что показывает |
|---|---|
| `simple-menu.bundle.json` | stable keys, semantic choice/url, aliases, acceptance and qualification |
| `lead-application.bundle.json` | Telegram lead qualification: DataFields, typed input/retry/timeout, Conditions, RuleSet, SafeTransform, static Subflow, content/assets, contact, HTTP and triggers |
| `lead-application.patch.template.json` | non-standalone Patch repair fragment; the drift test binds its placeholders to the exact snapshot revision/checksum before validation |
| `schedule-cancel.fragment.json` | non-standalone static schedule/cancel pattern; the drift test compiles it in a minimal valid Bundle |

Не копируйте привязки заполнителей из шаблона Patch. Получайте revision и
checksum только из текущей проверки/описания, как указано в
[протоколе диалога](../AUTHORING_CHAT_PROTOCOL_V1.md).
