## Getting data from the internet

1. ### Добавляем в  `build.gradle` (Module: app)  зависимости:

   ```groovy
   //retrofit2
   implementation "com.squareup.retrofit2:retrofit:$version_retrofit"
   // implementation "com.squareup.retrofit2:converter-scalars:$version_retrofit" - конвертирование в строку
   
   //moshi-синтаксический анализатор JSON для Android
   implementation "com.squareup.moshi:moshi-kotlin:$version_moshi"
   implementation "com.squareup.retrofit2:converter-moshi:$version_retrofit"
   
   //Glide - для загрузки, буферизации, декодирования и кеширования изображений
   implementation "com.github.bumptech.glide:glide:$version_glide"
   
   ```

2. ### Создаем class `NameApiService`  - это API для связи с веб-службой

   ```kotlin
   private const val BASE_URL = "https://android-kotlin-fun-mars-server.appspot.com/"
   
   private val moshi = Moshi.Builder()
      .add(KotlinJsonAdapterFactory())
      .build()
   
   private val retrofit = Retrofit.Builder()
   //    .addConverterFactory(ScalarsConverterFactory.create()) - конвертирование в строку
       .addConverterFactory(MoshiConverterFactory.create(moshi)) 
       .baseUrl(BASE_URL)
       .build()      
   
   interface MarsApiService{
       @GET("realestate")
   	suspend fun getProperties(): List<MarsProperty>
   }
   
   object NameApi {
       val retrofitService : NameApiService by lazy { 
          retrofit.create(NameApiService::class.java) }
   }
   ```

   - созданиe объекта Moshi (см п.5)

   - созданиe объекта Retrofit 

   - #### добавляем интерфейс, который определяет, как Retrofit взаимодействует с веб-сервером с помощью HTTP-запросов.

     > Чтобы сообщить Retrofit, что должен делать этот метод, используйте  **@GET**  аннотацию и укажите путь или конечную точку для этого метода веб-службы. В этом случае вызывается конечная точка ***realestate***. Когда `getProperties()`метод вызывается, Retrofit добавляет конечную точку `realestate`к базовому URL-адресу (который вы определили в построителе Retrofit) и создает объект. Этот объект используется для запуска запроса.
     >
     >  **Call<String>** - Вызов метода Retrofit, который отправляет запрос на веб-сервер и возвращает ответ

   - ####  Создаем общедоступный объект, вызываемый `NameApi`для инициализации службы Retrofit. Это стандартный шаблон кода K`otlin` для использования при создании объекта службы.

     >  каждый раз, когда ваше приложение вызывает `NameApi.retrofitService`, оно будет получать одноэлементный объект Retrofit, который реализует `NameApiService`
     >
     > **create()** - создает сам сервис Retrofit с `NameApiService`интерфейсом

3. ### Вызов веб-службы в `ViewModel`

   > `_response` - `LiveData` строка , которая определяет , что отображается в текстовом виде .

   ```kotlin
   class OverviewViewModel : ViewModel() {
       // Строка MutableLiveData, в которой хранится самый последний ответ
       private val _response = MutableLiveData<String>()
   
       // неизменяемые LiveData для строки ответа
       val response: LiveData<String>
           get() = _response
   
       init {
           getProperties()
       }
       
       //метод, в котором вы вызываеется служба Retrofit и обрабатывается возвращенная строка JSON
        private fun getProperties() {
                   viewModelScope.launch {
               try {
                   val listResult = NameApi.retrofitService.getProperties()
                   _response.value =
                       "Success: ${listResult.size} Mars properties retrieved"
               } catch (e: Exception) {
                   _response.value = "Failure: ${e.message}"
               }
           }
       }
   }
   
   ```

   ​	создаем метод , в котором вы вызываеется служба Retrofit и обрабатывается возвращенная строка JSON

   - import retrofit2.Callback

   - `enqueue()` - этот объект, чтобы запустить сетевой запрос в фоновом потоке

   -  **Implement methods**  `onResponse()`   и`onFailure()` 

     > `onFailure()` -  вызывается , когда ответ веб - службы не удается	
     >
     > `onResponse()` - вызывается , когда запрос успешно и веб - сервис возвращает ответ

4. ### Добавляем разрешения в Манифест

   > Добавьте эту строку непосредственно перед `<application>`тегом:

   ```xml
   <uses-permission android:name="android.permission.INTERNET" />
   ```

5. ### Обработка данных JSON

   *  [Moshi](https://github.com/square/moshi)  - синтаксический анализатор JSON для Android, который преобразует строку JSON в объекты `Kotlin`

     > Moshi анализирует данные JSON и преобразует их в объекты `Kotlin`. Для этого ему нужен класс данных `Kotlin` для хранения проанализированных результатов, поэтому следующим шагом будет создание этого класса.
     
   * Создаем класс `NameProperty`
   
     ```
     data class NameProperty(
      val id: String,
         @Json(name = "img_src") val imgSrcUrl: String,
      val type: String,
         val price: Double
  )
     ```
   
     > *  Класс создаем на основании полученного JSON объекта
     >*  Значения могут быть числами, строками и логическими значениями, а также другими объектами или массивами
     > *  каждая переменная в `NameProperty`классе соответствует имени ключа в объекте JSON
     >* `Double`может использоваться для обозначения любого номера JSON
     > * Когда Moshi анализирует JSON, он сопоставляет ключи по имени и заполняет объекты данных соответствующими значениями.
     >* Чтобы использовать имена переменных в вашем классе данных, которые отличаются от имен ключей в ответе JSON, используйте `@Json`аннотацию.

6. ###  **Создайте адаптера привязки     BindingAdapters.kt    и вызов Glide**

   

   

