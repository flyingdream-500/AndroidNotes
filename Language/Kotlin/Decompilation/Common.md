<details>
  <summary>Во что компилируются файлы с функциями в Kotlin?</summary>

Если в файле находятся функции верхнего уровня (не принадлежащие какому-либо классу), то они компилируются в статические методы класса. Название этого класса, как правило, создается на основе имени файла с добавлением суффикса Kt. Например, файл Utils.kt с функциями верхнего уровня компилируется в класс UtilsKt.

</details>

<details>
  <summary>Во что компилируется companion object в Kotlin?</summary>

Companion object в Kotlin компилируется в обычный статический внутренний класс с именем Companion, который содержит статические члены.

`MyClass.Companion.printMessage();`

</details>

<details>
  <summary>Где лучше объявить const переменную: в companion object или вне класса?</summary>

- В companion object: Если переменная относится к классу логически, например, это параметр конфигурации для этого класса или она будет использоваться только в контексте этого класса.
- На уровне файла: Если переменная более глобальная и не связана с конкретным классом, лучше вынести её на уровень файла. Это также помогает избегать ненужного использования класса, когда это не требуется.

</details>