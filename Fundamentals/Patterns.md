## Паттерны проектирования
* https://javarush.com/groups/posts/496-patternih-proektirovanija-v-java
* https://javarush.com/groups/posts/584-patternih-proektirovanija

In Android:

<details>
  <summary>Порождающие паттерны в Android</summary>

В Android-разработке порождающие паттерны проектирования часто применяются для управления созданием объектов и обеспечения гибкости кода. Вот несколько примеров порождающих паттернов в контексте Android:

**Singleton**

**Цель:** Обеспечить существование только одного экземпляра класса и предоставить к нему глобальную точку доступа.

**Пример:** Работа с базой данных или сетью, где вам нужен только один экземпляр, например, `SharedPreferences` или `Retrofit` клиент.

```kotlin
class MyDatabase private constructor(context: Context) {
    init {
        // Инициализация базы данных
    }

    companion object {
        @Volatile
        private var instance: MyDatabase? = null

        fun getInstance(context: Context): MyDatabase {
            return instance ?: synchronized(this) {
                instance ?: MyDatabase(context).also { instance = it }
            }
        }
    }
}
```

В этом примере `MyDatabase` гарантирует, что создается только один экземпляр и предоставляет глобальную точку доступа к нему.

**Factory Method**

**Цель:** Определить интерфейс для создания объекта, но позволить подклассам решать, какой класс инстанцировать.

**Пример:** Создание различных типов `View` в зависимости от типа данных.

```kotlin
abstract class ViewFactory {
    abstract fun createView(context: Context): View
}

class ButtonFactory : ViewFactory() {
    override fun createView(context: Context): View {
        return Button(context)
    }
}

class TextViewFactory : ViewFactory() {
    override fun createView(context: Context): View {
        return TextView(context)
    }
}

// Использование
fun createView(factory: ViewFactory, context: Context): View {
    return factory.createView(context)
}

// Пример использования в активности
val button = createView(ButtonFactory(), this)
val textView = createView(TextViewFactory(), this)
```

В этом примере `ViewFactory` позволяет создавать различные виды `View` без необходимости в жесткой привязке к конкретному типу `View`.

**Builder**

**Цель:** Позволяет создавать сложные объекты шаг за шагом.

**Пример:** Конфигурация и создание объекта `AlertDialog`.

```kotlin
class CustomDialog private constructor(
    private val title: String,
    private val message: String,
    private val positiveButtonText: String,
    private val negativeButtonText: String
) {
    class Builder(private val context: Context) {
        private var title: String = ""
        private var message: String = ""
        private var positiveButtonText: String = ""
        private var negativeButtonText: String = ""

        fun setTitle(title: String) = apply { this.title = title }
        fun setMessage(message: String) = apply { this.message = message }
        fun setPositiveButtonText(text: String) = apply { this.positiveButtonText = text }
        fun setNegativeButtonText(text: String) = apply { this.negativeButtonText = text }

        fun build(): CustomDialog {
            return CustomDialog(title, message, positiveButtonText, negativeButtonText)
        }
    }
}

// Использование
val dialog = CustomDialog.Builder(context)
    .setTitle("Title")
    .setMessage("Message")
    .setPositiveButtonText("OK")
    .setNegativeButtonText("Cancel")
    .build()
```

В этом примере `CustomDialog.Builder` упрощает процесс создания `CustomDialog`, позволяя поэтапно задавать необходимые параметры.

**Prototype**

**Цель:** Клонировать объекты, чтобы создавать новые экземпляры, используя существующий объект в качестве образца.

**Пример:** Создание новых объектов `Bitmap` на основе существующего изображения.

```kotlin
abstract class BitmapPrototype : Cloneable {
    abstract fun clone(): BitmapPrototype

    // Другие методы для работы с изображением
}

class ConcreteBitmap : BitmapPrototype() {
    // Реализация методов работы с изображением

    override fun clone(): ConcreteBitmap {
        return super.clone() as ConcreteBitmap
    }
}

// Использование
val originalBitmap = ConcreteBitmap()
val clonedBitmap = originalBitmap.clone()
```

В этом примере `ConcreteBitmap` может клонировать себя, чтобы создать новый объект на основе существующего.

Эти примеры показывают, как порождающие паттерны проектирования можно использовать в Android-разработке для управления созданием объектов, улучшения гибкости и уменьшения зависимости между компонентами системы.

</details>

<details>
  <summary>Структурные паттерны в Android</summary>

Структурные паттерны проектирования помогают организовать компоненты и классы, чтобы они могли работать вместе более эффективно. Вот несколько примеров структурных паттернов, применяемых в Android-разработке:

**Adapter**

**Цель:** Позволяет объектам с несовместимыми интерфейсами работать вместе, путем преобразования интерфейса одного класса в интерфейс, который ожидает другой класс.

**Пример:** Использование `RecyclerView` с адаптером для отображения данных.

```kotlin
class MyAdapter(private val items: List<String>) : RecyclerView.Adapter<MyAdapter.ViewHolder>() {

    inner class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textView: TextView = itemView.findViewById(R.id.textView)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_view, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.textView.text = items[position]
    }

    override fun getItemCount(): Int = items.size
}

// Использование в активности
val recyclerView: RecyclerView = findViewById(R.id.recyclerView)
recyclerView.layoutManager = LinearLayoutManager(this)
recyclerView.adapter = MyAdapter(listOf("Item 1", "Item 2", "Item 3"))
```

В этом примере `MyAdapter` адаптирует данные для отображения в `RecyclerView`, преобразуя их в элементы интерфейса.

**Bridge**

**Цель:** Разделяет абстракцию и реализацию, позволяя их изменять независимо друг от друга.

**Пример:** Использование `View` и `Renderer` для рендеринга графики.

```kotlin
interface Renderer {
    fun render(data: String)
}

class ConsoleRenderer : Renderer {
    override fun render(data: String) {
        println(data)
    }
}

abstract class Graph(val renderer: Renderer) {
    abstract fun draw()
}

class BarGraph(renderer: Renderer) : Graph(renderer) {
    override fun draw() {
        renderer.render("Drawing Bar Graph")
    }
}

// Использование
val renderer = ConsoleRenderer()
val barGraph = BarGraph(renderer)
barGraph.draw()
```

В этом примере `Graph` и `Renderer` разделены, что позволяет изменять их независимо, например, изменять тип рендеринга без изменения графических данных.

**Composite**

**Цель:** Позволяет клиентам работать с индивидуальными объектами и композициями объектов единообразно.

**Пример:** Работа с иерархией представлений в Android.

```kotlin
interface Component {
    fun draw()
}

class Leaf : Component {
    override fun draw() {
        println("Drawing Leaf")
    }
}

class Composite : Component {
    private val children = mutableListOf<Component>()

    fun add(child: Component) {
        children.add(child)
    }

    override fun draw() {
        println("Drawing Composite")
        children.forEach { it.draw() }
    }
}

// Использование
val leaf1 = Leaf()
val leaf2 = Leaf()
val composite = Composite()
composite.add(leaf1)
composite.add(leaf2)
composite.draw()
```

В этом примере `Composite` содержит несколько объектов `Component`, которые могут быть индивидуальными объектами или другими композициями объектов.

**Decorator**

**Цель:** Позволяет динамически добавлять новые функциональности к объекту.

**Пример:** Добавление функциональности к кнопке в Android.

```kotlin
interface Button {
    fun click()
}

class BasicButton : Button {
    override fun click() {
        println("Button clicked")
    }
}

class ButtonDecorator(private val button: Button) : Button {
    override fun click() {
        button.click()
        addExtraFunctionality()
    }

    private fun addExtraFunctionality() {
        println("Extra functionality added")
    }
}

// Использование
val button = BasicButton()
val decoratedButton = ButtonDecorator(button)
decoratedButton.click()
```

В этом примере `ButtonDecorator` добавляет дополнительную функциональность к существующему объекту `Button` без изменения его кода.

**Facade**

**Цель:** Предоставляет простой интерфейс к сложной системе классов.

**Пример:** Упрощение доступа к нескольким сервисам через один класс.

```kotlin
class NetworkService {
    fun connect() {
        println("Connecting to network...")
    }
}

class DatabaseService {
    fun query() {
        println("Querying database...")
    }
}

class Facade {
    private val networkService = NetworkService()
    private val databaseService = DatabaseService()

    fun performOperation() {
        networkService.connect()
        databaseService.query()
    }
}

// Использование
val facade = Facade()
facade.performOperation()
```

В этом примере `Facade` предоставляет упрощенный интерфейс для работы с `NetworkService` и `DatabaseService`, скрывая их сложность от клиента.

Эти примеры иллюстрируют, как структурные паттерны проектирования помогают организовать и упростить работу с компонентами и классами в Android-приложениях.

</details>

<details>
  <summary>Поведенченские паттерны в Android</summary>
Поведенческие паттерны проектирования описывают взаимодействие объектов и классов, а также распределение ответственности между ними. В Android они могут быть использованы для упрощения сложных взаимодействий и повышения гибкости кода. Вот несколько примеров поведенческих паттернов в Android:

1. **Observer (Наблюдатель)**:
    - **Пример**: В Android паттерн Наблюдатель используется, например, в архитектуре MVVM с LiveData. Объекты подписываются на обновления данных, и когда данные изменяются, подписчики автоматически уведомляются и могут обновить UI.
    - **Применение**: `LiveData`, `ViewModel`, `BroadcastReceiver`, `ContentObserver`.

2. **Command (Команда)**:
    - **Пример**: Этот паттерн часто используется в реализации команд в приложении, когда действия пользователя должны быть отложены или выполнены в определенной последовательности. В Android можно видеть пример этого паттерна в реализации `Runnable` и `Handler`.
    - **Применение**: Отложенные задачи, такие как `AsyncTask` (устаревший) и `ExecutorService`.

3. **Strategy (Стратегия)**:
    - **Пример**: В Android паттерн Стратегия может быть применен для реализации различных способов обработки данных или поведения компонентов. Например, при создании различных алгоритмов сортировки данных для отображения в списке.
    - **Применение**: Различные адаптеры для RecyclerView или `InputMethodManager`.

4. **State (Состояние)**:
    - **Пример**: Этот паттерн часто используется для управления состояниями в Android-приложениях. Например, в процессе аутентификации пользователь может находиться в разных состояниях: "Неавторизован", "Авторизован", "В процессе аутентификации". В зависимости от состояния, поведение приложения меняется.
    - **Применение**: Управление состояниями пользователя или UI-элементов (например, `ProgressBar`, `Button`).

5. **Memento (Хранитель)**:
    - **Пример**: Паттерн Memento позволяет сохранять и восстанавливать состояние объекта. В Android это можно наблюдать при сохранении состояния активностей и фрагментов, когда, например, приложение сворачивается или уничтожается.
    - **Применение**: `onSaveInstanceState()`, `SharedPreferences`, базы данных для сохранения данных.

6. **Chain of Responsibility (Цепочка обязанностей)**:
    - **Пример**: В этом паттерне запрос передается через цепочку обработчиков до тех пор, пока не будет обработан. В Android можно увидеть его использование в системе `TouchEvent`, где события касания проходят через цепочку элементов, пока не найдется обработчик.
    - **Применение**: Обработка событий в UI, `ViewGroup`, `TouchEvent`.

Эти паттерны помогают структурировать код, делая его более модульным и легко поддерживаемым, что важно для разработки Android приложений.

</details>