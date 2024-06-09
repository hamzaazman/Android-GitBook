# Room Local DB

Android de room kullanarak local veritabanları tanımlaması yapmak için var olan db dosyasını assest içerisine atıp ardından createFromAssest içerisinde çağırıp room' a local veritabanını tanımlayabiliriz.

```kotlin
@Database(entities = [FoodProduct::class, Meal::class], version = 1, exportSchema = true)
@TypeConverters(Converters::class)
abstract class FoodDatabase : RoomDatabase() {

    abstract fun getProductDao(): FoodProductDao

    companion object {

        private const val FOOD_DATABASE_NAME = "foodProducts"
        private const val FOOD_DATABASE_PATH = "databases/food.db"

        @Volatile
        private var instance: FoodDatabase? = null
        private val LOCK = Any()

        operator fun invoke(context: Context) = instance

            ?: synchronized(LOCK) {
                instance
                    ?: buildDatabase(
                        context
                    ).also { instance = it }
            }

        private fun buildDatabase(context: Context) = Room
            .databaseBuilder(
                context.applicationContext,
                FoodDatabase::class.java,
                FOOD_DATABASE_NAME
            )
            .createFromAsset(FOOD_DATABASE_PATH)
            .build()

    }


```

Assest klasörün yolunu tanımlayarak bunu createFromAsset içerisinde rooma tanımlamış olduk. Bu şekilde hazır verilerimizi room' a yükleyebiliriz.
