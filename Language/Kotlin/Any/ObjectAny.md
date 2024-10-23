<details>
  <summary>Эквивалентное отношение Object и Any</summary>

В Kotlin все классы по умолчанию наследуются от Any, который является аналогом Object в Java. Вот примеры использования некоторых методов Object в Kotlin:

1. equals()

В Kotlin оператор == используется для проверки структурного равенства, что эквивалентно вызову equals().

val str1 = "Hello"
val str2 = "Hello"

println(str1 == str2) // true

2. hashCode()

Вы можете вызвать hashCode() напрямую, если вам это нужно.

val str = "Hello"
println(str.hashCode())

3. toString()

Метод toString() можно переопределить в вашем классе.

class Person(val name: String) {
override fun toString(): String {
return "Person(name=$name)"
}
}

val person = Person("Alice")
println(person.toString()) // Person(name=Alice)

4. getClass()

В Kotlin можно использовать ::class или javaClass для получения информации о классе.

val str = "Hello"
println(str::class) // class kotlin.String
println(str.javaClass) // class java.lang.String
доступ можно использовать для получения информации о классе.

val str = "Hello"
println(str::class) // class kotlin.String
println(str.javaClass) // class java.lang.String




// Пример использования
val person = Person("Alice")
println(person.toString()) // Person(name=Alice)

5. clone()

Клонирование объектов в Kotlin обычно выполняется вручную или с использованием data class и метода copy().

data class Person(val name: String, val age: Int)

val person1 = Person("Alice", 30)
val person2 = person1.copy()

println(person1 == person2) // true
Эти примеры показывают, как использовать и переопределять методы, аналогичные методам Object в Java, в Kotlin.

</details>