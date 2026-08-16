# Controlled HTTP Egress V1

Статус: `IMPLEMENTED / VERIFIED / HTTP-1 DONE`

SayDone получит generic controlled outbound HTTP capability через existing BotSpec `call → registered public operation/capability`. Raw `fetch`, HTTP primitive в flow/runtime, arbitrary inbound callbacks, JS/SQL/eval не допускаются.

Реализованный HTTP-1A source/package contract задаёт stable `httpResources`
keys, absolute public `https:` endpoint, ровно `GET`/`POST`, allowlisted static
headers, flat DataField request mappings и typed JSON-pointer response mappings
в writable `flow`/`user` Data Fields. Generic `call` использует только
`HTTP_REQUEST` и literal resource key; compiler выводит `http.egress`, physical
effect key и SourceMap evidence. Secrets не входят в source/package: там
разрешён только opaque logical credential reference.

HTTP-1A выполняет pure syntax/shape validation и не делает DNS/network I/O.
HTTP-1B реализует secure resolution, redirect/timeout/size policy, encrypted
organization-scoped SecretStore lifecycle, durable request-value snapshot и
atomic typed completion через existing effect/outbox/worker. Resolver проверяет
все адреса и transport подключается только к validated address, сохраняя
Host/SNI. HTTP-1C добавил internal Test Request через тот же trusted credential,
request construction, DNS/SSRF, transport, parser и mapping path. В отличие от
worker он выполняет одну попытку без outbox, retry и DataField mutation.

Implementation обязана fail closed для SSRF, private/localhost/metadata networks, DNS rebinding, redirects, schemes, headers, timeouts, response size, retries/idempotency и log redaction. Конкретная library/technology этим direction contract не выбирается.

## Typed outcomes and Test Request

HTTP не является opaque `request → maybe exception`. Initial typed failure
kinds: `network`, `timeout`, `http_status`, `response_invalid`, `mapping_failed`.
Exact retry/idempotency semantics определяются HTTP-1 contract.

До deployment reusable HTTP Resource должен иметь first-class `Test Request`.
Safe inspector показывает status, allowlisted response headers, parsed JSON
shape без scalar values, DataField mapping proposal и validation errors без
secret leakage. Mapping diagnostic показывает requested path и bounded
available alternatives, а не только `mapping failed`. Raw response,
credentials, Authorization/API-key/cookie material, resolved addresses и
provider/network artifacts не входят в inspector DTO.

Response mappings пишут только в canonical Data Fields. HTTP-1 не добавляет
inbound arbitrary callbacks или отдельную variable system.
