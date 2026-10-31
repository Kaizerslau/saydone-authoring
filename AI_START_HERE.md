# Начало работы внешнего ИИ с SayDone Language

Создавайте полный C2 source, начиная с `language "saydone-r2/1"`, registry и
project. Используйте только синтаксис из текущих документов этого набора и
стабильные логические keys. Не создавайте JSON Bundle, runtime UUID, callback
payload, provider secret, URL оплаты, SQL или внутренние артефакты.

Порядок чтения:

1. [README](README.md);
2. [язык и базовый диалог](docs/authoring/c2/LANGUAGE.md);
3. [операции](docs/authoring/c2/CORE_OPERATIONS.md);
4. [внешние маршруты](docs/authoring/c2/ROUTING.md);
5. [DataField, HTTP и schedules](docs/authoring/c2/DATA_AND_INTEGRATIONS.md);
6. [примеры](docs/authoring/c2/examples/README.md).

Для оплаты используйте только описанный проверенный one-time path из
`hosted-checkout.c2` и `dynamic-checkout.c2`. One-time checkout не означает
поддержку recurring subscription, Closed Club membership, Telegram join
approval/removal, entitlement reminders либо automatic expiry/revoke. Если
сценарий требует такую lifecycle-композицию, укажите ограничение и не
придумывайте callbacks, операции или provider API.

Перед ответом проверьте, что every named call находится на одной строке,
references существуют, `message.button` имеет stable `key`, а provider-specific
условие написано явно. Если нужная capability не описана, не угадывайте её:
укажите ограничение вместо неподдерживаемого синтаксиса.

## Один Project — один полный C2

Для нового Project все Funnel можно писать сразу в одном C2-документе.
Один project и несколько funnel образуют общую composition; не создавайте
обязательные независимые исходники или версии каждой Funnel.
Практический workflow и отличие от legacy: [LANGUAGE.md](docs/authoring/c2/LANGUAGE.md#единый-project-и-несколько-funnel).
[Полный пример sales/support](docs/authoring/c2/examples/multi-funnel-routing.c2)
показывает локальные переходы и cross-funnel route через transient блоки.
Сохранение, проверка exact revision, публикация и operational Enable раздельны.
Legacy данные и механизмы совместимости сохраняются.
