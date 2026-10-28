# Внешние маршруты SayDone Language

`external.route` сопоставляет exact normalized semantic input с logical block.
Product composition Funnel и глобальная точная маршрутизация определены
принятым product architecture решением; runtime semantics
ниже остаются authority. Provider/integration ограничиваются только явно
написанными условиями, а технический route index не является ручным назначением
Funnel к каналу.
Ниже показаны изолированные top-level declarations: в текущей production grammar
весь named call, включая аргументы, должен быть записан на одной строке.

```text
external.route gift(kind: text, value: gift, target: sales.open_gift)
```

Это generic route: provider и integration не участвуют. Provider-aware вариант
может ограничиться `provider`, если достаточно проверить принадлежность каналу:

```text
external.route telegram_event(kind: event, value: registration, provider: telegram, target: events.registration)
```

`integration` добавляют только для привязки к конкретной integration, а
зарегистрированный typed fact — только когда route действительно опирается на
provider-specific факт. Например, Telegram `/start gift` выражается так:

```text
external.route telegram_gift(kind: text, value: gift, provider: telegram, fact: telegram.bot_command, factVersion: 1, factValue: "{\"arguments\":\"gift\",\"command\":\"start\",\"startParameterCandidate\":\"gift\"}", target: sales.open_gift)
```

Полные copy-ready examples, проверяемые production parser, доступны в
[examples](./examples/).

Unknown fact/version/payload fail closed через `C2ProviderFactRegistryV2`.
Нельзя придумывать `deeplink=true`: используются только факты, доказанные и
зарегистрированные provider adapter.

Lifecycle связывает logical target с server-owned stable `blockId` через
`bindC2ExternalRouteIndexV2`. Runtime index не содержит исходник SayDone Language и сортируется по
`routeKey`; source order не имеет смысла.

Внутри funnel `next`, `on.event` и `on.command` принимают только local block
key. Для cross-funnel перехода event/command сначала направляют в local
transient block с `saveState: false`, а уже он выполняет
`route other_funnel.block`.

Разрешение строгое:

- один match → CN-1D `ResolvedExactTargetApplicationServiceV1`;
- ноль matches + active flow → existing current-block path;
- ноль matches без active flow → `unhandled`;
- два и более matches → `ambiguous`, без fallback и исполнения.

Priority, first match, latest wins, hidden specificity, regex, contains, fuzzy
и similarity отсутствуют. Доказуемый overlap отклоняется до compile; оставшаяся
runtime ambiguity fail closed.

Telegram `/start` и `/start gift` — ordinary normalized input, не `START`, не
restart и не mandatory entry. Callback candidate не допускается в generic
external route: без active authorized continuation он остаётся stale/ignored.

В переходной production-схеме Telegram ingress включает neutral V2 routing
только когда для current deployment существует опубликованный immutable route
index. Отсутствующий index означает, что V1 artifact продолжает существующий
legacy path; это не создаёт route по умолчанию. Callback независимо от наличия
index всегда остаётся в отдельном G1 authorization path.
