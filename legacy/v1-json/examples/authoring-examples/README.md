# Исполняемые публичные примеры V1

Статус: `CURRENT / AUTHORING-2B`

Каждый `.bundle.json` — полный `SayDoneAuthoringBundleV1`; публичная проверка
дрейфа читает его напрямую через строгий разбор → компиляция → проверка состава → валидация
→ `ScenarioV2` → покрытие → квалификация. В примерах нет идентификаторов времени
выполнения, учётных данных или доступа к живому поставщику.

| Пример | Что показывает |
|---|---|
| `simple-menu.bundle.json` | стабильные ключи, семантические `choice`/`url`, псевдонимы, приёмочные сценарии и квалификация |
| `lead-application.bundle.json` | Telegram-бот для заявки: `DataField`, типизированный ввод с повтором и тайм-аутом, `Conditions`, `RuleSet`, `SafeTransform`, статический `Subflow`, контент и активы, контакт, HTTP и триггеры |
| `minimal/routing.bundle.json` | `command`, `fallback` и статический параметр запуска в отдельном небольшом Bundle |
| `minimal/typed-input.bundle.json` | один `DataField` и типизированный числовой ввод |
| `minimal/logic.bundle.json` | Conditions, `RuleSet`, `Subflow` и `SafeTransform` в отдельном квалифицированном пути |
| `minimal/http.bundle.json` | контролируемый `HTTP_REQUEST` и один типизированный результат |
| `repairs/*.json` | детерминированно полученные invalid → `RepairReportV1` → bound Patch → fixed семьи для entry и `DataField` |
| `COMPLETENESS_MATRIX_V1.json` | проверяемое сопоставление public primitive, fixture, canonical contract и repair family |
| `lead-application.patch.template.json` | отдельный фрагмент `Patch` для исправления; проверка дрейфа привязывает его заполнители к точной версии и контрольной сумме снимка перед валидацией |
| `schedule-cancel.fragment.json` | отдельный статический шаблон `schedule`/`cancel`; проверка дрейфа компилирует его в минимальный корректный `Bundle` |

Не копируйте привязки заполнителей из шаблона `Patch`. Получайте версию и
контрольную сумму только из текущей проверки или описания, как указано в
[протоколе диалога](../AUTHORING_CHAT_PROTOCOL_V1.md).
