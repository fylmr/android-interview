 Вопросы к собеседованию по Android 

- [Java](#java)
- [Android](#android)
  - [Activity](#activity)
  - [Fragments](#fragments)
  - [Content provider](#content-provider)
  - [Services](#services)
  - [Broadcasts](#broadcasts)
  - [Custom Views](#custom-views)
  - [Другое](#%D0%B4%D1%80%D1%83%D0%B3%D0%BE%D0%B5)
- [Архитектура](#%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0)
- [Библиотеки](#%D0%B1%D0%B8%D0%B1%D0%BB%D0%B8%D0%BE%D1%82%D0%B5%D0%BA%D0%B8)
  - [Moxy](#moxy)
  - [RxJava](#rxjava)

# Java

- Виды референсов: https://habr.com/ru/post/169883/
  1. **SoftReference** — если GC видит что объект доступен только через цепочку soft-ссылок, то он удалит его из памяти, если памяти будет не хватать. Удобно для кеширования. 
  2. **WeakReference** — если GC видит что объект доступен только через цепочку weak-ссылок, то он удалит его из памяти. Применение: WeakHashMap. Это реализация Map<K,V> которая хранит ключ, используя weak-ссылку. И когда GC удаляет ключ с памяти, то удаляется вся запись с Map. 
  Близкое к Андроиду применение [в разделе WeakReference этой статьи](https://xakep.ru/2016/06/01/android-multithreading-1/#toc09.).
  3. **PhantomReference** — если GC видит что объект доступен только через цепочку phantom-ссылок, то он его удалит из памяти. После нескольких запусков GC. 
   Особенностей у этого типа ссылок две. Первая это то, что метод **get() всегда возвращает null**. Именно из-за этого PhantomReference имеет смысл использовать только вместе с ReferenceQueue. Вторая особенность – в отличие от SoftReference и WeakReference, GC добавит phantom-ссылку в ReferenceQueue после того как выполнится метод finalize(). То есть фактически, в отличии от SoftReference и WeakReference, объект еще есть в памяти.

- Volatile/Atomic
  - *volatile* помечает объект как доступный для нескольких тредов. Ожидаемо работает только для атомарных операций. Например, операция инкремента не атомарна, из-за чего процесс будет сбоить.
  - *Atomic*-классы использут compare and swap алгоритм: обновляет значение, только если предыдущее совпадает с ожидаемым old value.

- Описать Handler
  - Handler - это механизм, который позволяет работать с очередью сообщений между потоками. Он привязан к конкретному потоку и работает с его очередью. Handler умеет помещать сообщения в очередь. При этом он ставит самого себя в качестве получателя этого сообщения. И когда приходит время, система достает сообщение из очереди и отправляет его адресату (т.е. в Handler) на обработку.
  
- Абстрактные классы против интерфейсов
  - Переменные в интерфейсе по умолчанию final
  - В абстрактном классе могут использоваться модификаторы доступа, в интерфейсе по умолчанию всё public
  - Можно наследовать несколько интерфейсов, но только один класс

# Android

## Activity

- Жизненный цикл Activity
  - `OnCreate()` Создаётся view.
  - `OnStart()` Активити становится видно
  - `OnResume()` Активити становится доступным для ввода пользователя
  - `OnPause()` Активити видно, но недоступо для ввода пользователя (важно для многооконного режима)
  - `OnStop()` Активити больше не видно
  - `OnDestroy()` Активити уничтожается
  - `OnRestart()` Активити пересоздаётся, вызывается после уничтожения и перед созданием

- Какие методы вызываются при переходе между активити
  - A: onCreate, onStart, onResume
    A: *(переход)* onPause
    B: onCreate, onStart, onResume
    A: onStop
    B: *(обратный переход)* onPause
    A: onRestart, onStart, onResume
    B: onStop, onDestroy
   ![иллюстрация процесса](https://lh5.googleusercontent.com/-MMX3o4pdsd0/ToybUtq-EFI/AAAAAAAAAbw/ri5MQ1Jg5sI/s800/20111005_L0024_L_TwoActSchema.jpg) 

- Когда `onDestroy()` вызывается без `onPause()` и `onStop()`?
  - Если активити закрывается через метод `finish()` до начала onStart и onResume.

- Почему `setContentView()` располагают в `onCreate()`
  - Это тяжёлая операция, и выгоднее производить её только единожды, при создании активити.

- Launch modes
  - **Standard** При вызове, активити создаётся заново.
  - **SingleTop** Активити создаётся заново, только если она не вверху активити-стека. 
  - **SingleTask** Стек стирается до момента, пока эта активити не окажется наверху стека.
  - **SingleInstance** Похож на SingleTask, но при создании активити, она уйдёт в новый таск.

- Как очистить бэкстек при создании активити
  - Использовать флаг `FLAG_ACTIVITY_CLEAR_TOP`. 
  - Использовать `FLAG_ACTIVITY_CLEAR_TASK` и `FLAG_ACTIVITY_NEW_TASK` вместе.

- Разница между `FLAG_ACTIVITY_CLEAR_TASK` и `FLAG_ACTIVITY_CLEAR_TOP`
  - Первый очистит всё, что есть в таске, второй — только до той же активити, что и запускаемая.

- Как при пересоздании активити сохранить некоторый объект (кроме onSaveInstanceState)
  - Методы [onRetainCustomNonConfigurationInstance()](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onretaincustomnonconfigurationinstance) и 
[getLastCustomNonConfigurationInstance()](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#getlastcustomnonconfigurationinstance) класса FragmentActivity (или AppCompatActivity)
  - Если активити — просто держатель для фрагментов, лучше в самих фрагментах делать `setRetainInstance(true)` 

## Fragments

- Жизненный цикл Fragment
  - ![Жизненный цикл фрагмента](https://i.stack.imgur.com/fRxIQ.png)

- Отличия `commit()`, `commitNow()`, `commitAllowingStateLoss()` и `commitNowAllowingStateLoss()`
  - Если вызвать `commit()` после `onSaveInstanceState()`, система выкинет IllegalStateException ([подробнее о причинах и процессе](https://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html)). 
  - `commitAllowingStateLoss()` в этом случае не вызовет исключения, но состояние менеджера фрагментов может быть потеряно. 
  - `commit()` помещает транзакцию для выполнения, когда главный поток будет свободен. Для синхронного выполнения висящих транзакций можно использовать метод `executePendingTransactions()`. Альтернатива этому — метод `commitNow()`, который исполнит транзакцию сразу, однако не сможет поместить её в backstack ([подробнее](https://medium.com/@bherbst/the-many-flavors-of-commit-186608a015b1)).

## Content provider
  
- Зачем нужен Content Provider
  - Позволяет передавать данные между приложениями и процессами. Используется в связке с `ContentResolver`.

## Services

- Виды Service/IntentService
  - Часть приложения без UI
  - **Foreground** Выполняется в основном потоке. Во время исполнения показывает уведомление. Пример — аудиоплеер.
  - **Background** Выполняется в фоновом потоке. Начиная с API 26, приложения в фоне не могут создавать фоновые сервисы, решением в этом случае может быть `WorkManager`.
  - **Bound** Клиент-серверный подход: посылаем запросы, получаем результаты.  
  - **IntentService** Создаёт и работает в собственном потоке, выполняет работу из метода `onHandleIntent()`, после чего останавливается. Операции в `onHandleIntent()` не могут быть прерваны, связь с основным потоком отсутствует.  

- Жизненный цикл сервисов
  ![иллюстрация](https://developer.android.com/images/service_lifecycle.png?hl=ru)

- Метод `startService()`
  - startService не размножает экземпляры сервиса, а только создает очередь из заданий для него. И если IntentService просто выполнит все задания последовательно, одно за другим, то благодаря классу Service у нас появляется возможность запустить все задачи одновременно в независимых потоках. Достаточно в методе onStartCommand создавать новые потоки для каждой задачи.

- Остановка сервиса
  - Сервис, создaнный с помощью одноименного класса, будет жить, пока его принудительно не остановят. Сделать это можно либо откуда-то снаружи методом stopService, с указанием интента, которым он был запущен, либо внутри самого сервиса методом stopSelf. 
  - Как узнать, что сервис уже все сделал, особенно если мы поставили перед ним сразу несколько задач? В этом нам поможет параметр startId у метода onstartcommand — это порядковый номер каждого вызова сервиса, который увеличивается на единицу при каждом запуске. 
  - Создав новый поток, разумно завeршать его методом stopSelf, указывая startId в качестве параметра. С таким параметром ОС не будет сразу завершать сервис, а сравнит переданный идентификатор с тем, который был выдан при последнем запуске onStartCommand. Если они не равны, значит, в очередь была добавлена новая задача и сервис остановлен не будет.

- Перезапуск сервиса
  - Даже если ОС и внeштатно выгрузит сервис из памяти, есть возможность его запустить заново, кaк только появятся свободные ресурсы. 
  - Метод onStartCommand возвращает переменную, указывaющую ОС, как следует поступить с сервисом, если он был принудительно остановлен. Существует три варианта того, как ОС может поступить с сервисом, если он был принудительно завершен:
    - `START_NOT_STICKY` — не будет перезапущен системой, и все останется так, как есть. Подходит для случая, когда он выпoлняет не самые важные задачи, и приложение может позже при необходимости самостоятельно его перезапустить. 
    - `START_STICKY` — будет запущен заново, но в Intent, который ОС создаст для его запуска, не будет никаких параметров. Такой вариант работает с аудиоплеерами — он должен быть активным в фоне, но не обязательно при этом автоматически начинать проигрывать песни. 
    - `START_REDELIVER_INTENT` — сервис будет запущен с теми же параметрами, которые были при его последнем старте. Это удобно, когда в фоне загружается большой файл и его загрузка была прервана. 

## Broadcasts

- Как обновить UI-поток из фонового сервиса
  - Использовать Local Broadcast.

- BroadcastReceiver в Android 8
  1. Apps that target Android 8.0 or higher can no longer register broadcast receivers for implicit broadcasts in their manifest. An implicit broadcast is a broadcast that does not target that app specifically. For example, ACTION_PACKAGE_REPLACED is an implicit broadcast, since it is sent to all registered listeners, letting them know that some package on the device was replaced. However, ACTION_MY_PACKAGE_REPLACED is not an implicit broadcast, since it is sent only to the app whose package was replaced, no matter how many other apps have registered listeners for that broadcast.
  2. Apps can continue to register for explicit broadcasts in their manifests.
  3. Apps can use Context.registerReceiver() at runtime to register a receiver for any broadcast, whether implicit or explicit.
  4. Broadcasts that require a signature permission are exempted from this restriction, since these broadcasts are only sent to apps that are signed with the same certificate, not to all the apps on the device. 

## Custom Views

- onTouchEvent()
  - https://stfalcon.com/ru/blog/post/learning-android-gestures

## Другое

- Разница между Serializable и Parcelable
  - В первом используется рефлексия, довольно медленный процесс, однако разработчику нужно писать меньше кода. В Parcelable мы описываем только те вещи, которые нужно сериализовать, из-за чего кода становится больше. 

- Виды Intent-ов
  - **Implicit** Вызываете системный интент: отправить СМС, позвонить, открыть карты и так далее
  - **Explicit** Вызов других активити внутри приложения
  - **Sticky** Интент, который остаётся после завершения бродкаста. Например, при подписке на `ACTION_BATTERY_CHANGED`, мы получим последний посланный интент. Поэтому, если нам нужно только текущее состояние батареи, слушать дальнейшие бродкасты не обязательно. 
  - **Pending** Интент, который может быть исполнен в будущем на правах вашего приложения 

- Может ли приложение работать в разных процессах? Зачем это нужно? Как можно организовать межпроцессорное взаимодействие?
  - Может. Для частей приложения (активити, сервисы, бродкасты и контент провайдеры) надо указать флаг process. 
  - +: Получаем больше памяти. Не всё приложение при недостатке памяти будет убито. 
  - -: Поскольку каждый процесс живёт в отдельном инстансе Дальвика, делиться информацией сложно. Для этого используются AIDL, интенты, handler-ы, messenger-ы

- Из-за чего возникает ошибка Application Not Responding (ANR)
  - Когда UI не отвечает несколько секунд. Случается обычно из-за блокированного главного треда.

- Как обнаружить ANR?
```kotlin
class App : Application() {

    var tick = 0 % Integer.MAX_VALUE

    override fun onCreate() {
        super.onCreate()

        val handler = Handler(mainLooper)

        thread {
            while (true) {
                val lastTick = tick + 1
                handler.post { tick += 1 }
                TimeUnit.SECONDS.sleep(5)

                if (lastTick != tick)
                    Log.d("APP", "ANR")
            }
        }

    }
}
```

# Архитектура

**TBD**

# Библиотеки

## Moxy

- Moxy создаёт $$PresenterBinder, хранит хеш-мапы c биндерами в MoxyReflector

## RxJava

- Subjects
  - Publish Subject: Излучает(emit) все последующие элементы наблюдаемого источника в момент подписки.
  - Replay Subject: Излучает(emit все элементы источника наблюдаемого(Observable), независимо от того, когда подписчик(subscriber) подписывается.
  - Behavior Subject: Он излучает(emit) совсем недавно созданый элемент и все последующие элементы наблюдаемого источника, когда наблюдатель(observer) присоединяется к нему.
  - Async Subject: Он выдает только последнее значение наблюдаемого источника(и только последнее). Чтоб его подписчики получили содержание, на сабджекте нужно вызвать onComplete

- flatMap / switchMap / concatMap / concatMapEager
  - `flatMap` не следит за порядком получаемых значений. Для каждого из передаваемых ему значений он создаёт свой observable и может прийти в любом порядке.
  - `switchMap` когда новый объект появлятся из observable выше, он отписывается от остальных — получает только последнее из значений. 
  - `concatMap` похож на `flatMap`, но сохраняет порядок значений. Но есть и минус: `concatMap` ждёт заверешения каждого observable перед переходом к следующему
  - `concatMapEager` похож на `concatMap`, но выполняет код observable асинхронно. 
