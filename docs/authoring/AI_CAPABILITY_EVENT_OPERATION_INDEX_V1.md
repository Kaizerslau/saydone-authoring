# Public capability, event and operation index V1

Статус: `CURRENT / derived-and-checked against canonical registries`

Этот индекс описывает только public authoring surface. Он не предоставляет
конфигурацию internals Billing, Closed Club, Broadcasts или runtime.

## BotSpec primitives

`entry`, `blocks`, `actions`, typed `input`, `on.text`, static start payloads,
static commands, `fallback`, `conditions`, `subflows`, `ruleSets`, `DataFields`,
`SafeTransforms`, `content`, logical `assets`, `httpResources` и semantic message
actions. Точная shape и restrictions — в [BotSpec contract](BOT_SPEC_V1_CONTRACT.md).

Raw `START`, `TEXT_RECEIVED`, `COMMAND_RECEIVED` и `UNKNOWN_INPUT` не являются
public `on.event` values (D-053). Их routing остаётся compiler/runtime-owned.

## Public module result events

| Event | Required capability |
|---|---|
| `ORDER_CREATED`, `ORDER_CREATE_FAILED` | `billing.orders` |
| `CHECKOUT_CREATED`, `CHECKOUT_CREATE_FAILED`, `PAYMENT_SUCCEEDED`, `PAYMENT_REQUIRES_RECONCILIATION` | `billing.checkouts` |
| `ASSET_DELIVERED`, `ASSET_DELIVERY_FAILED` | `files.deliver_asset` |
| `CLOSED_CLUB_PURCHASE_INITIATED`, `CLOSED_CLUB_PURCHASE_INITIATION_FAILED`, `CLOSED_CLUB_CHECKOUT_READY`, `CLOSED_CLUB_CHECKOUT_FAILED` | `closed-club.purchase` |

## Registered `call` operations

| Operation | Required capability |
|---|---|
| `CREATE_ASSET_ORDER` | `billing.orders` |
| `CREATE_CHECKOUT` | `billing.checkouts` |
| `DELIVER_ASSET` | `files.deliver_asset` |
| `INITIATE_CLOSED_CLUB_PURCHASE` | `closed-club.purchase` |
| `ADD_CURRENT_USER_TO_SEGMENT`, `REMOVE_CURRENT_USER_FROM_SEGMENT` | `broadcasts.segments` |
| `HTTP_REQUEST` | `http.egress` |

Other compiler-derived capabilities are `messaging.send_content`,
`analytics.record_event` and `jobs.schedule_event`. Operation input is bounded
by its canonical registry and contract; no dynamic or arbitrary module call is
available.

Package-owned button aliases and scheduled events are derived from the authored
source/package and are not global module events. See
[event/action registry contract](EVENT_ACTION_REGISTRY_CONTRACT.md) for their
provenance boundary.
