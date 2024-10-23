## View

## Fragment
Серия статей про неочевдные вещи с фрагментами:
* https://habr.com/ru/companies/tinkoff/articles/690134/

## Fragment manager
**The fragment lifecycle during a fragment transaction**
(https://scribe.rip/@mohit2656422/android-the-fragment-lifecycle-during-a-fragment-transaction-188c2df05238)

[FragmentManager](https://developer.android.com/reference/androidx/fragment/app/FragmentManager) is the class responsible for performing actions on your app's fragments, such as adding, removing, or replacing them and adding them to the back stack.

You can access the FragmentManager from an activity or from a fragment.

Fragments can host one or more child fragments. Inside a fragment, you can get a reference to the FragmentManager that manages the fragment's children through [getChildFragmentManager()](https://developer.android.com/reference/androidx/fragment/app/Fragment#getChildFragmentManager()). If you need to access its host FragmentManager, you can use [getParentFragmentManager()](https://developer.android.com/reference/androidx/fragment/app/Fragment#getParentFragmentManager()).

Given this setup, you can think about each host as having a FragmentManager associated with it that manages its child fragments. This is illustrated in figure 2 along with property mappings between supportFragmentManager, parentFragmentManager, and childFragmentManager.
![image](https://github.com/flyingdream-500/AndroidNotes/assets/57061500/a1eb3b05-3d4e-4c1c-8723-5c27ab1735dc)

**Perform a transaction**

```kotlin
supportFragmentManager.commit {
   replace<ExampleFragment>(R.id.fragment_container)
   setReorderingAllowed(true)
   addToBackStack("name") // Name can be null
}
```

[setReorderingAllowed(true)](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#setReorderingAllowed(boolean)) optimizes the state changes of the fragments involved in the transaction so that animations and transitions work correctly.

Calling [addToBackStack()](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#addToBackStack(java.lang.String)) commits the transaction to the back stack. The user can later reverse the transaction and bring back the previous fragment by tapping the Back button. If you added or removed multiple fragments within a single transaction, all those operations are undone when the back stack is popped. The optional name provided in the addToBackStack() call gives you the ability to pop back to a specific transaction using [popBackStack()](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#popBackStack(java.lang.String,%20int)).

Find an existing fragment
You can get a reference to the current fragment within a layout container by using [findFragmentById()](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#findFragmentById(int)). Use findFragmentById() to look up a fragment either by the given ID when inflated from XML or by the container ID when added in a FragmentTransaction.
```kotlin
val fragment: ExampleFragment =
        supportFragmentManager.findFragmentById(R.id.fragment_container) as ExampleFragment
```

Alternatively, you can assign a unique tag to a fragment and get a reference using [findFragmentByTag()](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#findFragmentByTag(java.lang.String)). You can assign a tag using the android:tag XML attribute on fragments that are defined within your layout or during an add() or replace() operation within a FragmentTransaction.

```kotlin
val fragment: ExampleFragment =
        supportFragmentManager.findFragmentByTag("tag") as ExampleFragment
```

**Support multiple back stacks**(https://developer.android.com/guide/fragments/fragmentmanager#multiple-back-stacks)

In some cases, your app might need to support multiple back stacks. A common example is if your app uses a bottom navigation bar. FragmentManager lets you support multiple back stacks with the saveBackStack() and restoreBackStack() methods. These methods let you swap between back stacks by saving one back stack and restoring a different one.

_**saveBackStack()**_ works similarly to calling popBackStack() with the optional name parameter: the specified transaction and all transactions after it on the stack are popped. The difference is that saveBackStack() [saves the state](https://developer.android.com/guide/fragments/saving-state) of all fragments in the popped transactions.

```kotlin
supportFragmentManager.commit {
  replace<ExampleFragment>(R.id.fragment_container)
  setReorderingAllowed(true)
  addToBackStack("replacement")
}
supportFragmentManager.saveBackStack("replacement")
```

**Note: You can use saveBackStack() only with transactions that call setReorderingAllowed(true) so that the transactions can be restored as a single, atomic operation.**

You can call restoreBackStack() with the same name parameter to restore all of the popped transactions and all of the saved fragment states:

`supportFragmentManager.restoreBackStack("replacement")`

**Note: You can't use saveBackStack() and restoreBackStack() unless you pass an optional name for your fragment transactions with addToBackStack().**

Provide dependencies to your fragments (https://developer.android.com/guide/fragments/fragmentmanager#dependencies)


### RecyclerView
**Как происходит рендеринг экрана сообщений ВКонтакте** (https://habr.com/ru/companies/vk/articles/501988/)

**- setHasFixedSize (boolean)**
Когда размер RecyclerView постоянный и не зависит от элементов (грубо говоря, не wrap_content). Установка флага помогает немного повысить скорость у RecyclerView, чтобы он избежал лишних вычислений.

**- setNestedScrollingEnabled (boolean)**
Незначительная оптимизация, отключающая поддержку NestedScroll.Если нет CollapsingToolbar или других фич, зависящих от NestedScroll, можно смело выставить этот флаг в false.

**- setItemViewCacheSize (cache_size)**
Настройка внутреннего кэша RecyclerView.
* есть ViewHolder, отображаемый на экране;
* есть RecycledViewPool, хранящий ViewHolder;
* когда ViewHolder уходит с экрана — он помещается в RecycledViewPool.
  Есть еще промежуточный кеш. Он называется ItemViewCache. В чём его суть? Когда ViewHolder уходит с экрана, он помещается не в RecycledViewPool, а в промежуточный кеш (ItemViewCache). Все изменения в адаптере применяются как к видимым ViewHolder, так и к ViewHolder внутри ItemViewCache. А к ViewHolder внутри RecycledViewPool изменения не применяются.
  Через setItemViewCacheSize мы можем задать размер этого промежуточного кеша.
  Чем он больше, тем быстрее будет скролл на небольшие расстояния, но операции обновления будут выполняться дольше (из-за ViewHolder.onBind и т. д.).

**Отслеживание Adapter.onFailedToRecyclerView()**
У вас есть список, в котором какие-то элементы (или их часть) анимируются с альфой. В тот момент, когда View, будучи в процессе анимации, уходит с экрана, то она не уходит в RecycledViewPool. Почему? RecycledViewPool видит, что View сейчас анимируется за счёт флага View.hasTransientState, и просто её игнорирует. Поэтому в следующий раз при скролле вверх-вниз картинка не будет браться из RecycledViewPool, а создастся заново.
Самое правильное решение — когда ViewHolder уходит с экрана, нужно отменять все анимации.

![image](https://github.com/flyingdream-500/AndroidNotes/assets/57061500/a0fb4cab-6d80-4574-b681-1a70d95aa047)

**DiffUtil**
Выполнять DiffUtil на основном потоке немного затратно, потому что список может быть очень большим. Так что обычно вычисление DiffUtil запускается на фоновом потоке.

ListAdapter и AsyncListDiffer это делают за вас. ListAdapter расширяет обычный Adapter и запускает всё асинхронно — достаточно сделать submitList и весь расчёт изменений улетает на внутренний фоновый поток. ListAdapter умеет учитывать кейс частых обновлений: если его вызвать три раза подряд, он возьмёт только последний результат.

**Глобальный RecycledViewPool**

В стандартном подходе, когда зашли в чат и вышли из него, RecycledViewPool (и ViewHolder в нём) просто уничтожаются, и мы каждый раз тратим ресурсы создание ViewHolder.

Это можно решить глобальным RecycledViewPool:
* в рамках Application живёт RecycledViewPool как синглтон;
* переиспользуется на экране сообщений, когда пользователь ходит между экранами;
* устанавливается как RecyclerView.setRecycledViewPool(pool).


Recycler View

Видео: https://www.youtube.com/watch?v=2-TIYJRkypk&ab_channel=YandexforDevelopers
![image](https://github.com/flyingdream-500/AndroidNotes/assets/57061500/809ddc0a-700f-440a-b544-ec6b41375cf0)
![image](https://github.com/flyingdream-500/AndroidNotes/assets/57061500/d7d35849-7a6e-4437-b933-48099e956819)
![image](https://github.com/flyingdream-500/AndroidNotes/assets/57061500/0f5cd241-0a26-4342-8050-e8ca4f719e8d)
<img width="664" alt="Снимок экрана 2024-03-10 в 15 16 59" src="https://github.com/flyingdream-500/AndroidNotes/assets/57061500/824f02ca-8d85-4fa7-a700-da3107b599d6">


Anatomy of Recycler View: https://medium.com/android-news/anatomy-of-recyclerview-part-1-a-search-for-a-viewholder-404ba3453714
Delegate Adapter: https://habr.com/ru/articles/341738/


## Compose

### Optimization:
https://habr.com/ru/companies/ozontech/articles/742854/
https://habr.com/ru/articles/796437/

## Navigation
