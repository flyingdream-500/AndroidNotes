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