# Ответы на билеты по Android-разработке

## 1. Что такое Activity и жизненный цикл Activity
**Activity** — это компонент приложения, который предоставляет экран для взаимодействия с пользователем. Каждая Activity имеет свое окно для отрисовки UI.

**Жизненный цикл Activity** — это последовательность состояний, через которые проходит Activity от создания до уничтожения. Управляется через колбэк-методы:

*   `onCreate()`: Вызывается при первом создании Activity. Здесь происходит инициализация: создание UI, привязка данных.
*   `onStart()`: Вызывается, когда Activity становится видимой для пользователя.
*   `onResume()`: Вызывается, когда Activity выходит на передний план и готова к взаимодействию.
*   `onPause()`: Вызывается, когда Activity теряет фокус (например, ее перекрывает другая Activity). Здесь освобождаются ресурсы и сохраняются данные.
*   `onStop()`: Вызывается, когда Activity становится невидимой для пользователя.
*   `onDestroy()`: Вызывается перед уничтожением Activity для финального освобождения ресурсов.
*   `onRestart()`: Вызывается после `onStop()` перед тем, как Activity снова станет видимой (`onStart()`).

---

## 2. Что такое Fragment и его жизненный цикл
**Fragment** — это модульная часть Activity, которая имеет свой UI, логику и жизненный цикл. Фрагменты позволяют создавать гибкие и переиспользуемые UI.

**Отличия от Activity**:
*   Фрагменты не могут существовать без Activity.
*   Activity представляет собой целый экран, а фрагменты — его части.
*   Фрагменты проще переиспользовать в разных Activity.

**Жизненный цикл Fragment**:
*   `onAttach()`: Фрагмент прикрепляется к Activity.
*   `onCreate()`: Инициализация фрагмента.
*   `onCreateView()`: Создание и возврат иерархии View.
*   `onViewCreated()`: Вызывается после `onCreateView()`, когда View создано.
*   `onStart()`/`onResume()`: Аналогично Activity.
*   `onPause()`/`onStop()`: Аналогично Activity.
*   `onDestroyView()`: Иерархия View уничтожается.
*   `onDestroy()`: Финальное уничтожение фрагмента.
*   `onDetach()`: Фрагмент открепляется от Activity.

**Пример подключения**:
*   **XML**: `<fragment android:name="com.example.MyFragment" ... />`
*   **Код**: `supportFragmentManager.beginTransaction().add(R.id.container, MyFragment()).commit()`

---

## 3. ViewBinding: зачем нужен и как подключить
**ViewBinding** — это механизм, который генерирует класс привязки для каждого XML-файла макета, обеспечивая безопасный доступ к View.

**Зачем нужен**:
*   **Null-безопасность**: Прямые ссылки на View исключают `NullPointerException`.
*   **Типобезопасность**: Поля в классе привязки имеют правильные типы, что предотвращает ошибки приведения типов (`ClassCastException`).
*   **Производительность**: ViewBinding работает быстрее, чем `findViewById`, так как поиск View происходит один раз во время компиляции, а не во время выполнения.

**Как подключить** (в `build.gradle` модуля):
```groovy
android {
    //...
    buildFeatures {
        viewBinding true
    }
}
```

**Пример использования во фрагменте**:
```kotlin
private var _binding: MyFragmentLayoutBinding? = null
private val binding get() = _binding!!

override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
    _binding = MyFragmentLayoutBinding.inflate(inflater, container, false)
    return binding.root
}

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    binding.textView.text = "Hello ViewBinding!"
}

override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
}
```

---

## 4. MVVM: основные компоненты и связи между ними
**MVVM (Model-View-ViewModel)** — это архитектурный паттерн, разделяющий логику приложения на три компонента:

*   **View (Представление)**: UI-слой (Activity, Fragment), который отображает данные и передает действия пользователя в ViewModel. Подписывается на изменения данных из ViewModel.
*   **ViewModel (Модель представления)**: Хранит и управляет данными для UI. Переживает изменения конфигурации (например, поворот экрана). Не имеет прямых ссылок на View.
*   **Model (Модель)**: Слой данных (Repository, UseCases), отвечающий за бизнес-логику и предоставление данных (из сети, базы данных).

**Связи**: `View <—> ViewModel <—> Model`
1.  **View** сообщает **ViewModel** о действиях пользователя.
2.  **ViewModel** запрашивает данные у **Model** (через UseCase).
3.  **Model** возвращает данные в **ViewModel**.
4.  **ViewModel** обновляет свои данные (например, `LiveData` или `StateFlow`).
5.  **View**, подписанная на эти данные, автоматически обновляет UI.

**Как ViewModel получает данные из UseCase**: ViewModel вызывает метод UseCase. UseCase обращается к Repository, выполняет бизнес-логику и возвращает результат в ViewModel.

---

## 5. Room: создание Entity
**Entity** — это класс, который представляет таблицу в базе данных Room.

*   **Как создать таблицу**: Класс помечается аннотацией `@Entity`. Имя таблицы по умолчанию совпадает с именем класса.
*   **`@PrimaryKey`**: Обязательная аннотация для первичного ключа. Свойство `autoGenerate = true` позволяет автоматически генерировать уникальные ID.
*   **`@ColumnInfo`**: Позволяет задать имя столбца в таблице, отличное от имени поля в классе.

**Пример**:
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int,
    
    @ColumnInfo(name = "first_name")
    val firstName: String,
    
    val age: Int
)
```

---

## 6. Room: DAO-интерфейсы
**DAO (Data Access Object)** — это интерфейс, который определяет методы для доступа к базе данных (вставки, обновления, удаления, запросы).

**Аннотации**:
*   `@Query`: Для выполнения SQL-запросов (SELECT, UPDATE, DELETE).
*   `@Insert`: Для вставки данных. Можно указать `onConflict` стратегию (например, `OnConflictStrategy.REPLACE`).
*   `@Update`: Для обновления записей.
*   `@Delete`: Для удаления записей.

**Возврат списка через `LiveData` или `Flow`**: DAO-методы могут возвращать данные, обернутые в `LiveData` или `Flow`. Это позволяет UI автоматически обновляться при изменении данных в базе.
* **Разница** -- LiveData предназначен для управления данными, связанными с жизненным циклом UI-компонентов, в то время как Flow предлагает более широкие возможности для обработки асинхронных данных и более гибкую работу с потоками, особенно в контексте корутин
**Пример**:
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)

    @Delete
    suspend fun deleteUser(user: User)
}
```

---

## 7. Room: создание базы данных
Класс базы данных должен быть абстрактным и наследоваться от `RoomDatabase`.

*   **`@Database`**: Аннотация для класса базы данных. В ней указываются `entities` (список классов-сущностей) и `version` (версия базы данных).
*   **Инициализация**: База данных создается с помощью `Room.databaseBuilder()` или `Room.inMemoryDatabaseBuilder()`.
* **Room.databaseBuilder** - это статический метод в библиотеке Room Persistence Library для Android, который используется для создания экземпляра RoomDatabase.
**Пример**:
```kotlin
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}

// Инициализация в синглтоне или DI-контейнере
val db = Room.databaseBuilder(
    applicationContext,
    AppDatabase::class.java, "database-name"
).build()
```

---

## 8. Room: миграции
**Миграция** — это процесс обновления схемы базы данных при изменении ее версии.

*   **Что делать при изменении схемы**: При изменении структуры таблиц (добавление/удаление столбцов и т.д.) необходимо увеличить версию `@Database` и предоставить `Migration`.
*   **`fallbackToDestructiveMigration()`**: Этот метод удаляет старую базу данных и создает новую. Все данные будут потеряны. Используется в основном на ранних этапах разработки.
*   **`addMigrations()`**: Позволяет предоставить объект `Migration` для миграции между версиями. Внутри `Migration` пишется SQL-код для изменения схемы.

**Пример миграции**:
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN last_name TEXT")
    }
}

val db = Room.databaseBuilder(...)
    .addMigrations(MIGRATION_1_2)
    .build()
```

---

## 9. Room: связи между таблицами (One-To-Many и Many-To-Many)
Room позволяет моделировать связи между таблицами с помощью специальных аннотаций.

*   **`@Embedded`**: Позволяет вложить один объект в другой. Поля вложенного объекта рассматриваются как столбцы той же таблицы.
*   **`@Relation`**: Используется для моделирования связей "один-ко-многим" и "многие-ко-многим".
    *   `parentColumn`: Имя столбца в родительской сущности (первичный ключ).
    *   `entityColumn`: Имя столбца в дочерней сущности (внешний ключ).

**Пример (One-To-Many)**: Один пользователь (`User`) может иметь много плейлистов (`Playlist`).
```kotlin
data class UserWithPlaylists(
    @Embedded val user: User,
    @Relation(
         parentColumn = "id",
         entityColumn = "user_id"
    )
    val playlists: List<Playlist>
)

// В DAO
@Transaction
@Query("SELECT * FROM users WHERE id = :userId")
fun getUserWithPlaylists(userId: Int): Flow<UserWithPlaylists>
```

---

## 10. KAPT и KSP: для чего используются в Room и Dagger
**KAPT (Kotlin Annotation Processing Tool)** и **KSP (Kotlin Symbol Processing)** — это инструменты, которые обрабатывают аннотации и генерируют код на этапе компиляции.

*   **Как работают**: Они сканируют исходный код на наличие специальных аннотаций (например, `@Dao`, `@Inject`). На основе этих аннотаций они генерируют дополнительный Kotlin/Java код, который используется в приложении (например, реализации DAO или фабрики для Dagger).
*   **Использование в Room и Dagger**:
    *   **Room** использует их для генерации реализаций `DAO`-интерфейсов и класса базы данных.
    *   **Dagger** использует их для генерации кода, который связывает зависимости (графы зависимостей).
*   **Отличия KAPT и KSP**:
    *   **KSP** — это более современная и производительная альтернатива KAPT. Он напрямую анализирует Kotlin-код, что делает его значительно быстрее.
    *   **KAPT** сначала создает Java-заглушки из Kotlin-кода, а затем запускает Java-процессоры аннотаций, что медленнее. KSP рекомендуется для новых проектов.

---

## 11. RecyclerView: адаптер, ViewHolder, отображение списка
**RecyclerView** — это гибкий и производительный компонент для отображения больших списков данных.

*   **Adapter**: Связывает данные со списком. Он создает `ViewHolder`'ы и привязывает к ним данные.
*   **ViewHolder**: Хранит ссылки на View для одного элемента списка. Это позволяет избежать постоянных вызовов `findViewById`.
*   **LayoutManager**: Определяет, как элементы будут расположены (линейно, сеткой и т.д.).

**Базовый пример адаптера**:
```kotlin
class MyAdapter(private val items: List<String>) : RecyclerView.Adapter<MyAdapter.MyViewHolder>() {

    class MyViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val textView: TextView = view.findViewById(R.id.item_text)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.list_item, parent, false)
        return MyViewHolder(view)
    }

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        holder.textView.text = items[position]
    }

    override fun getItemCount() = items.size
}
```

*   **DiffUtil**: Инструмент для вычисления разницы между двумя списками и эффективного обновления `RecyclerView`. Он позволяет анимировать изменения (добавление, удаление, перемещение) без необходимости вызывать `notifyDataSetChanged()`. Используется вместе с `ListAdapter`.

---

## 12. Glide vs Picasso: различия и примеры использования
**Glide** и **Picasso** — популярные библиотеки для загрузки и кэширования изображений.

**Различия**:
*   **Кэширование**: Glide кэширует изображения в меньшем размере (по размеру `ImageView`), что экономит память и ускоряет загрузку. Picasso по умолчанию кэширует полноразмерное изображение.
*   **GIF**: Glide поддерживает загрузку GIF-анимаций "из коробки", Picasso — нет.
*   **Жизненный цикл**: Glide автоматически управляет загрузками в соответствии с жизненным циклом Activity/Fragment.
*   **Производительность**: Glide обычно считается более производительным из-за умного кэширования и использования памяти.

**Пример использования (Glide)**:
```kotlin
Glide.with(context)
    .load("https://example.com/image.jpg")
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .into(imageView)
```

---

## 13. REST API: Retrofit + Coroutines
* **Retrofit** — это типобезопасный HTTP-клиент для Android и Java. Он упрощает работу с REST API.
* **REST API** -- архитектурный стиль для создания сетевых приложений, позволяющий клиентам и серверам взаимодействовать друг с другом
* **Coroutines** -- легковесные потоки выполнения, которые позволяют создавать асинхронный код, не блокирующий основной поток
*   **Пример интерфейса с `@GET`**:
    ```kotlin
    interface ApiService {
        @GET("users/{id}")
        suspend fun getUser(@Path("id") userId: Int): User
    }
    ```
    `suspend` указывает, что это функция для корутин.

*   **Вызов в ViewModel через UseCase**:
    ```kotlin
    // UseCase
    class GetUserUseCase(private val repository: UserRepository) {
        suspend operator fun invoke(userId: Int) = repository.getUser(userId)
    }

    // ViewModel
    class UserViewModel(private val getUserUseCase: GetUserUseCase) : ViewModel() {
        fun fetchUser(userId: Int) {
            viewModelScope.launch {
                val user = getUserUseCase(userId)
                // Update UI with user
            }
        }
    }
    ```

---

## 14. ViewModel + LiveData
**ViewModel** хранит и управляет данными, связанными с UI, а **LiveData** — это наблюдаемый хранитель данных.

*   **Хранение данных при поворотах**: `ViewModel` переживает изменения конфигурации, поэтому данные в ней не теряются при повороте экрана.
*   **`LiveData`**: Оповещает `View` об изменениях данных, только если `View` находится в активном состоянии.
* **MutableLiveData** - это изменяемая версия класса LiveData, используемая в Android для отслеживания изменений данных, которые могут быть изменены извне
**Пример с `MutableLiveData`**:
```kotlin
class MyViewModel : ViewModel() {
    private val _userName = MutableLiveData<String>()
    val userName: LiveData<String> = _userName

    fun loadUser() {
        // ... загрузка данных ...
        _userName.value = "John Doe"
    }
}

// В Activity или Fragment
viewModel.userName.observe(viewLifecycleOwner) { name ->
    textView.text = name
}
```

---

## 15. Coroutines: launch, async, withContext
**Coroutines (корутины)** — это инструмент для управления асинхронными операциями в Kotlin.

*   **`launch`**: Запускает корутину, которая не возвращает результат ("запустил и забыл"). Возвращает объект `Job`, который можно использовать для отмены.
*   **`async`**: Запускает корутину и возвращает результат в виде объекта `Deferred`. Результат можно получить, вызвав `await()`.
*   **`withContext`**: Переключает контекст (поток) для выполнения блока кода. Используется для переноса ресурсоемких операций в фоновые потоки.

**Диспетчеры**:
*   **`Dispatchers.Main`**: Основной поток Android. Используется для взаимодействия с UI.
*   **`Dispatchers.IO`**: Оптимизирован для операций ввода-вывода (сеть, диск, база данных).
*   **`Dispatchers.Default`**: Оптимизирован для ресурсоемких вычислений (сортировка, обработка данных).

**Пример**:
```kotlin
viewModelScope.launch { // Запуск в основном потоке
    val data = withContext(Dispatchers.IO) {
        // ... сетевой запрос или работа с БД ...
        "Some data"
    }
    // Обновление UI с 'data' в основном потоке
}
```

---

## 16. Dependency Injection с Dagger 2
**Dagger 2** — это фреймворк для внедрения зависимостей (DI), основанный на генерации кода.

*   **`@Module`**: Класс, который предоставляет зависимости.
*   **`@Provides`**: Метод внутри модуля, который создает и возвращает зависимость.
*   **`@Component`**: Интерфейс, который соединяет модули и точки внедрения.
*   **`@Inject`**: Аннотация для запроса зависимости (в конструкторе, поле или методе).

**Подключение ViewModel через Dagger**:
Обычно используется `ViewModelFactory` и `Map<Class<out ViewModel>, Provider<ViewModel>>` для предоставления `ViewModel`.

**Пример**:
```kotlin
@Module
class AppModule {
    @Provides
    @Singleton
    fun provideApiService(): ApiService {
        // ... создание Retrofit ...
    }
}

@Component(modules = [AppModule::class])
interface AppComponent {
    fun inject(activity: MainActivity)
}

// Внедрение
class MainActivity : AppCompatActivity() {
    @Inject lateinit var apiService: ApiService
    
    override fun onCreate(savedInstanceState: Bundle?) {
        (application as MyApp).appComponent.inject(this)
        super.onCreate(savedInstanceState)
    }
}
```

---

## 17. Dependency Injection с Koin
**Koin** — это легковесный фреймворк для DI в Kotlin, использующий DSL (предметно-ориентированный язык).

*   **Определение модулей**: Зависимости объявляются в модулях с помощью функций `single` (синглтон), `factory` (новая инстанция каждый раз).
*   **Инжект ViewModel**: Koin предоставляет специальные делегаты `by viewModel()` для простого внедрения ViewModel.

**Пример**:
```kotlin
// Модуль
val appModule = module {
    single { ApiService() }
    viewModel { MyViewModel(get()) } // 'get()' резолвит ApiService
}

// Запуск Koin в Application
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApp)
            modules(appModule)
        }
    }
}

// Использование в Activity/Fragment
class MyActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModel()
}
```

---

## 18. UnitTest ViewModel
**Unit-тесты** проверяют небольшие части кода (методы, классы) изолированно.

*   **Инструменты**:
    *   **JUnit**: Фреймворк для написания тестов.
    *   **Mockito**: Библиотека для создания моков (mock) — поддельных объектов для имитации зависимостей.
    *   **Truth**: Библиотека для написания утверждений (assertions), более читаемая, чем стандартные.
*   **Тест с моканным UseCase**:
    1.  Создать мок для UseCase с помощью Mockito.
    2.  Определить, что должен возвращать мок при вызове его методов (`when(...).thenReturn(...)`).
    3.  Создать экземпляр ViewModel, передав в него мок.
    4.  Вызвать метод ViewModel, который нужно протестировать.
    5.  Проверить результат (например, значение `LiveData`) с помощью Truth.

**Пример**:
```kotlin
@Test
fun `test fetch user`() = runBlockingTest {
    // Given
    val useCase = mock(GetUserUseCase::class.java)
    val user = User("John")
    `when`(useCase.invoke(1)).thenReturn(Result.success(user))
    
    val viewModel = UserViewModel(useCase)
    
    // When
    viewModel.fetchUser(1)
    
    // Then
    val liveDataValue = viewModel.user.getOrAwaitValue()
    assertThat(liveDataValue.name).isEqualTo("John")
}
```

---

## 19. AndroidManifest.xml: важные элементы
**AndroidManifest.xml** — это файл, который предоставляет системе Android всю необходимую информацию о приложении.
* **service** -- компонент приложения, который предназначен для выполнения длительных операций в фоновом режиме, без необходимости наличия пользовательского интерфейса
* **receiver** -- компонент, позволяющий приложению реагировать на системные или пользовательские события (широковещательные сообщения)
* **provider** -- компонент приложения, который позволяет предоставлять данные другим приложениям, реализуя унифицированный интерфейс для доступа к данным
*   **`<uses-permission>`**: Запрос разрешений (например, доступ в интернет, к камере, геолокации).
    ` <uses-permission android:name="android.permission.INTERNET" />`
*   **`<application>`**: Основной тег, описывающий приложение.
*   **`<activity>`, `<service>`, `<receiver>`, `<provider>`**: Регистрация компонентов приложения. Для Activity здесь указывается, какая из них является точкой входа (`LAUNCHER`).
*   **`<intent-filter>`**: Описывает, какие Intent'ы может принимать компонент.

---

## 20. Сервис в Android: создание и запуск
**Service** — это компонент приложения, который может выполнять длительные операции в фоновом режиме без предоставления UI.

*   **Создание**: Наследуется от класса `Service` и переопределяются методы, такие как `onStartCommand()` и `onCreate()`.
*   **Запуск**:
    *   `startService()`: Запускает сервис для выполнения длительной задачи. Сервис будет работать, пока не будет остановлен явно (`stopSelf()` или `stopService()`).
    *   `bindService()`: Привязывает компонент (например, Activity) к сервису для взаимодействия с ним.
*   **Разница между обычным и Foreground-сервисом**:
    *   **Обычный (фоновый) сервис**: Может быть убит системой при нехватке памяти. Имеет ограничения на работу в фоне на новых версиях Android.
    *   **Foreground-сервис**: Имеет более высокий приоритет и редко убивается системой. Обязательно должен отображать видимое для пользователя уведомление. Используется для задач, которые пользователь должен осознавать (проигрывание музыки, отслеживание геолокации).

---

## 21. BroadcastReceiver
**BroadcastReceiver** — это компонент, который позволяет приложению получать системные или пользовательские сообщения (Intent'ы), называемые "широковещательными сообщениями".

*   **Регистрация**:
    *   **Статическая (в манифесте)**: Ресивер объявляется в `AndroidManifest.xml`. Приложение может получить сообщение, даже если оно не запущено.
    *   **Динамическая (в коде)**: Ресивер регистрируется с помощью `registerReceiver()` и отменяется с помощью `unregisterReceiver()`. Работает только пока приложение активно.
*   **Пример: мониторинг уровня заряда**:
    1.  Создать класс, наследующийся от `BroadcastReceiver`.
    2.  В методе `onReceive()` обработать `Intent` с `ACTION_BATTERY_CHANGED`.
    3.  Зарегистрировать ресивер в манифесте или динамически в коде для получения этих сообщений.

---

## 22. Jetpack Compose: создание базового UI
**Jetpack Compose** — это современный декларативный UI-фреймворк для создания нативных интерфейсов в Android.

*   **Декларативный подход**: Вы описываете, *что* должно быть на экране, а не *как* это нарисовать.
*   **Composable-функции**: UI строится из функций, помеченных аннотацией `@Composable`.
*   **`remember`**: Функция для хранения состояния внутри Composable-функции. Состояние сохраняется при рекомпозиции (перерисовке).

**Пример**:
```kotlin
@Composable
fun GreetingScreen() {
    var name by remember { mutableStateOf("") }

    Column(modifier = Modifier.padding(16.dp)) {
        Text(text = "Hello, ${if (name.isNotBlank()) name else "World"}!")
        
        OutlinedTextField(
            value = name,
            onValueChange = { name = it },
            label = { Text("Name") }
        )
        
        Button(onClick = { /* ... */ }) {
            Text("Click Me")
        }
    }
}
```

---

## 23. Отличия XML UI и Jetpack Compose
| Характеристика | XML UI | Jetpack Compose |
| :--- | :--- | :--- |
| **Подход** | Императивный (как делать) | Декларативный (что делать) |
| **Язык** | XML для макетов, Kotlin/Java для логики | Только Kotlin |
| **Состояние** | Управляется вручную (`setText`, `setVisibility`) | Реактивное, управляется через `State` и `remember` |
| **Код** | Разделение на XML и Kotlin/Java | Меньше кода, логика и UI в одном месте |
| **Переиспользование** | Через `<include>`, `<merge>`, Custom Views | Проще, через обычные функции |
| **Предпросмотр** | В XML-редакторе | Встроенный и интерактивный (`@Preview`) |

**Преимущества Compose**:
*   Меньше кода, быстрее разработка.
*   Интуитивно понятное управление состоянием.
*   Мощные инструменты для анимации и кастомных UI.
*   Полная совместимость с существующим XML UI.

**Недостатки Compose**:
*   Более новый, требует времени на изучение.
*   Может быть менее производительным в некоторых специфических сценариях (хотя постоянно улучшается).
  
* **Преиущества XML**: гибкость, универсальность, читаемость, возможность совместной работы, иерархическая структура.
* **Недостатки XML**: большие размеры файлов, сложность обработки, проблемы с производительностью.
---

## 24. Пагинация с Room и Paging 3
**Paging 3** — это библиотека из Jetpack для постепенной загрузки и отображения больших наборов данных.

*   **Как использовать `PagingSource` с Room**:
    1.  В DAO-интерфейсе определить метод, который возвращает `PagingSource<Int, MyEntity>`. Room автоматически сгенерирует реализацию.
    2.  `Int` — это тип ключа (например, номер страницы), `MyEntity` — тип загружаемых данных.

**Пример `PagingSource` в DAO**:
```kotlin
@Dao
interface MyDao {
    @Query("SELECT * FROM items ORDER BY name ASC")
    fun getItems(): PagingSource<Int, Item>
}
```

*   **Подключение к RecyclerView через `PagingDataAdapter`**:
    1.  В ViewModel создать `Pager` и `Flow<PagingData<Item>>`.
    2.  `PagingDataAdapter` — это специальный адаптер, который работает с `PagingData`.
    3.  В Activity/Fragment собрать `Flow` и передать данные в адаптер с помощью `submitData()`.

---

## 25. Сортировка и фильтрация данных в Room
Сортировка и фильтрация выполняются с помощью стандартных SQL-операторов в `@Query`.

*   **`ORDER BY`**: Для сортировки (ASC — по возрастанию, DESC — по убыванию).
*   **`WHERE`**: Для фильтрации по условию.
*   **`LIKE`**: Для поиска по шаблону (например, для поиска по части строки).

**Передача параметров в `@Query`**:
Параметры в SQL-запрос передаются через аргументы метода, помеченные двоеточием.

**Пример**:
```kotlin
@Dao
interface ProductDao {
    // Фильтрация по категории и сортировка по цене
    @Query("SELECT * FROM products WHERE category = :category ORDER BY price DESC")
    fun getProductsByCategorySortedByPrice(category: String): Flow<List<Product>>

    // Поиск по имени
    @Query("SELECT * FROM products WHERE name LIKE '%' || :query || '%'")
    fun searchProductsByName(query: String): Flow<List<Product>>
}
```

---

## 26. Использование Room + Flow
**Flow** — это асинхронный поток данных из библиотеки корутин Kotlin.

*   **Почему стоит использовать Flow**: Когда DAO-метод возвращает `Flow`, он становится "реактивным". Любое изменение данных в таблице (вставка, обновление, удаление) автоматически вызовет новую эмиссию данных в `Flow`. Это позволяет UI обновляться без дополнительного кода.
*   **Пример автоматического обновления UI**:
    ```kotlin
    // DAO
    @Query("SELECT * FROM users")
    fun getUsers(): Flow<List<User>>

    // ViewModel
    val users: StateFlow<List<User>> = userDao.getUsers()
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())

    // UI (Jetpack Compose)
    val userList by viewModel.users.collectAsState()
    LazyColumn {
        items(userList) { user ->
            Text(user.name)
        }
    }
    ```
    Когда в таблицу `users` будет добавлена новая запись, `userList` автоматически обновится, и Compose перерисует список.

---

## 27. Сохранение данных во ViewModel через SavedStateHandle
**`SavedStateHandle`** — это объект, который позволяет ViewModel сохранять и восстанавливать состояние после того, как процесс приложения был убит системой (например, из-за нехватки памяти).

*   **Как использовать**:
    1.  Добавьте `SavedStateHandle` в конструктор вашей `ViewModel`.
    2.  Используйте методы `get`, `set`, `getLiveData` для работы с данными. `SavedStateHandle` работает как `Map` или `Bundle`.

**Пример**:
```kotlin
class MyViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {

    // Получаем LiveData, которая автоматически сохраняется и восстанавливается
    val query: MutableLiveData<String> = savedStateHandle.getLiveData("query")

    fun setQuery(newQuery: String) {
        // Установка значения также сохраняет его в SavedStateHandle
        query.value = newQuery
    }
}
```
Теперь, если система убьет приложение, а пользователь вернется к нему, значение `query` будет восстановлено.

---

## 28. Gradle: зависимости и плагины
**Gradle** — это система автоматической сборки, используемая в Android-проектах.

*   **Зависимости**:
    *   `implementation`: Зависимость доступна только внутри текущего модуля. Это предпочтительный способ, так как он ускоряет сборку. Уменьшает время пересборки, так как        изменения в такой зависимости не требуют перекомпиляции зависимых модулей.
    *   `api`: Зависимость становится частью API модуля и транзитивно доступна модулям, которые зависят от текущего. Используется, когда нужно предоставить зависимость другим модулям (например, в библиотеках). Увеличивает время сборки, так как изменения в api-зависимости требуют перекомпиляции всех зависимых модулей.
    *   `kapt`: Для подключения процессоров аннотаций (старый способ). Генерирует код во время компиляции.
    *   `ksp`: Для подключения процессоров аннотаций на основе KSP (новый, быстрый способ).
*   **Плагины**: Расширяют возможности Gradle (например, `com.android.application` для сборки приложения, `kotlin-android` для поддержки Kotlin).

**Пример добавления библиотеки**:
```groovy
// build.gradle (Module)
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'com.google.devtools.ksp' version '1.8.0-1.0.9' // Пример плагина KSP
}

dependencies {
    implementation 'androidx.core:core-ktx:1.9.0'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    
    def room_version = "2.5.0"
    implementation "androidx.room:room-runtime:$room_version"
    ksp "androidx.room:room-compiler:$room_version" // Подключение Room через KSP
}
```

---

## 29. Отличия между launchActivity и startActivityForResult
*  **startActivityForResult()** (устаревший, но используется в старом коде)
* Запускает Activity и ожидает результат через колбэк onActivityResult().
* Работает через Intent и requestCode для идентификации запроса.

* **launch()** с ActivityResultLauncher (современный способ, через Activity Result API)
* Использует ActivityResultContracts для типизированного взаимодействия.
* Не требует requestCode, более удобен и безопасен.
**Современный подход — Activity Result API**:
`startActivityForResult` и `onActivityResult` были заменены на **Activity Result API**. Он более безопасен и гибок.

1.  Регистрируется `ActivityResultLauncher` с помощью `registerForActivityResult`.
2.  В качестве контракта передается, например, `ActivityResultContracts.StartActivityForResult()`.
3.  В лямбде-колбэке обрабатывается результат.
4.  Для запуска используется метод `launch()` лаунчера.

**Пример**:
```kotlin
val startForResult = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result: ActivityResult ->
    if (result.resultCode == Activity.RESULT_OK) {
        val intent = result.data
        // ... обработка данных из intent ...
    }
}

// Запуск
val intent = Intent(this, SecondActivity::class.java)
startForResult.launch(intent)

// В SecondActivity для возврата результата
val resultIntent = Intent()
resultIntent.putExtra("key", "value")
setResult(Activity.RESULT_OK, resultIntent)
finish()
```

---

## 30. Как реализовать UseCase в Clean Architecture
**UseCase (или Interactor)** — это класс в `domain`-слое, который инкапсулирует одну конкретную бизнес-операцию.

*   **Пример `GetUsersUseCase`**:
    ```kotlin
    // В domain слое
    class GetUsersUseCase(private val userRepository: UserRepository) {
        suspend operator fun invoke(): Result<List<User>> {
            return try {
                Result.success(userRepository.getUsers())
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    ```
    *   Имя класса описывает его действие.
    *   Переопределение `operator fun invoke` позволяет вызывать класс как функцию: `getUsersUseCase()`.
    *   Он зависит от абстракции (`UserRepository`), а не от реализации.
*   **Где и как вызывать в MVVM**: `ViewModel` вызывает `UseCase` для выполнения бизнес-логики.
    ```kotlin
    // ViewModel
    class UsersViewModel(private val getUsersUseCase: GetUsersUseCase) : ViewModel() {
        fun loadUsers() {
            viewModelScope.launch {
                val result = getUsersUseCase()
                // ... обработать результат ...
            }
        }
    }
    ```

---

## 31. Многомодульная архитектура проекта
Разделение проекта на несколько независимых модулей.

*   **Разделение на слои**:
    *   `app`: Основной модуль приложения, связывает все вместе. Зависит от всех `feature` и `core` модулей.
    *   `core`: Общий код, используемый во всем приложении (утилиты, базовые классы, DI).
    *   `data`: Реализация репозиториев, работа с сетью (Retrofit) и базой данных (Room). Зависит от `domain`.
    *   `domain`: Бизнес-логика, UseCases, модели и интерфейсы репозиториев. Не зависит ни от чего, кроме Kotlin/Java.
    *   `feature_*`: Модули для каждой отдельной фичи приложения (например, `:feature_profile`, `:feature_feed`). Зависят от `domain` и `core`.
*   **Преимущества**:
    *   **Масштабируемость**: Проще добавлять новые фичи.
    *   **Скорость сборки**: Gradle может пересобирать только измененные модули.
    *   **Разделение ответственности**: Четкие границы между командами и фичами.
    *   **Переиспользование кода**: `core` и `domain` модули могут быть переиспользованы.

---

## 32. Обработка ошибок и Result-обёртка
**`Result`-обёртка** — это элегантный способ обработки успешных результатов и ошибок без использования `try-catch` блоков повсюду.

*   **Использование `sealed class`**:
    ```kotlin
    sealed class Result<out T> {
        data class Success<out T>(val data: T) : Result<T>()
        data class Error(val exception: Exception) : Result<Nothing>()
    }
    ```
    Это позволяет использовать `when` для исчерпывающей проверки всех исходов. (В Kotlin уже есть встроенный класс `kotlin.Result`, который можно использовать).

*   **Пример в UseCase и ViewModel**:
    ```kotlin
    // UseCase
    class MyUseCase(private val repository: MyRepository) {
        suspend operator fun invoke(): kotlin.Result<MyData> {
            return try {
                kotlin.Result.success(repository.fetchData())
            } catch (e: Exception) {
                kotlin.Result.failure(e)
            }
        }
    }

    // ViewModel
    fun loadData() {
        viewModelScope.launch {
            val result = myUseCase()
            result.onSuccess { data ->
                // Обновить UI успехом
            }.onFailure { error ->
                // Показать ошибку
            }
        }
    }
    ```

---

## 33. Compose: создание собственного компонента
Создание кастомного компонента в Compose — это просто создание новой `@Composable` функции.

*   **Пример кастомной кнопки**:
    ```kotlin
    @Composable
    fun CustomButton(
        text: String,
        onClick: () -> Unit,
        modifier: Modifier = Modifier,
        backgroundColor: Color = MaterialTheme.colors.primary,
        icon: ImageVector? = null
    ) {
        Button(
            onClick = onClick,
            modifier = modifier,
            colors = ButtonDefaults.buttonColors(backgroundColor = backgroundColor)
        ) {
            if (icon != null) {
                Icon(
                    imageVector = icon,
                    contentDescription = null,
                    modifier = Modifier.size(ButtonDefaults.IconSize)
                )
                Spacer(Modifier.size(ButtonDefaults.IconSpacing))
            }
            Text(text)
        }
    }
    ```
*   **Параметры**: Компонент принимает параметры для настройки его вида и поведения (текст, цвет, иконка, обработчик клика).
*   **`modifier`**: Важный параметр для настройки внешних атрибутов (отступы, размеры, рамки). Его всегда следует передавать.

---

## 34. Foreground Service и уведомления
**Foreground Service** — это сервис, выполняющий длительную задачу, о которой пользователь должен знать. Он обязан показывать постоянное уведомление.

*   **Как отобразить уведомление**:
    1.  Создать канал уведомлений (для Android 8+).
    2.  Создать `Notification` с помощью `NotificationCompat.Builder`.
    3.  Внутри сервиса, в `onCreate()` или `onStartCommand()`, вызвать `startForeground(notificationId, notification)`.
*   **Требования Android 8+**: Foreground-сервисы должны быть запущены с помощью `startForegroundService()`, а не `startService()`. После запуска у приложения есть несколько секунд, чтобы вызвать `startForeground()`, иначе система вызовет ANR (Application Not Responding).

---

## 35. Сравнение LiveData и StateFlow
**LiveData** и **StateFlow** — это классы для хранения и наблюдения за состоянием.

| Характеристика | LiveData | StateFlow |
| :--- | :--- | :--- |
| **Библиотека** | Android Architecture Components | Kotlin Coroutines |
| **Осведомленность о ЖЦ** | Встроенная | Требуется `lifecycleScope.launchWhenStarted` или `repeatOnLifecycle` |
| **Начальное значение** | Не обязательно | Обязательно |
| **Основной/Фоновый поток** | Работает только в Main Thread | Может эмитить данные из любого потока (но собирать в Main) |
| **Холодный/Горячий** | Горячий (активен, если есть наблюдатели) | Горячий (всегда активен в `scope`) |
| **Операторы** | Ограниченный набор (`Transformations`) | Богатый набор операторов из `Flow` (`map`, `filter` и т.д.) |

**Почему StateFlow предпочтительнее в новых проектах**:
*   Является частью Kotlin, а не только Android.
*   Более гибкий благодаря операторам `Flow`.
*   Лучше интегрируется с корутинами и структурированной многопоточностью.
*   `StateFlow` с `repeatOnLifecycle(Lifecycle.State.STARTED)` является более безопасным и эффективным способом сбора данных в UI по сравнению с `LiveData`.

---

## 36. Как работает Notification и его каналы
**Notification** — это сообщение, которое можно отобразить пользователю за пределами UI приложения.

*   **Каналы уведомлений (Android 8.0 / API 26+)**:
    *   Все уведомления должны быть приписаны к какому-либо каналу.
    *   Каналы позволяют пользователям группировать уведомления и управлять их настройками (звук, вибрация, важность) централизованно через системные настройки.
    *   Канал создается один раз при первом запуске приложения.

**Пример простого уведомления**:
```kotlin
// 1. Создать канал (один раз)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val name = "Channel Name"
    val descriptionText = "Channel Description"
    val importance = NotificationManager.IMPORTANCE_DEFAULT
    val channel = NotificationChannel("CHANNEL_ID", name, importance).apply {
        description = descriptionText
    }
    val notificationManager: NotificationManager =
        getSystemService(context, NotificationManager::class.java) as NotificationManager
    notificationManager.createNotificationChannel(channel)
}

// 2. Создать и показать уведомление
val builder = NotificationCompat.Builder(this, "CHANNEL_ID")
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("My notification")
    .setContentText("Much longer text that cannot fit one line...")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)

with(NotificationManagerCompat.from(this)) {
    notify(notificationId, builder.build()) // notificationId is a unique int
}
```

---

## 37. Какие бывают типы ViewGroup и их различия
**ViewGroup** — это контейнер для других View.

*   **`LinearLayout`**: Располагает дочерние элементы в одной строке (горизонтально или вертикально). Простой и быстрый, но негибкий для сложных макетов.
*   **`RelativeLayout`**: Располагает дочерние элементы относительно друг друга или относительно родителя. Более гибкий, но может приводить к сложной и вложенной иерархии.
*   **`FrameLayout`**: Располагает элементы друг на друге (в стопке, как слои). Идеален для отображения одного элемента, перекрытого другим (например, иконка поверх изображения).
*   **`ConstraintLayout`**: Самый мощный и гибкий. Позволяет создавать сложные макеты с плоской иерархией, что улучшает производительность. Элементы располагаются с помощью "ограничений" (constraints) относительно друг друга и родителя. Рекомендуется для большинства макетов.

---

## 38. Что такое suspend-функции и зачем они нужны
**`suspend`-функция** — это функция, которая может быть приостановлена и возобновлена позже.

*   **Описание асинхронной логики**: `suspend`-функции — это центральный элемент корутин. Они позволяют писать асинхронный код так, как будто он синхронный, без колбэков.
*   **Правила**:
    *   `suspend`-функцию можно вызвать только из другой `suspend`-функции или из корутин-билдера (например, `launch`, `async`).
    *   Они не блокируют поток. Когда `suspend`-функция приостанавливается (например, ждет ответ от сети), поток, в котором она была вызвана, освобождается и может выполнять другую работу.
*   **Использование**: Идеальны для длительных операций, таких как сетевые запросы, работа с базой данных или файлами.

**Пример**:
```kotlin
suspend fun fetchUser(id: String): User {
    // Эта функция не блокирует поток, а приостанавливает корутину
    return api.getUser(id) 
}
```

---

## 39. Отличие launch и async в корутинах
**`launch`** и **`async`** — это два основных корутин-билдера.

*   **`launch`**:
    *   **Цель**: "Запустил и забыл". Используется, когда результат операции не нужен.
    *   **Возвращаемое значение**: Возвращает `Job`, который можно использовать для отслеживания состояния корутины (активна, завершена) или для ее отмены.
    *   **Исключения**: Необработанные исключения внутри `launch` "убивают" родительскую корутину.

*   **`async`**:
    *   **Цель**: Выполнить асинхронную операцию и получить от нее результат.
    *   **Возвращаемое значение**: Возвращает `Deferred<T>`, который является `Job` с результатом. Результат `T` получается вызовом `await()`.
    *   **Исключения**: Исключение хранится в `Deferred` и выбрасывается только при вызове `await()`.

**Когда что использовать**:
*   Используйте **`launch`** для операций, которые не возвращают результат (например, обновление данных в БД).
*   Используйте **`async`** для операций, результат которых нужен для дальнейшей работы (например, получение данных из сети для их последующей обработки). Можно использовать для параллельного выполнения нескольких запросов.
* **coroutineScope** объект, который управляет жизненным циклом корутин
---

## 40. Что такое ViewModelScope и зачем он нужен
**`viewModelScope`** — это `CoroutineScope`, привязанный к жизненному циклу `ViewModel`.

*   **Автоматическое завершение**: Все корутины, запущенные в `viewModelScope`, автоматически отменяются, когда `ViewModel` уничтожается (когда вызывается ее метод `onCleared()`).
*   **Зачем нужен**: Это обеспечивает безопасность и предотвращает утечки памяти и лишнюю работу. Если `ViewModel` больше не нужна (например, пользователь ушел с экрана), все запущенные в ней сетевые запросы или другие фоновые задачи будут автоматически остановлены.

**Пример**:
```kotlin
class MyViewModel : ViewModel() {
    fun fetchData() {
        // Эта корутина будет автоматически отменена,
        // когда ViewModel будет уничтожена.
        viewModelScope.launch {
            repository.longRunningOperation()
        }
    }
}
```

---

# ◆ DOMAIN

## 1. Что входит в слой domain и зачем он нужен?
В слой **domain** входят:
*   **Бизнес-модели (Entities)**: POJO/data классы, описывающие основные сущности бизнеса (например, `User`, `Product`).
*   **UseCases (Interactors)**: Классы, инкапсулирующие конкретные бизнес-правила и сценарии использования.
*   **Интерфейсы репозиториев**: Абстракции, которые определяют контракт для получения данных (`UserRepository`), скрывая детали их источника.

**Зачем нужен**: Это ядро приложения. Он инкапсулирует самую важную часть — бизнес-логику — и делает ее независимой от деталей реализации (UI, база данных, сеть). Это повышает тестируемость, масштабируемость и переиспользуемость кода.

## 2. Что такое UseCase и какую роль он играет в архитектуре?
**UseCase** (или Interactor) — это класс, который представляет один конкретный сценарий использования в приложении. Например, `GetUserProfileUseCase`, `AddItemToCartUseCase`.

**Роль**:
*   **Инкапсуляция бизнес-логики**: Содержит правила, которые должны быть выполнены для конкретной задачи.
*   **Разделение ответственности**: `ViewModel` не знает, *как* получить или обработать данные, она просто вызывает соответствующий `UseCase`.
*   **Переиспользуемость**: Один и тот же `UseCase` может использоваться разными `ViewModel`.
*   **Тестируемость**: `UseCase` легко тестировать изолированно, так как он зависит только от абстракций (интерфейсов репозиториев).

## 3. В каком слое описываются бизнес-правила?
Бизнес-правила описываются исключительно в **domain**-слое. В основном внутри **UseCases**.

## 4. Почему в domain не должен быть зависимостей от Android SDK?
`domain`-слой должен быть "чистым". Он не должен зависеть от фреймворков и библиотек, специфичных для платформы (Android, iOS, Web).

**Причины**:
*   **Переносимость**: "Чистый" `domain`-слой можно легко перенести на другую платформу (например, в KMM-проект для iOS или в десктопное приложение).
*   **Тестируемость**: Его можно тестировать на локальной JVM без необходимости запускать эмулятор Android.
*   **Долговечность**: Android SDK и библиотеки меняются часто. Бизнес-логика меняется гораздо реже. Отсутствие зависимостей делает ее более стабильной.

## 5. Что такое Repository interface в domain и зачем он нужен?
**Интерфейс репозитория** (`UserRepository`) — это абстракция в `domain`-слое, которая определяет контракт для работы с данными (`getUsers()`, `saveUser()`).

**Зачем он нужен**:
*   **Принцип инверсии зависимостей (DIP)**: `domain`-слой не зависит от `data`-слоя, а наоборот, `data`-слой зависит от абстракций в `domain`.
*   **Сокрытие деталей реализации**: `UseCase` не знает, откуда репозиторий берет данные — из сети, из кэша, из базы данных или из всего сразу. Он просто вызывает метод интерфейса. Это позволяет легко подменять источники данных.

## 6. Можно ли использовать ViewModel в domain? Почему?
**Нет, ни в коем случае.** `ViewModel` — это компонент, специфичный для Android и UI-слоя (`presentation`).

**Почему**:
*   Это нарушит главное правило Clean Architecture: `domain` не должен ничего знать о внешних слоях, особенно о UI.
*   Это создаст циклическую зависимость (`UI` -> `Domain` -> `UI`), что является анти-паттерном.
*   Это сделает `domain`-слой непереносимым и нетестируемым на JVM.

## 7. Какие зависимости допустимы в domain-слое?
Только:
*   Стандартная библиотека Kotlin/Java.
*   Библиотеки для корутин (`kotlinx.coroutines.core`).
*   Библиотеки для DI, которые не тянут за собой платформенные зависимости (например, `javax.inject`).
*   Любые другие "чистые" Kotlin/Java библиотеки.

**Никаких `android.*`, `androidx.*`, Retrofit, Room и т.д.**

## 8. Как обрабатывать ошибки в UseCase — лучше через sealed class, Result, Either?
Все три подхода хороши и преследуют одну цель — сделать обработку ошибок явной.

*   **`kotlin.Result`**: Встроенный в Kotlin. Удобен и прост. Имеет `onSuccess` и `onFailure`. Отличный выбор по умолчанию.
*   **Собственный `sealed class`**: Дает больше гибкости. Можно определить не только `Success` и `Error`, но и другие состояния, например, `Loading`, или более специфичные типы ошибок (`NetworkError`, `DatabaseError`).
*   **`Either` (из библиотеки Arrow)**: Функциональный подход (`Either<Error, Success>`). Мощный инструмент, но может быть избыточным для простых проектов и требует изучения библиотеки Arrow.

**Вывод**: **`kotlin.Result`** — самый прагматичный и простой выбор для большинства проектов. Собственный `sealed class` — если нужна большая кастомизация ошибок.

## 9. Почему важно, чтобы слой domain не зависел от слоёв ui и data?
Это **ключевой принцип** Clean Architecture. Зависимости направлены внутрь, к `domain`.

*   **Стабильность**: `domain` — самый стабильный слой. UI и источники данных меняются часто, бизнес-логика — редко.
*   **Заменяемость**: Можно легко заменить UI (с XML на Compose) или источник данных (с Retrofit на gRPC), не трогая `domain`-слой.
*   **Независимость разработки и тестирования**: `domain`-слой можно разрабатывать и тестировать полностью независимо от других слоев.

## 10. Как оформляется многократное использование UseCase из разных ViewModel?
Просто. `UseCase` не хранит состояние, он просто выполняет действие. Поэтому один и тот же экземпляр `UseCase` можно безопасно внедрять (через DI) и использовать в нескольких `ViewModel`.

**Пример**: `GetUserInfoUseCase` может использоваться в `UserProfileViewModel` и `UserEditViewModel`. Обе `ViewModel` получат его в качестве зависимости в конструкторе.

---

# ◆ DATA

## 11. Что входит в слой data?
В слой **data** входят:
*   **Реализации репозиториев**: Классы, которые реализуют интерфейсы репозиториев из `domain`-слоя.
*   **Источники данных (Data Sources)**: Классы, отвечающие за конкретный источник — сеть (Retrofit API Service), база данных (Room DAO), SharedPreferences.
*   **Мапперы (Mappers)**: Классы для преобразования моделей данных (например, `UserDto` из сети в `UserEntity` для БД, а затем в `User` для `domain`).
*   **Модели данных (DTO, Entity)**: Классы, специфичные для источников данных (`UserDto` для Retrofit, `UserEntity` для Room).

## 12. Как связаны data и domain слои?
`data`-слой **реализует** абстракции, определенные в `domain`-слое.
`Data` -> `Domain` (направление зависимости).

`UserRepositoryImpl` (в `data`) `implements` `UserRepository` (в `domain`).
Таким образом, `data` знает о `domain`, но `domain` ничего не знает о `data`.

## 13. Зачем интерфейсы Repository выносятся в domain, а реализации в data?
Это **Принцип инверсии зависимостей (DIP)**.
*   `domain` (высокоуровневый модуль) не должен зависеть от `data` (низкоуровневого модуля). Оба должны зависеть от абстракций.
*   **Интерфейс репозитория** — это и есть эта абстракция.
*   `domain` владеет контрактом (интерфейсом), а `data` предоставляет его реализацию. Это позволяет `domain` оставаться независимым от деталей получения данных.

## 14. Что такое мапперы (Mapper) и почему их размещают в data?
**Маппер** — это класс или функция, которая преобразует один тип объекта в другой.

В `data`-слое мапперы нужны для преобразования между:
*   DTO (Data Transfer Object) из сети -> `domain`-модель.
*   Entity из БД -> `domain`-модель.
*   `domain`-модель -> Entity для сохранения в БД.

Они находятся в `data`-слое, потому что `data`-слой знает обо всех типах моделей (`DTO`, `Entity`, `domain`-модель). `domain`-слой должен работать только со своими собственными моделями и не знать о `DTO` или `Entity`.

## 15. Как связаны DTO, Entity, Model между слоями?
*   **`DTO` (Data Transfer Object)**: Модель для сетевого слоя. Ее структура соответствует ответу от API. Находится в `data`.
*   **`Entity`**: Модель для слоя базы данных (Room). Ее структура соответствует таблице в БД. Находится в `data`.
*   **`Model` (или `Business Object`)**: Модель для `domain`-слоя. Отражает бизнес-сущность. Используется в `UseCase` и `ViewModel`. Находится в `domain`.

**Поток данных (из сети в UI)**:
`Retrofit API` -> `DTO` -> `Mapper` -> `Domain Model` -> `UseCase` -> `ViewModel` -> `UI`

## 16. Как из слоя data получить данные из сети и отправить в domain?
1.  **RepositoryImpl** (в `data`) имеет зависимость от `ApiService` (Retrofit).
2.  `RepositoryImpl` вызывает метод `ApiService` для получения `DTO`.
3.  `RepositoryImpl` использует **маппер** для преобразования `DTO` в `domain`-модель.
4.  `RepositoryImpl` возвращает `domain`-модель, как того требует интерфейс репозитория из `domain`-слоя.

## 17. Где хранить логику кэширования: в data или domain?
Логика кэширования (решение, когда идти в сеть, а когда взять из БД) — это деталь реализации. Поэтому она должна быть инкапсулирована в **data**-слое, внутри **реализации репозитория**.

`UseCase` в `domain`-слое не должен об этом беспокоиться. Он просто вызывает `userRepository.getUsers()`, а уже `UserRepositoryImpl` решает, вернуть ли свежие данные из сети (и сохранить их в БД) или отдать закэшированные.

## 18. Как правильно работать с Room в Clean Architecture?
*   **DAO** и **Database** классы находятся в `data`-слое.
*   **Entity**-классы для Room также находятся в `data`-слое.
*   `RepositoryImpl` имеет зависимость от **DAO**.
*   Когда `RepositoryImpl` получает данные из DAO (в виде `Entity`), он **маппит** их в `domain`-модели перед тем, как вернуть их в `UseCase`.

## 19. Можно ли в data слое использовать UseCase? Почему?
**Нет, никогда.**
Это создаст циклическую зависимость: `UseCase` (в `domain`) зависит от `Repository` (интерфейс в `domain`), который реализуется в `data`. Если `data` будет зависеть от `UseCase`, получится `Domain -> Data -> Domain`.

`UseCase` — это потребитель данных, а `data`-слой — их поставщик. Поток однонаправленный.

## 20. В каком слое реализуется логика работы с API и базой данных?
В **data**-слое. Это его прямая ответственность — быть мостом между приложением и внешними источниками данных.

---

# ◆ UI (presentation)

## 21. Что входит в ui слой (или presentation)?
В слой **UI (Presentation)** входят:
*   **View**: `Activity`, `Fragment`, `Composable`-функции. Отвечают только за отображение данных и передачу событий.
*   **ViewModel**: Хранит состояние UI (`UiState`), обрабатывает события от `View` и вызывает `UseCase`.
*   **UI-модели (UiModel)**: Классы данных, специально подготовленные для отображения (например, отформатированные строки, ресурсы).
*   **DI-компоненты** для UI-слоя.

## 22. Почему ViewModel — часть ui слоя?
Потому что `ViewModel` тесно связана с жизненным циклом и потребностями UI.
*   Она хранит **состояние UI**, которое переживает изменения конфигурации.
*   Она предоставляет данные в формате, удобном для **отображения** (`LiveData`, `StateFlow`).
*   Она готовит данные специально для `View`. `ViewModel` не содержит бизнес-логики, а только **логику отображения**.

## 23. Как ViewModel взаимодействует с UseCase?
`ViewModel` получает `UseCase` в качестве зависимости через конструктор (DI).
Когда от `View` поступает событие (например, нажатие кнопки), `ViewModel`:
1.  Запускает корутину в `viewModelScope`.
2.  Вызывает нужный метод `UseCase`.
3.  Получает результат от `UseCase` (`Result<DomainModel>`).
4.  Обрабатывает результат и обновляет свое состояние (`UiState`).

## 24. Что такое состояние UI (UiState) и зачем его выделяют?
**UiState** — это, как правило, `data class` или `sealed class`, который представляет все, что нужно для отрисовки одного экрана в любой момент времени.

**Пример**:
```kotlin
data class ProfileUiState(
    val isLoading: Boolean = false,
    val userName: String? = null,
    val error: String? = null
)
```

**Зачем выделяют**:
*   **Единый источник правды**: У экрана есть только один объект состояния. Это упрощает логику и отладку.
*   **Атомарные обновления**: Все состояние экрана обновляется одновременно, что предотвращает рассинхронизацию UI.
*   **Предсказуемость**: Легко понять, как будет выглядеть экран, просто посмотрев на объект `UiState`.
*   **Тестируемость**: Легко тестировать `ViewModel`, проверяя, как ее методы меняют `UiState`.

## 25. Где и как лучше обрабатывать ошибки, приходящие из UseCase?
Ошибки обрабатываются в **ViewModel**.

**Как**:
`UseCase` возвращает `Result<T>` (`Success` или `Failure`). `ViewModel` использует `when` или `onSuccess/onFailure` для обработки:
*   В случае `Success` -> обновить `UiState` данными.
*   В случае `Failure` -> извлечь ошибку, преобразовать ее в понятное для пользователя сообщение (например, "Нет интернета") и обновить `UiState` этим сообщением об ошибке.

`View` просто наблюдает за полем `error` в `UiState` и показывает `Snackbar` или `Toast`, если оно не `null`.

## 26. Можно ли использовать LiveData/StateFlow в domain? Почему это ошибка?
**Нет.** `LiveData` и `StateFlow` — это инструменты для Android/UI слоя.

**Почему это ошибка**:
*   **Нарушение чистоты `domain`**: `LiveData` — это часть Android SDK. `StateFlow` — это "горячий" поток, предназначенный для разделения состояния с UI. `domain`-слою это не нужно.
*   **Неправильная ответственность**: `domain` не должен заботиться о том, как данные будут наблюдаться в UI. Он просто должен предоставлять данные (или `Flow` для реактивных источников). `ViewModel` уже решает, как их представить для `View` (например, преобразовав холодный `Flow` в горячий `StateFlow`).

## 27. Как в ViewModel вызвать несколько UseCase?
Просто вызвать их последовательно или параллельно, в зависимости от задачи.

*   **Последовательно**:
    ```kotlin
    viewModelScope.launch {
        val user = getUserUseCase()
        val settings = getSettingsUseCase(user.id)
        // ...
    }
    ```
*   **Параллельно** (если они не зависят друг от друга, для скорости):
    ```kotlin
    viewModelScope.launch {
        val userDeferred = async { getUserUseCase() }
        val configDeferred = async { getConfigUseCase() }
        val user = userDeferred.await()
        val config = configDeferred.await()
        // ...
    }
    ```

## 28. Где хранятся UI-модели (UiModel) и чем они отличаются от Entity или DTO?
**UiModel** хранятся в **ui (presentation)** слое.

**Отличия**:
*   **Назначение**: `UiModel` создана специально для **отображения**. Она может содержать уже отформатированные строки, ссылки на ресурсы (`R.string.xxx`), цвета (`R.color.xxx`).
*   **Структура**: Ее структура оптимизирована для конкретного экрана, а не для API или таблицы БД.
*   **Пример**:
    *   `Domain Model`: `User(birthDate: Long)`
    *   `UiModel`: `UserUi(age: String)` -> "Возраст: 25 лет"
Маппинг из `DomainModel` в `UiModel` происходит в `ViewModel`.

## 29. Как разделить логику бизнес-процесса и UI-обработку в ViewModel?
Это главная задача `ViewModel` в связке с `UseCase`.

*   **Бизнес-логика**: Полностью делегируется **`UseCase`**. `ViewModel` не знает, *как* получить или обработать данные, она просто вызывает соответствующий `UseCase`.
*   **UI-обработка (логика представления)**: Остается в **`ViewModel`**. Это включает:
    *   Вызов `UseCase`.
    *   Обработка результата (`Success`/`Error`).
    *   Маппинг `domain`-моделей в `UI`-модели.
    *   Обновление `UiState`.

Таким образом, `ViewModel` выступает как **дирижер** между `View` и `domain`-слоем.
