# MVVM - Boilerplate

### Type Converter

{% code overflow="wrap" %}
```kotlin
class Converter {

    @TypeConverter
    fun fromListOfStringsToString(value: List<String>): String {
        val gson = Gson()
        val type = object : TypeToken<List<String>>() {}.type
        return gson.toJson(value, type)
    }

    @TypeConverter
    fun fromStringToListOfStrings(value: String): List<String> {
        val gson = Gson()
        val type = object : TypeToken<List<String>>() {}.type
        return gson.fromJson(value, type)
    }
}
```
{% endcode %}

### Network Module

{% code overflow="wrap" %}
```kotlin
@Module
@InstallIn(SingletonComponent::class)
class NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(client: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(client)
            .addConverterFactory(MoshiConverterFactory.create())
            .build()

    }
    
    @Provides
    @Singleton
    fun provideService(retrofit: Retrofit): PostApi {
        return retrofit.create(PostApi::class.java)
    }
}
```
{% endcode %}

## Handle Internet Connection

```kotlin
    private fun hasInternetConnection(): Boolean {
        val connectivityManager =
            getApplication<Application>().getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        val activeNetwork = connectivityManager.activeNetwork ?: return false
        val capabilities = connectivityManager.getNetworkCapabilities(activeNetwork) ?: return false
        return when {
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> true
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> true
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_ETHERNET) -> true
            else -> false
        }
    }
```

### Check Response Data

{% code overflow="wrap" %}
```kotlin
   private fun handleProductResponse(response: Response<Product>): NetworkResult<Product>? {
        when {
            response.message().contains("timeout") -> {
                return NetworkResult.Error("TimeOut")
            }
            response.code() == 402 -> {
                return NetworkResult.Error("API Key Limited")
            }
            response.body()!!.results.isEmpty() -> {
                return NetworkResult.Error("Product not found")
            }
            response.isSuccessful -> {
                return NetworkResult.Success(response.body()!!)
            }
            else -> {
                return NetworkResult.Error(response.message())
            }
        }
    }
```
{% endcode %}

### Coroutine Scope And Exception Handler

```kotlin
    val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
        onError("Exception handled: ${throwable.localizedMessage}")
    }
```

```kotlin
    CoroutineScope(Dispatchers.IO + exceptionHandler).launch {
        val response = mainRepository.getAllMovies()  
    }
```

### AsynListDiffer - DifferCallback

```kotlin
    private val differCallback = object : DiffUtil.ItemCallback<Result>() {
        override fun areItemsTheSame(oldItem: Result, newItem: Result): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: Result, newItem: Result): Boolean {
            return oldItem == newItem
        }

    }

    val differ = AsyncListDiffer(this, differCallback)
```

### Network Bound Resource&#x20;

```kotlin
inline fun <T, K> networkBoundResource(
    crossinline query: () -> Flow<T>,
    crossinline fetch: suspend () -> K,
    crossinline saveFetchResult: suspend (K) -> Unit,
    crossinline shouldFetch: (T) -> Boolean = { true }
) = channelFlow<Resource<*>> {
    val data = query().first()
    send(Resource.loading(null))

    if (shouldFetch(data)) {
        try {
            saveFetchResult(fetch())
            try {
                query().collect { send(Resource.success(it)) }

            } catch (e: Exception) {
                send(Resource.error(data = null, message = e.localizedMessage ?: "Error!"))
            }
        } catch (e: Exception) {
            try {
                query().collect {
                    send(
                        Resource.error(
                            data = null,
                            message = e.localizedMessage ?: "Error $it"
                        )
                    )
                }
            } catch (e: Exception) {
                send(Resource.error(data = null, message = e.localizedMessage ?: "Error"))
            }
        }
    } else {
        try {
            query().collect { send(Resource.success(it)) }
        } catch (e: Exception) {
            send(Resource.error(data = null, message = e.localizedMessage ?: "Error"))
        }
    }
}
```
