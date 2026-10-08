# Контролируемый HTTP-ресурс

<a id="http-resources"></a>

<a id="http-call"></a>

`httpResources` объявляет статический ресурс; `call` вызывает только
зарегистрированную операцию `HTTP_REQUEST` с её статическим `resourceKey`.
Исходное описание никогда не содержит значение учётных данных, прямой `fetch`,
произвольные заголовки, сетевой результат или ID поставщика.

```json
{
  "httpResources": {
    "crm_submit": {
      "endpoint": "https://crm.example.test/v1/applications",
      "method": "POST",
      "idempotency": "provider_key",
      "credential": { "reference": "crm_api", "mode": "bearer" },
      "headers": { "content-type": "application/json" },
      "request": { "body": [{ "name": "company", "field": "company_name" }] },
      "response": { "contentType": "application/json", "mappings": [{ "path": "/id", "field": "external_application_id", "type": "text" }] }
    }
  }
}
```

`endpoint` должен быть обычной строкой JSON с абсолютным публичным адресом
`https:`: без Markdown, учётных данных, фрагмента, прямого IP-адреса или локального имени узла.
Допустимы только `GET`/`POST`; POST требует `provider_key`, GET использует
`none`. Заголовки только статические `accept`/`content-type`. Сопоставления —
ограниченные именованные поля в канонические `DataField` и типизированные пути JSON Pointer
к ответу; нельзя использовать шаблоны, неявное преобразование типов, повторные
записи или динамический ключ ресурса. Итоги:
`network`, `timeout`, `http_status`, `response_invalid`, `mapping_failed`.

Когда не применять: не описывайте в `BotSpec` входящий обратный вызов, произвольный
клиент API, поток аутентификации или живой тестовый запрос.
