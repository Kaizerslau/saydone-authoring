# Приёмочные сценарии и квалификация

<a id="acceptance"></a>

`acceptance` независимо описывает ожидаемое поведение бота; `qualification`
связывает бизнес-требования со сценариями. Оба используют только ключи `blocks`
и `DataField` из исходного описания.

```json
{
  "acceptanceVersion": 1,
  "scenarios": [{
    "key": "start", "description": "Старт открывает меню",
    "steps": [{ "type": "start" }, { "type": "expectBlock", "block": "menu" }]
  }]
}
```

Обычные значения `type` шага: `start`, `command`, `text`, `contact`, `button`,
публичный `event`, `scheduledEvent`, `processEffects`, `advanceClock`, `restart` и
проверки `expectBlock`, `expectTransition`, `expectContext`, `expectMessage`,
`expectEffects`, `expectBusinessEvent`, `expectSchedule`, `expectNoPendingWork`.
Каждый упомянутый блок, действие или `DataField` должен существовать в исходном
описании. Не используйте сгенерированные идентификаторы состояний, правил, эффектов,
обратных вызовов или базы данных либо
синтаксис `ScenarioV2`.

Используйте хотя бы один сценарий для каждого бизнес-пути, требуемого
`qualification`. Каноническая проверка преобразует `acceptance`, выполняет
`ScenarioV2`, вычисляет покрытие и проверяет `qualification`; исходное описание
не может добавлять сценарии в пакет после компиляции.
