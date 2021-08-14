1. ##### Добавляем зависимости в  ***build.gradle(app)***

​	[Declaring dependencies](https://developer.android.com/jetpack/androidx/releases/room)

2. ##### Data class 

```kotlin
@Entity(tableName = "tableName")
data class NameClass(
    @PrimaryKey(autoGenerate = true) var classID: Int = 0,
    @ColumnInfo(name = "name") var name: String?
    )
```

3. ##### ClassDAO

```kotlin
@Dao
interface NameDAO {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(vararg nameaArray: NameDataClass)

    @Update
    suspend fun update(name: NameDataClass)

    @Query("SELECT * from question WHERE id = :key")
    suspend fun get(key: Int): NameDataClass?

    @Delete
    suspend fun delete(name: NameDataClass)

    @Query("DELETE FROM elements")
    suspend fun clear()

    @Query("SELECT * FROM question")
    fun getAllElements(): Flow<List<NameDataClass>>
}
```

4. ##### NameDB

*@TypeConverters(Converters::class)* -  если необходимо сохранять массив в DB 

*class QuestionDBCallback*   - загрузка данных в DB при первом запуске

*class Converters* - конвертируем массив в String и обратно

> ```kotlin
> @Database(entities = [NameClass::class], version = 1, exportSchema = false)
> @TypeConverters(Converters::class)
> abstract class NameDB : RoomDatabase() {
> 
>     abstract fun nameDAO(): NameDAO
> 
>     companion object {
>         @Volatile
>         private var INSTANCE: NameDB? = null
> 
>         fun getInstance(
>             context: Context,
>             scope: CoroutineScope
>         ): NameDB {
>             Log.i("MyLog", "getInstance")
>             synchronized(this) {
>                 var instance = INSTANCE
> 
>                 if (instance == null) {
>                     instance = Room.databaseBuilder(
>                         context.applicationContext,
>                         NameDB::class.java,
>                         "tableName"
>                     )
>                         .addCallback(NameDBCallback(scope)) // стартовые данные
>                         .fallbackToDestructiveMigration()
>                         .build()
>                     INSTANCE = instance
>                 }
>                 return instance
>             }
>         }
>     }
> 
>     private class NameDBCallback(
>         private val scope: CoroutineScope
>     ) : RoomDatabase.Callback() {
>         override fun onCreate(db: SupportSQLiteDatabase) {
>             super.onCreate(db)
>             INSTANCE?.let { database ->
>                 scope.launch {
>                     var namenDAO = database.nameDAO()
>                     questionDAO.clear()
>                     val nameClass = Array c данными
>                     nameClass.forEach { namenDAO.insert(it) }
>                 }
>             }
>         }
>     }
> }
> 
> 
> //JSON
>     implementation "org.jetbrains.kotlinx:kotlinx-serialization-json:1.0.1"
> 
> class Converters {
> 
>     @TypeConverter
>     fun fromList(value: ArrayList<String>?): String {
>         return Json.encodeToString(value)
>     }
> 
>     @TypeConverter
>     fun toList(value: String): ArrayList<String>? {
>         return Json.decodeFromString(value)
>     }
> }
> ```



5. ##### Reposytory

```kotlin
class NameRepository(private val nameDAO: NameDAO) {

    val allelements: Flow<List<NameClass>> = nameDAO.getAllElements()

    @Suppress("RedundantSuspendModifier")
    @WorkerThread
    suspend fun insert( nameClass: NameClass) {
        nameDAO.insert( nameClass)
    }
}
```

6. ##### ViewModel

```kotlin
class NameViewModel (
    private val repository: NamenRepository,
    application: Application
) : AndroidViewModel(application) {

    val elements: LiveData<List<NameClass>> = repository.allElements.asLiveData()

    fun insert(nameClass: NameClass) = viewModelScope.launch {
        repository.insert(nameClass)
    }

}
```



7. ##### Factory

```kotlin
class NameFactory (
    private val repository: NameRepository,
    private val application: Application
) : ViewModelProvider.Factory {

    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(StartViewModel::class.java)) {
            return NameViewModel(repository, application) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

8. #####  Fragment

   ```kotlin
   class StartFragment : Fragment() {
       private val applicationScope = CoroutineScope(SupervisorJob())
       private val database by lazy { NameDB.getInstance(requireActivity(), applicationScope) }
       private val repository by lazy { NameRepository(database.NameDAO()) }
       private var _binding: FragmentNameBinding? = null
       private val binding get() = _binding!!
       private lateinit var elements: List<Element>
   
   
       override fun onCreateView(
           inflater: LayoutInflater, container: ViewGroup?,
           savedInstanceState: Bundle?
       ): View? {
           _binding = FragmentNameBinding.inflate(inflater, container, false)
           val application = requireNotNull(this).activity?.application
           val viewModelFactory = application?.let { NameFactory(repository, it) }
           val mainViewModel =
               viewModelFactory?.let { ViewModelProvider(this, it) }?.get(NameViewModel::class.java)
           mainViewModel?.elements?.observe(viewLifecycleOwner, Observer {
               elements = it
           })
   ```

