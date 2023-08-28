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



## ViewBinding Delegate

```kotlin
/** Activity binding delegate, may be used since onCreate up to onDestroy (inclusive) */
inline fun <T : ViewBinding> AppCompatActivity.viewBinding(crossinline factory: (LayoutInflater) -> T) =
    lazy(LazyThreadSafetyMode.NONE) {
        factory(layoutInflater)
    }

/** Fragment binding delegate, may be used since onViewCreated up to onDestroyView (inclusive) */
fun <T : ViewBinding> Fragment.viewBinding(factory: (View) -> T): ReadOnlyProperty<Fragment, T> =
    object : ReadOnlyProperty<Fragment, T>, DefaultLifecycleObserver {
        private var binding: T? = null

        override fun getValue(thisRef: Fragment, property: KProperty<*>): T =
            binding ?: factory(requireView()).also {
                // if binding is accessed after Lifecycle is DESTROYED, create new instance, but don't cache it
                if (viewLifecycleOwner.lifecycle.currentState.isAtLeast(Lifecycle.State.INITIALIZED)) {
                    viewLifecycleOwner.lifecycle.addObserver(this)
                    binding = it
                }
            }

        override fun onDestroy(owner: LifecycleOwner) {
            binding = null
        }
    }

/** Binding delegate for DialogFragments implementing onCreateDialog (like Activities, they don't
 *  have a separate view lifecycle), may be used since onCreateDialog up to onDestroy (inclusive) */
inline fun <T : ViewBinding> DialogFragment.viewBinding(crossinline factory: (LayoutInflater) -> T) =
    lazy(LazyThreadSafetyMode.NONE) {
        factory(layoutInflater)
    }

/** Not really a delegate, just a small helper for RecyclerView.ViewHolders */
inline fun <T : ViewBinding> ViewGroup.viewBinding(factory: (LayoutInflater, ViewGroup, Boolean) -> T) =
    factory(LayoutInflater.from(context), this, false)
```

## Network Resource

```kotlin
sealed class Resource<T> {

    data class Success<T>(val data: T) : Resource<T>()

    data class Error<T>(val exception: Throwable) : Resource<T>()

    data class Loading<T>(val data: T? = null) : Resource<T>()

    fun <R> mapData(transform: (T) -> R): Resource<R> = when (this) {
        is Success -> Success(
            transform(data)
        )

        is Error -> Error(
            exception
        )

        is Loading -> Loading(
            data?.let { transform(it) }
        )
    }
}


sealed class Status {
    object Content : Status()
    data class Error(val exception: Throwable) : Status()
    object Loading : Status()
    object ContentWithLoading : Status()
}

class ResourceError(
    @StringRes
    val actionText: Int,
    @StringRes
    val errorTitle: Int,
    @StringRes
    val errorDesc: Int
)

class NoContentListingException : Exception()

object ErrorResolver {

    fun resolve(throwable: Throwable): ResourceError {
        return when (throwable) {
            is NoContentListingException -> ResourceError(
                actionText = R.string.no_content_error_action,
                errorDesc = R.string.no_content_error_message,
                errorTitle = R.string.default_error_title
            )

            else ->
                ResourceError(
                    actionText = R.string.common_action_retry,
                    errorDesc = R.string.default_error_message,
                    errorTitle = R.string.default_error_title
                )
        }
    }
}

class StatusViewState(private val status: Status) {

    fun getStateInfo(context: Context): StateLayout.StateInfo {
        return when (status) {
            Status.Content -> StateLayout.provideContentStateInfo()
            is Status.Error -> provideErrorState(context, status.exception)
            Status.Loading -> StateLayout.provideLoadingStateInfo()
            Status.ContentWithLoading -> StateLayout.provideLoadingWithContentStateInfo()
        }
    }

    private fun provideErrorState(context: Context, exception: Throwable): StateLayout.StateInfo {
        val resourceError = ErrorResolver.resolve(exception)
        return StateLayout.StateInfo(
            infoImage = R.drawable.state_info,
            infoTitle = context.getString(resourceError.errorTitle),
            infoMessage = context.getString(resourceError.errorDesc),
            infoButtonText = context.getString(resourceError.actionText),
            state = StateLayout.State.ERROR
        )
    }
}
```
