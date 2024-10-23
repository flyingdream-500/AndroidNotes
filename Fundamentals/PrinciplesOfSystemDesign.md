### ООП
* https://devcolibri.com/%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-%D0%BE%D0%BE%D0%BF-%D0%B8-%D1%81-%D1%87%D0%B5%D0%BC-%D0%B5%D0%B3%D0%BE-%D0%B5%D0%B4%D1%8F%D1%82/

Наследование, композиция, агрегация

* https://javarush.com/groups/posts/1967-otnoshenija-mezhdu-klassami-nasledovanie-kompozicija-i-agregirovanie-

Пример инкапсуляции в Java:
* https://stackoverflow.com/questions/586363/why-is-super-super-method-not-allowed-in-java

<details>
  <summary>Агрегация и композиция на примере Android</summary>
В Android-разработке агрегация и композиция часто используются для управления зависимостями между различными компонентами приложения. Вот примеры для каждой концепции:

**Агрегация**

**Пример агрегации в Android: Использование репозитория в ViewModel**

 В этом примере `ViewModel` использует `Repository`, который содержит логику работы с данными, но `Repository` может существовать независимо от `ViewModel`.

```kotlin
// Repository
class UserRepository(private val apiService: ApiService) {
    fun getUser(userId: String): LiveData<User> {
        // Логика получения данных
    }
}

// ViewModel
class UserViewModel(private val userRepository: UserRepository) : ViewModel() {
    val userLiveData: LiveData<User> = userRepository.getUser("user_id")

    // Другие методы
}

// Activity or Fragment
class UserActivity : AppCompatActivity() {
    private lateinit var userViewModel: UserViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)

        // Предположим, что UserRepository создан где-то еще
        val userRepository = UserRepository(ApiService.create())
        userViewModel = ViewModelProvider(this, UserViewModelFactory(userRepository))
            .get(UserViewModel::class.java)

        userViewModel.userLiveData.observe(this, Observer { user ->
            // Обновление UI
        })
    }
}
```

Здесь `UserViewModel` использует `UserRepository`, но `UserRepository` может существовать независимо от `UserViewModel`. `UserRepository` предоставляет данные, но не зависит от `UserViewModel` и может быть использован в других частях приложения.

**Композиция**

**Пример композиции в Android: Фрагмент и его компоненты**

В этом примере `Fragment` содержит несколько дочерних представлений, таких как `RecyclerView` и `Button`. Эти компоненты управляются жизненным циклом `Fragment`, и их создание и уничтожение управляются `Fragment`.

```kotlin
// Fragment
class UserListFragment : Fragment() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: UserAdapter

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_user_list, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        recyclerView = view.findViewById(R.id.recycler_view)
        adapter = UserAdapter()
        recyclerView.adapter = adapter

        // Инициализация и настройка RecyclerView
    }

    override fun onDestroyView() {
        super.onDestroyView()
        // Очистка ресурсов, если необходимо
    }
}

// Adapter
class UserAdapter : RecyclerView.Adapter<UserAdapter.UserViewHolder>() {
    // Реализация адаптера
}
```

В этом примере `UserListFragment` содержит `RecyclerView` и `UserAdapter`. Жизненный цикл этих компонентов управляется `UserListFragment`. Если `UserListFragment` уничтожается, все его дочерние компоненты также уничтожаются, что делает это примером композиции.

**Различия**

- **Агрегация**: `UserViewModel` и `UserRepository` имеют слабую связь. `UserRepository` может существовать независимо от `UserViewModel` и быть использован в других местах.

- **Композиция**: `UserListFragment` содержит `RecyclerView` и `UserAdapter`, которые полностью управляются жизненным циклом `Fragment`. Если `Fragment` уничтожается, то и все его дочерние компоненты также уничтожаются.

Эти примеры иллюстрируют, как можно использовать агрегацию и композицию в Android для управления зависимостями и структурированием компонентов приложения.

</details>

Полиморфизм - развернуто:

* https://ievetrov.ru/kotlin-%D1%81-%D0%BD%D1%83%D0%BB%D1%8F-%D1%83%D1%80%D0%BE%D0%BA%D0%B8/%D1%83%D1%80%D0%BE%D0%BA-18-%D0%BE%D0%BE%D0%BF-%D0%BF%D0%BE%D0%BB%D0%B8%D0%BC%D0%BE%D1%80%D1%84%D0%B8%D0%B7%D0%BC-3-%D1%82%D0%B8%D0%BF%D0%B0-ad-hoc-subtyping-parametric/

### SOLID
* https://otus.ru/nest/post/1820/

<details>
  <summary>SOLID principles in Context</summary>

Анализируя архитектуру Context в Android через призму принципов SOLID, можно выделить следующие моменты:

**1. Single Responsibility Principle (Принцип единственной ответственности)**

Принцип единственной ответственности гласит, что каждый класс должен иметь одну единственную причину для изменения, то есть выполнять одну задачу. Context в Android не вполне соответствует этому принципу, так как он выполняет множество различных функций:

- Доступ к ресурсам (например, getResources()).
- Управление службами (например, getSystemService()).
- Запуск новых Activity (например, startActivity()).
- Работа с файловой системой (например, openFileOutput()).

Такая многозадачность нарушает SRP, поскольку класс Context отвечает за множество разных аспектов работы приложения.

**2. Open/Closed Principle (Принцип открытости/закрытости)**

Принцип открытости/закрытости предполагает, что классы должны быть открыты для расширения, но закрыты для модификации.

В Android классы, реализующие Context (например, Activity, Application), обычно расширяют базовый функционал через наследование или композицию. Однако сам класс Context является абстрактным и определяет интерфейс, который нужно реализовать. В этом смысле Context соответствует OCP, поскольку его функциональность можно расширять (через наследование), не изменяя сам класс.

**3. Liskov Substitution Principle (Принцип подстановки Барбары Лисков)**

Принцип Лисков подразумевает, что объекты базового класса должны заменяться объектами его подклассов без нарушения работы программы.

В контексте Android, классы, которые наследуют Context (например, Activity, Service, Application), обычно соблюдают этот принцип. Например, Activity может использоваться там, где требуется Context, без нарушений в работе программы. Таким образом, LSP в большинстве случаев соблюдается.

**4. Interface Segregation Principle (Принцип разделения интерфейсов)**

Принцип разделения интерфейсов подразумевает, что клиенты не должны зависеть от интерфейсов, которые они не используют.

Context объединяет множество различных функций в одном интерфейсе, что нарушает этот принцип. Например, Context предоставляет методы для работы с ресурсами, службами, файловой системой и т.д., хотя конкретному классу может быть нужна только часть этих функций.

**5. Dependency Inversion Principle (Принцип инверсии зависимостей)**

Принцип инверсии зависимостей требует, чтобы высокоуровневые модули не зависели от низкоуровневых модулей, а оба типа модулей зависели от абстракций.

В Android Context часто передается в классы через конструкторы или методы, что соответствует принципу инверсии зависимостей. Однако, поскольку Context предоставляет конкретную реализацию, это может привести к проблемам при тестировании или повторном использовании кода, если не использовать абстракции.

**Итог**

Хотя Context является важной частью Android и предоставляет множество полезных функций, его архитектура не полностью соответствует принципам SOLID. Нарушения, особенно касающиеся SRP и ISP, являются результатом того, что Context объединяет в себе множество обязанностей и функций, которые могут быть полезны, но в то же время делают его перегруженным и трудным для тестирования.
***

</details>


<details>
  <summary>SOLID в Android</summary>

Принципы SOLID — это набор пяти принципов объектно-ориентированного дизайна, которые помогают создавать гибкие и поддерживаемые системы. Вот как каждый из этих принципов может быть применен на практике в Android-разработке:

**1. Принцип единственной ответственности (Single Responsibility Principle, SRP)**

**Принцип:** Каждый класс должен иметь только одну причину для изменений, то есть один уровень ответственности.

**Пример в Android:**

```kotlin
// Пример плохого кода, нарушающего SRP
class UserProfileActivity : AppCompatActivity() {
    private lateinit var userProfile: UserProfile

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user_profile)

        // Загрузка данных
        userProfile = loadUserProfile()
        
        // Отображение данных
        displayUserProfile(userProfile)
        
        // Сохранение данных
        saveUserProfile(userProfile)
    }

    private fun loadUserProfile(): UserProfile {
        // Логика загрузки профиля пользователя
    }

    private fun displayUserProfile(profile: UserProfile) {
        // Логика отображения профиля пользователя
    }

    private fun saveUserProfile(profile: UserProfile) {
        // Логика сохранения профиля пользователя
    }
}

// Пример хорошего кода, следуя SRP
class UserProfileActivity : AppCompatActivity() {
    private lateinit var userProfile: UserProfile
    private val userProfileManager = UserProfileManager()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user_profile)

        userProfile = userProfileManager.loadUserProfile()
        displayUserProfile(userProfile)
    }

    private fun displayUserProfile(profile: UserProfile) {
        // Логика отображения профиля пользователя
    }
}

class UserProfileManager {
    fun loadUserProfile(): UserProfile {
        // Логика загрузки профиля пользователя
    }

    fun saveUserProfile(profile: UserProfile) {
        // Логика сохранения профиля пользователя
    }
}
```

В хорошем коде `UserProfileActivity` отвечает только за отображение профиля пользователя, а `UserProfileManager` отвечает за управление данными профиля.

**2. Принцип открытости/закрытости (Open/Closed Principle, OCP)**

**Принцип:** Классы должны быть открыты для расширения, но закрыты для модификации.

**Пример в Android:**

```kotlin
// Пример плохого кода, нарушающего OCP
class NotificationService {
    fun sendNotification(notificationType: String, message: String) {
        if (notificationType == "EMAIL") {
            sendEmail(message)
        } else if (notificationType == "SMS") {
            sendSms(message)
        }
        // Добавление нового типа уведомления требует изменения этого класса
    }

    private fun sendEmail(message: String) {
        // Логика отправки email
    }

    private fun sendSms(message: String) {
        // Логика отправки SMS
    }
}

// Пример хорошего кода, следуя OCP
interface Notification {
    fun send(message: String)
}

class EmailNotification : Notification {
    override fun send(message: String) {
        // Логика отправки email
    }
}

class SmsNotification : Notification {
    override fun send(message: String) {
        // Логика отправки SMS
    }
}

class NotificationService {
    fun sendNotification(notification: Notification, message: String) {
        notification.send(message)
    }
}
```

В хорошем коде добавление нового типа уведомления не требует изменения `NotificationService`. Достаточно создать новый класс, реализующий интерфейс `Notification`.

**3. Принцип подстановки Лисков (Liskov Substitution Principle, LSP)**

**Принцип:** Объекты должны быть заменяемыми их подтипами без нарушения правильности программы.

**Пример в Android:**

```kotlin
// Пример плохого кода, нарушающего LSP
open class Button {
    open fun click() {
        // Логика нажатия кнопки
    }
}

class ToggleButton : Button() {
    override fun click() {
        // Логика переключения
    }
}

fun performClick(button: Button) {
    button.click()
}

// Пример хорошего кода, следуя LSP
open class Button {
    open fun click() {
        // Логика нажатия кнопки
    }
}

class ToggleButton : Button() {
    override fun click() {
        // Логика переключения
    }
}

fun performClick(button: Button) {
    button.click() // Работает как с Button, так и с ToggleButton
}
```

В хорошем коде, если `ToggleButton` заменяет `Button`, поведение функции `performClick` остается корректным.

**4. Принцип разделения интерфейса (Interface Segregation Principle, ISP)**

**Принцип:** Клиенты не должны зависеть от интерфейсов, которые они не используют.

**Пример в Android:**

```kotlin
// Пример плохого кода, нарушающего ISP
interface UserActions {
    fun login()
    fun logout()
    fun resetPassword()
}

class User : UserActions {
    override fun login() {
        // Логика входа
    }

    override fun logout() {
        // Логика выхода
    }

    override fun resetPassword() {
        // Логика сброса пароля
    }
}

// Пример хорошего кода, следуя ISP
interface Login {
    fun login()
    fun logout()
}

interface PasswordReset {
    fun resetPassword()
}

class User : Login, PasswordReset {
    override fun login() {
        // Логика входа
    }

    override fun logout() {
        // Логика выхода
    }

    override fun resetPassword() {
        // Логика сброса пароля
    }
}
```

В хорошем коде интерфейсы разделены на более специализированные, и `User` реализует только те методы, которые ему действительно нужны.

**5. Принцип инверсии зависимостей (Dependency Inversion Principle, DIP)**

**Принцип:** Модули высшего уровня не должны зависеть от модулей низшего уровня. Оба типа модулей должны зависеть от абстракций.

**Пример в Android:**

```kotlin
// Пример плохого кода, нарушающего DIP
class UserService {
    private val userRepository = UserRepository()

    fun getUser(userId: String): User {
        return userRepository.getUser(userId)
    }
}

class UserRepository {
    fun getUser(userId: String): User {
        // Логика получения пользователя
    }
}

// Пример хорошего кода, следуя DIP
interface UserRepository {
    fun getUser(userId: String): User
}

class UserService(private val userRepository: UserRepository) {
    fun getUser(userId: String): User {
        return userRepository.getUser(userId)
    }
}

class LocalUserRepository : UserRepository {
    override fun getUser(userId: String): User {
        // Логика получения пользователя
    }
}
```

В хорошем коде `UserService` зависит от абстракции (`UserRepository`), а не от конкретной реализации. Это позволяет легко заменять реализации, например, для использования разных источников данных.

Эти примеры показывают, как можно применять принципы SOLID для создания более гибких, модульных и легко поддерживаемых приложений в Android.

</details>