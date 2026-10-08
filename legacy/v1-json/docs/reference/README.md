# Справочник публичных конструкций SayDone V1

Это краткий слой для создания исходного JSON. Он не описывает реализацию,
хранение, развёртывание или историю разработки. Для каждого раздела сначала
прочитайте точную форму, затем один минимальный пример. Каноническая проверка
SayDone остаётся окончательной.

Точная структурная форма доступна в [JSON Schema](../schemas/README.md).
Прохождение схемы подтверждает только структуру; окончательное решение принимает
каноническая проверка SayDone.

| Нужно описать | Откройте |
|---|---|
| Каркас бота, `entry`, блоки и действия | [structure.md](structure.md) |
| Канонические значения | [data-fields.md](data-fields.md) |
| Ввод, команды, текст, `fallback`, параметр запуска | [inputs-and-routing.md](inputs-and-routing.md) |
| `Conditions`, `RuleSet`, `Subflow`, `SafeTransform` | [logic.md](logic.md) |
| Текст, `content`, кнопки, расписания | [content-and-actions.md](content-and-actions.md) |
| Контролируемый HTTP-ресурс | [http.md](http.md) |
| Приёмочные сценарии и квалификация | [acceptance-and-qualification.md](acceptance-and-qualification.md) |
| `Bundle`, `Patch` и `RepairReportV1` | [bundle-patch-repair.md](bundle-patch-repair.md) |

Не создавайте `BotPackageV1`, сгенерированные ID, `callback_data`, JSON
поставщика, SQL, учётные данные, JS/eval или данные HTTP-запроса. Используйте
только статические логические ключи и JSON-формы этого справочника.
