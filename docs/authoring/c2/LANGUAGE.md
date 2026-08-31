# Язык C2

Документ начинается с `language`, `registry` и `project`; далее объявляются
content, fields, HTTP-ресурсы, rulesets, funnels и subflows. Блок содержит
операции и переходы.

В текущем production parser и formatter named operation, control call и
declaration должны быть полностью сбалансированы на одной строке. Многострочное
оформление аргументов пока не поддерживается. Это текущая граница реализации,
а не обещание неизменного свойства языка.

```c2
language "saydone-r2/1"
registry "r2-oracle-1"
project hello(name: "Приветствие", locale: "ru")
content greeting(text: "Здравствуйте", required: [])

funnel main {
  block greeting {
    message.send(content: greeting)
  }
}
```

См. полный [minimal.c2](examples/minimal.c2).
