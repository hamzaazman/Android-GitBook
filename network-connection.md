# Network Connection

### ConnectionManager.kt

```kotlin
class ConnectionManager(context: Context, private val externalScope: CoroutineScope) {

    private val connectivityManager = context.getSystemService(ConnectivityManager::class.java)

    val connectionAsStateFlow: StateFlow<Boolean>
        get() = _connectionFlow
            .stateIn(
                scope = externalScope,
                started = SharingStarted.WhileSubscribed(5000),
                initialValue = isConnected
            )

    private val _connectionFlow = callbackFlow {
        val networkCallback = object : ConnectivityManager.NetworkCallback() {
            override fun onLost(network: Network) {
                trySend(false)
            }

            override fun onCapabilitiesChanged(
                network: Network,
                networkCapabilities: NetworkCapabilities
            ) {
                if (networkCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
                    && networkCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)
                ) {
                    trySend(true)
                }
            }
        }
        subscribe(networkCallback)
        awaitClose {
            unsubscribe(networkCallback)
        }
    }

    private val isConnected: Boolean
        get() {
            val activeNetwork = connectivityManager.activeNetwork
            return if (activeNetwork == null) {
                false
            } else {
                val netCapabilities = connectivityManager.getNetworkCapabilities(activeNetwork)
                (netCapabilities != null
                        && netCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
                        && netCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED))
            }
        }

    private fun subscribe(networkCallback: ConnectivityManager.NetworkCallback) {
        connectivityManager.registerDefaultNetworkCallback(networkCallback)
    }

    private fun unsubscribe(networkCallback: ConnectivityManager.NetworkCallback) {
        connectivityManager.unregisterNetworkCallback(networkCallback)
    }
}
```

### MainActivity.kt

```kotlin

class MainActivity : ComponentActivity() {

    private val myConnectivityManager by lazy { ConnectionManager(this, lifecycleScope) }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            val connectionState by myConnectivityManager.connectionAsStateFlow.collectAsState()

            NetworkConnectionTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    ConnectivityUiView(
                        isOnline = connectionState,
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }

```

### HomeScreen.kt

```kotlin
@Composable
fun HomeScreen(isOnline: Boolean, modifier: Modifier = Modifier) {
    val transition = updateTransition(targetState = isOnline, label = "Online Status Transition")

    val bgColor by transition.animateColor(label = "Background Color") { online ->
        if (online) Color(0xFF4CAF50) else Color(0xFF9E9E9E)
    }

    val msgStr = stringResource(
        id = if (isOnline) {
            R.string.you_are_online_msg
        } else {
            R.string.you_are_offline_msg
        }
    )

    Card(
        modifier = modifier
            .fillMaxWidth()
            .padding(16.dp),
        shape = RoundedCornerShape(8.dp),
        colors = androidx.compose.material3.CardDefaults.cardColors(
            containerColor = bgColor
        ),
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.Center
        ) {
            Crossfade(targetState = isOnline, label = "") { isOnline ->
                val icon = if (isOnline) R.drawable.online else R.drawable.offline

                Icon(
                    contentDescription = null,
                    tint = Color.White,
                    modifier = Modifier.padding(end = 8.dp),
                    painter = painterResource(id = icon)
                )
            }
            Text(
                text = msgStr,
                style = TextStyle(color = Color.White, fontWeight = FontWeight.Bold),
                textAlign = TextAlign.Center
            )
        }
    }
}
```

### Demo

{% embed url="https://github.com/user-attachments/assets/1c5a0ac5-4f5c-4361-88c2-02de20e05bdd" %}

### Github

{% embed url="https://github.com/hamzaazman/NetworkConnection" %}
