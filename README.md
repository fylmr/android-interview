# Вопросы к собеседованию по Android 

- [Вопросы к собеседованию по Android](#%D0%B2%D0%BE%D0%BF%D1%80%D0%BE%D1%81%D1%8B-%D0%BA-%D1%81%D0%BE%D0%B1%D0%B5%D1%81%D0%B5%D0%B4%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8E-%D0%BF%D0%BE-android)
  - [Структуры данных](#%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D1%8B-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)
  - [Java](#java)
  - [Android](#android)
  - [Архитектура](#%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0)
  - [Библиотеки](#%D0%B1%D0%B8%D0%B1%D0%BB%D0%B8%D0%BE%D1%82%D0%B5%D0%BA%D0%B8)
    - [Moxy](#moxy)
    - [RxJava](#rxjava)

## Структуры данных

## Java

- Виды референсов: https://habr.com/ru/post/169883/
  1. **SoftReference** — если GC видит что объект доступен только через цепочку soft-ссылок, то он удалит его из памяти, если памяти будет не хватать. Удобно для кеширования. 
  2. **WeakReference** — если GC видит что объект доступен только через цепочку weak-ссылок, то он удалит его из памяти. Применение: WeakHashMap. Это реализация Map<K,V> которая хранит ключ, используя weak-ссылку. И когда GC удаляет ключ с памяти, то удаляется вся запись с Map. Думаю не сложно понять, как это происходит. 
  3. **PhantomReference** — если GC видит что объект доступен только через цепочку phantom-ссылок, то он его удалит из памяти. После нескольких запусков GC. 
   Особенностей у этого типа ссылок две. Первая это то, что метод **get() всегда возвращает null**. Именно из-за этого PhantomReference имеет смысл использовать только вместе с ReferenceQueue. Вторая особенность – в отличие от SoftReference и WeakReference, GC добавит phantom-ссылку в ReferenceQueue после того как выполнится метод finalize(). То есть фактически, в отличии от SoftReference и WeakReference, объект еще есть в памяти.

- Volatile/Atomic
  - *volatile* помечает объект как доступный для нескольких тредов. Операция инкремента не атомарна, из-за чего процесс будет сбоить
  - *Atomic*-классы использут compare and swap алгоритм: обновляет значение, только если предыдущее совпадает с ожидаемым old value.

## Android

- Жизненный цикл Activity
  - `OnCreate()` Создаётся view.
  - `OnStart()` Активити становится видно
  - `OnResume()` Активити становится доступным для ввода пользователя
  - `OnPause()` Активити видно, но недоступо для ввода пользователя (важно для многооконного режима)
  - `OnStop()` Активити больше не видно
  - `OnDestroy()` Активити уничтожается
  - `OnRestart()` Активити пересоздаётся, вызывается после уничтожения и перед созданием

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

- Жизненный цикл Fragment
  - ![Жизненный цикл фрагмента](https://i.stack.imgur.com/fRxIQ.png)

- Описать Content Provider
  - Позволяет передавать данные между приложениями и процессами. Используется в связке с `ContentResolver`.

- Описать Service/IntentService
  - Часть приложения без UI
  - **Foreground** Выполняется в основном потоке. Во время исполнения показывает уведомление. Пример — аудиоплеер.
  - **Background** Выполняется в фоновом потоке. Начиная с API 26, приложения в фоне не могут создавать фоновые сервисы, решением в этом случае может быть `WorkManager`.
  - **Bound** Клиент-серверный подход: посылаем запросы, получаем результаты. 
  - **IntentService** Создаёт и работает в собственном потоке. 

- Описать Handler
  - Handler - это механизм, который позволяет работать с очередью сообщений между потоками. Он привязан к конкретному потоку и работает с его очередью. Handler умеет помещать сообщения в очередь. При этом он ставит самого себя в качестве получателя этого сообщения. И когда приходит время, система достает сообщение из очереди и отправляет его адресату (т.е. в Handler) на обработку.

- Разница между Serializable и Parcelable
  - В первом используется рефлексия, довольно медленный процесс, однако разработчику нужно писать меньше кода. В Parcelable мы описываем только те вещи, которые нужно сериализовать, из-за чего кода становится больше. 

- Как обновить UI-поток из фонового сервиса
  - Использовать Local Broadcast.

- Виды Intent-ов
  - **Implicit** Вызываете системный интент: отправить СМС, позвонить, открыть карты и так далее
  - **Explicit** Вызов других активити внутри приложения
  - **Sticky** Интент, который остаётся после завершения бродкаста. Например, при подписке на `ACTION_BATTERY_CHANGED`, мы получим последний посланный интент. Поэтому, если нам нужно только текущее состояние батареи, слушать дальнейшие бродкасты не обязательно. 
  - **Pending** Интент, который может быть исполнен в будущем на правах вашего приложения 

- Из-за чего возникает ошибка Application Not Responding (ANR)
  - Когда UI не отвечает 5 секунд. Случается обычно из-за блокированного главного треда.

- BroadcastReceiver в Android 8
  1. Apps that target Android 8.0 or higher can no longer register broadcast receivers for implicit broadcasts in their manifest. An implicit broadcast is a broadcast that does not target that app specifically. For example, ACTION_PACKAGE_REPLACED is an implicit broadcast, since it is sent to all registered listeners, letting them know that some package on the device was replaced. However, ACTION_MY_PACKAGE_REPLACED is not an implicit broadcast, since it is sent only to the app whose package was replaced, no matter how many other apps have registered listeners for that broadcast.
  2. Apps can continue to register for explicit broadcasts in their manifests.
  3. Apps can use Context.registerReceiver() at runtime to register a receiver for any broadcast, whether implicit or explicit.
  4. Broadcasts that require a signature permission are exempted from this restriction, since these broadcasts are only sent to apps that are signed with the same certificate, not to all the apps on the device. 

- Может ли приложение работать в разных процессах? Зачем это нужно? Как можно организовать межпроцессорное взаимодействие?
  - Может. Для частей приложения (активити, сервисы, бродкасты и контент провайдеры) надо указать флаг process. 
  - +: Получаем больше памяти. Не всё приложение при недостатке памяти будет убито. 
  - -: Поскольку каждый процесс живёт в отдельном инстансе Дальвика, делиться информацией сложно. Для этого используются AIDL, интенты, handler-ы, messenger-ы

- onTouchEvent()
  - https://stfalcon.com/ru/blog/post/learning-android-gestures


## Архитектура

## Библиотеки

### Moxy

- Moxy создаёт $$PresenterBinder, хранит хеш-мапы c биндерами в MoxyReflector

### RxJava

- Subjects
  - Publish Subject: Излучает(emit) все последующие элементы наблюдаемого источника в момент подписки.
  - Replay Subject: Излучает(emit все элементы источника наблюдаемого(Observable), независимо от того, когда подписчик(subscriber) подписывается.
  - Behavior Subject: Он излучает(emit) совсем недавно созданый элемент и все последующие элементы наблюдаемого источника, когда наблюдатель(observer) присоединяется к нему.
  - Async Subject: Он выдает только последнее значение наблюдаемого источника(и только последнее). Чтоб его подписчики получили содержание, на сабджекте нужно вызвать onComplete