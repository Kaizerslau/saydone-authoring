# Публичный индекс возможностей, событий и операций V1

<a id="capabilities"></a>

Статус: `CURRENT / выведен и проверен по каноническим реестрам`

Этот индекс описывает только публичную поверхность авторского описания. Он не предоставляет
конфигурацию внутренних частей Billing, Closed Club, Broadcasts или времени выполнения.

## Конструкции `BotSpec`

`entry`, `blocks`, `actions`, типизированный `input`, `on.text`, статические
значения параметра запуска, статические команды, `fallback`, `conditions`, `subflows`,
`ruleSets`, `DataField`, `SafeTransforms`, `content`, логические `assets`,
`httpResources` и семантические действия сообщения. Точная форма находится в [публичном справочнике](reference/README.md).

`START`, `TEXT_RECEIVED`, `COMMAND_RECEIVED` и `UNKNOWN_INPUT` не являются
публичными значениями `on.event`.

## Публичные события результата модулей

<a id="module-result-events"></a>

| Событие | Требуемая возможность |
|---|---|
| `ORDER_CREATED`, `ORDER_CREATE_FAILED` | `billing.orders` |
| `CHECKOUT_CREATED`, `CHECKOUT_CREATE_FAILED`, `PAYMENT_SUCCEEDED`, `PAYMENT_REQUIRES_RECONCILIATION` | `billing.checkouts` |
| `ASSET_DELIVERED`, `ASSET_DELIVERY_FAILED` | `files.deliver_asset` |
| `CLOSED_CLUB_PURCHASE_INITIATED`, `CLOSED_CLUB_PURCHASE_INITIATION_FAILED`, `CLOSED_CLUB_CHECKOUT_READY`, `CLOSED_CLUB_CHECKOUT_FAILED` | `closed-club.purchase` |

## Зарегистрированные операции `call`

| Операция | Требуемая возможность |
|---|---|
| `CREATE_ASSET_ORDER` | `billing.orders` |
| `CREATE_CHECKOUT` | `billing.checkouts` |
| `DELIVER_ASSET` | `files.deliver_asset` |
| `INITIATE_CLOSED_CLUB_PURCHASE` | `closed-club.purchase` |
| `ADD_CURRENT_USER_TO_SEGMENT`, `REMOVE_CURRENT_USER_FROM_SEGMENT` | `broadcasts.segments` |
| `HTTP_REQUEST` | `http.egress` |

Другие возможности, выводимые компилятором: `messaging.send_content`,
`analytics.record_event` и `jobs.schedule_event`. Ввод операции ограничен
каноническим реестром и контрактом; динамический или произвольный вызов модуля недоступен.

Псевдонимы кнопок и запланированные события выводятся из авторского исходного
описания и не являются глобальными событиями модулей.
