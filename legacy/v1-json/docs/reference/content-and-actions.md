# Контент, сообщения и семантические действия

<a id="content"></a>

<a id="capabilities"></a>

<a id="call-operations"></a>

`send` содержит встроенный `text` либо зарегистрированный `content`; не
передавайте разметку или JSON поставщика. Ключ `content` статичен, `format`:
`plain`, `telegram_html` или `telegram_markdown_v2`. Переменные —
зарегистрированные `DataField`; подстановка детерминированно экранирует
динамические значения. Произвольные HTML-шаблонные выражения, двоичные данные,
`base64`, адрес хранилища и Telegram `file_id` не входят в исходное описание.

```json
{
  "type": "send",
  "content": "application_summary",
  "buttons": [{ "text": "Оставить контакт", "action": { "type": "request_contact" } }]
}
```

Семантические значения `type` кнопки: `choice`, `url`, `request_contact`,
`request_location`. `choice` имеет статический бизнес-маршрут и необязательную
запись в канонический `DataField`;
`url` принимает ограниченный безопасный абсолютный HTTPS URL;
`request_contact` не зависит от поставщика и сочетается с типизированным вводом `telegram_contact`;
`request_location` в V1 только исходящий и не заявляет входящий захват
значения. Не смешивайте семантические кнопки с прежним форматом
`run`/`set`/`aliases` в одном `send`; не
используйте `callback_data`.

Расписание статично и ограничено:

```json
{ "type": "schedule", "id": "followup", "after": "30m", "run": "followup", "replace": true }
```

Используйте `cancelSchedule` с тем же логическим id. Идентификаторы расписания
создаёт SayDone.
