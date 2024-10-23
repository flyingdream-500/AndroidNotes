## Kotlin

Kotlin interview questions:
* https://habr.com/ru/articles/721084/

### Delegate


<details>
  <summary>Примеры использования делегатов в Kotlin в Android</summary>

В Kotlin для Android делегаты предоставляют мощный инструмент для упрощения кода и управления состоянием. Делегаты позволяют перемещать повторяющуюся логику из компонентов Android в отдельные функции или классы. Вот несколько примеров использования делегатов в Android:

1. **`lazy` для отложенной инициализации**

Одним из самых популярных делегатов в Kotlin является `lazy`. Он позволяет отложить инициализацию переменной до момента её первого использования. Это полезно в Android, когда необходимо инициализировать ресурсоемкие объекты только по мере необходимости.

Пример: Инициализация `View` в `Activity`

```kotlin
class MainActivity : AppCompatActivity() {

    private val textView: TextView by lazy {
        findViewById(R.id.my_text_view)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // textView будет инициализирован здесь, при первом обращении
        textView.text = "Hello, World!"
    }
}
```

2. **`observable` для наблюдения за изменениями свойств**

Делегат `observable` позволяет отслеживать изменения свойства и выполнять какое-либо действие при его изменении. Это полезно для обновления UI при изменении данных.

#### Пример: Автоматическое обновление UI при изменении данных

```kotlin
class UserViewModel : ViewModel() {

    var username: String by Delegates.observable("<no name>") { _, oldValue, newValue ->
        Log.d("UserViewModel", "Username changed from $oldValue to $newValue")
        // Можно обновить UI здесь
    }
}
```

3. **`vetoable` для валидации изменений свойств**

Делегат `vetoable` позволяет отменить изменение значения свойства, если оно не удовлетворяет определенным условиям.

#### Пример: Валидация ввода данных

```kotlin
class User {
    var age: Int by Delegates.vetoable(0) { _, _, newValue ->
        newValue >= 0 // Разрешаем изменение, только если возраст неотрицательный
    }
}

fun main() {
    val user = User()
    user.age = 25  // Примет значение 25
    println(user.age)  // Вывод: 25

    user.age = -1  // Изменение отклонено
    println(user.age)  // Вывод: 25
}
```

4. **`Map` делегаты для работы с `Bundle` или `Intent`**

Классический случай использования делегатов в Android — доступ к параметрам `Intent` или `Bundle`. Делегат `Map` позволяет легко работать с такими объектами, как `Bundle`, превращая их в свойство.

#### Пример: Доступ к аргументам `Fragment` через делегаты

```kotlin
class MyFragment : Fragment() {

    private val args: Bundle by lazy { arguments ?: Bundle() }

    val userId: String by args
    val userName: String by args

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // userId и userName можно использовать прямо здесь
        Log.d("MyFragment", "User ID: $userId, User Name: $userName")
        return inflater.inflate(R.layout.fragment_my, container, false)
    }
}
```

Для использования этого подхода необходимо написать расширение для `Bundle`, позволяющее автоматически извлекать значения:

```kotlin
operator fun Bundle.getValue(thisRef: Any?, property: KProperty<*>): String {
    return getString(property.name) ?: ""
}
```

5. **Создание собственных делегатов**

Вы можете создавать свои делегаты для выполнения специфических задач.

#### Пример: Делегат для работы с `SharedPreferences`

```kotlin
class SharedPreferencesDelegate<T>(
    private val preferences: SharedPreferences,
    private val key: String,
    private val defaultValue: T
) : ReadWriteProperty<Any?, T> {

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return when (defaultValue) {
            is String -> preferences.getString(key, defaultValue as String) as T
            is Int -> preferences.getInt(key, defaultValue as Int) as T
            is Boolean -> preferences.getBoolean(key, defaultValue as Boolean) as T
            else -> throw IllegalArgumentException("Unsupported type")
        }
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        with(preferences.edit()) {
            when (value) {
                is String -> putString(key, value)
                is Int -> putInt(key, value)
                is Boolean -> putBoolean(key, value)
                else -> throw IllegalArgumentException("Unsupported type")
            }
            apply()
        }
    }
}

class MySettings(preferences: SharedPreferences) {
    var userToken: String by SharedPreferencesDelegate(preferences, "userToken", "")
    var isLoggedIn: Boolean by SharedPreferencesDelegate(preferences, "isLoggedIn", false)
}
```

Этот пример показывает, как создать делегат для удобного доступа и хранения данных в `SharedPreferences`.

Эти примеры демонстрируют, как делегаты могут быть использованы для упрощения кода, улучшения читаемости и удобства работы с состояниями и ресурсами в Android приложениях на Kotlin.

</details>

### Type

<details>
  <summary>Почему lateinit var недоступен для примитивных типов в Kotlin?</summary>

- Kotlin имеет различие между примитивными типами (Int, Boolean и т.д.) и их объектными аналогами (Integer, Boolean и т.д.). lateinit работает с объектами, а не с примитивами. Использование lateinit с примитивными типами потребовало бы их автоматической упаковки (boxing) в объектные аналоги, что противоречит его основному предназначению и увеличивало бы накладные расходы на упаковку и распаковку.

- Примитивные типы в Kotlin (такие как Int, Boolean и другие) всегда имеют значения по умолчанию. Например, для Int это 0, для Boolean — false. Таким образом, необходимость откладывать инициализацию для таких типов отсутствует, поскольку у них уже есть валидные значения по умолчанию.

- Можно использовать делегаты, такие как Delegates.notNull() для примитивных типов, если важно отложить инициализацию.

`import kotlin.properties.Delegates`

`var number: Int by Delegates.notNull<Int>()`

</details>

### Value class


<details>
  <summary>Примеры</summary>

В Kotlin начиная с версии 1.5 появилась концепция **`value class`** (классов-значений). Это классы, которые представляют данные внутри себя как одно значение, но по сути работают как примитивные типы для повышения производительности. Они особенно полезны для создания типов-обёрток, где нужен дополнительный смысл для примитивных данных, но без накладных расходов на создание полноценного объекта.

Вот несколько примеров использования `value class` в различных сценариях:

1. **Типы-обёртки для валидации данных**

Иногда требуется создать специальный тип для данных с определёнными ограничениями, вместо того чтобы просто передавать примитивный тип, например, `String`.

****Пример: Email****

```kotlin
@JvmInline
value class Email(val value: String) {
    init {
        require(value.contains("@")) { "Email must contain '@' symbol" }
    }
    
    override fun toString(): String = value
}

fun sendEmail(email: Email) {
    println("Sending email to ${email.value}")
}

fun main() {
    val email = Email("user@example.com")
    sendEmail(email) // Отправка корректного email
}
```

- **Преимущества**: `Email` обёртывает строку и добавляет валидацию внутри конструктора, проверяя, что строка действительно является корректным email-адресом.

2. **Улучшение читаемости и безопасности типов**

Иногда вместо использования примитивных типов лучше применять `value class` для повышения читаемости кода и предотвращения путаницы между различными значениями одного типа.

****Пример: Мили и Километры****

```kotlin
@JvmInline
value class Miles(val value: Double)

@JvmInline
value class Kilometers(val value: Double)

fun convertToKilometers(miles: Miles): Kilometers {
    return Kilometers(miles.value * 1.60934)
}

fun main() {
    val distanceInMiles = Miles(10.0)
    val distanceInKilometers = convertToKilometers(distanceInMiles)
    println("Distance in kilometers: ${distanceInKilometers.value}")
}
```

- **Преимущества**: Использование отдельных типов для миль и километров предотвращает случайные ошибки (например, путаницу между единицами измерения), и код становится более читаемым.

3. **Представление идентификаторов**

`value class` можно использовать для представления идентификаторов (ID), которые логически являются примитивами, но их нужно выделить как отдельный тип для удобства и безопасности.

****Пример: User ID****

```kotlin
@JvmInline
value class UserId(val value: Long)

fun getUserById(userId: UserId) {
    println("Fetching user with ID: ${userId.value}")
}

fun main() {
    val userId = UserId(12345L)
    getUserById(userId) // Безопасно передаём ID пользователя
}
```

- **Преимущества**: Теперь ID пользователя не может быть случайно перепутан с другими `Long` значениями, что делает код более надёжным.

4. **Оптимизация работы с единичными объектами**

Классы-значения могут быть полезны для представления небольших структур данных, где нужно избежать накладных расходов на создание полноценного объекта.

****Пример: Координаты****

```kotlin
@JvmInline
value class Coordinates(val value: Pair<Double, Double>) {
    val latitude: Double get() = value.first
    val longitude: Double get() = value.second
}

fun printCoordinates(coordinates: Coordinates) {
    println("Latitude: ${coordinates.latitude}, Longitude: ${coordinates.longitude}")
}

fun main() {
    val coordinates = Coordinates(40.7128 to -74.0060)
    printCoordinates(coordinates)
}
```

- **Преимущества**: `Coordinates` хранит координаты как пару значений, но при этом не требует создания полноценного объекта, что улучшает производительность.

5. **Оптимизация работы с `inline` функциями**

Классы-значения могут использоваться с `inline` функциями для оптимизации производительности, избегая создания лишних объектов.

****Пример: Оптимизация работы с `inline` функцией****

```kotlin
@JvmInline
value class Age(val value: Int)

inline fun isAdult(age: Age): Boolean {
    return age.value >= 18
}

fun main() {
    val age = Age(20)
    println("Is adult: ${isAdult(age)}")
}
```

- **Преимущества**: `inline` функция в сочетании с `value class` избегает создания объекта для `Age`, что снижает накладные расходы при вызове.

****Важные особенности `value class`:****
1. **Один параметр**: `value class` может содержать только одно свойство.
2. **Нет явной структуры объектов**: В рантайме классы-значения ведут себя как примитивные типы, что позволяет экономить память.
3. **Ограничения**:
    - Нельзя использовать `init` блок для сложной логики (нельзя изменять внутреннее состояние после создания).
    - `value class` не может иметь вторичных конструкторов.
    - Нельзя создавать экземпляры через `new`, так как они работают как примитивы.

****Заключение****

`value class` отлично подходят для улучшения читаемости, повышения безопасности типов и оптимизации работы с примитивными данными, позволяя работать с ними как с объектами, но без накладных расходов на их создание.


</details>

### Decompilation

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

### Exception
* https://habr.com/ru/articles/763518/

### Scope functions

<details>
  <summary>Реализация scope-функции let:</summary>

`inline fun <T, R> T.let(block: (T) -> R): R {`
`return block(this)`
`}`

</details>

### Object and Any

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

### Typealias

<details>
  <summary>Область видимости</summary>

- Можно прописать модификаторы доступа для видимости

</details>

### Inner and Nested class

<details>
  <summary>Особенности</summary>

- Inner класс можно вызвать лишь внутри outer-класса(так как он имеет доступ к его private полям)
- Inner и Nested классы нужны для инкапсуляции логики внутри класса
- Пример nested-класса - ErrorCodes внутри NetworkManager

</details>


### Inlining

Inline, noinline, crossinline, reified:
* https://habr.com/ru/articles/775120/
* https://dzen.ru/a/Y7qwPlHuW0YfaRtK
* https://apptractor.ru/info/techhype/voprosy-s-sobesedovaniy-kogda-nelzya-ispolzovat-inline-funktsii.html


## Java

### JMM

Видео:
* https://www.youtube.com/watch?v=iqatGX1HmPo&ab_channel=VKTeam

Статьи:
* https://habr.com/ru/articles/685518/

### JVM
Stack & Heap:
* https://topjava.ru/blog/stack-and-heap-in-java
  String Pool:
* https://topjava.ru/blog/rukovodstvo-po-string-pool-v-java
  Garbage Collector:
* https://ziginsider.github.io/Garbage_Collector_Java/
* https://itsobes.ru/JavaSobes/kak-rabotaet-sborka-musora/

## Common
### Collections
ArrayList:
* https://habr.com/ru/articles/128269/
* https://javarush.com/groups/posts/2472-podrobnihy-razbor-klassa-arraylist
* https://javarush.com/groups/posts/1936-rabota-arraylist-v-kartinkakh--

LinkedList:
* https://javarush.com/groups/posts/1938-linkedlist
* https://habr.com/ru/articles/127864/

HashMap:
* https://habr.com/ru/articles/128017/

### Sequences
* https://androidschool.ru/2024/01/11/sequence-api-vs-collection/

### Generics

* Видео: https://www.youtube.com/watch?v=mh6LV9aBypo
* Статья: https://habr.com/ru/companies/redmadrobot/articles/301174/
* Еще статья: https://skillbox.ru/media/code/dzheneriki-v-java-dlya-tekh-kto-postarshe/