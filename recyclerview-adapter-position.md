# RecyclerView Adapter Position

{% hint style="info" %}
Öncelikle istenilen koşulu hatırlayalım, recyclerview içerisindeki listenin pozisyonuna erişip sadece bir item' a tıklama event' i eklemekti.
{% endhint %}

Bunun için adapter içerisinde bir OnItemClickListener adında interface oluşturuyorum ve bunun içerisine de holder tarafında vereceğimiz adapter pozisyonu için parametre atıyorum.

{% code title="ExampleAdapter.kt" %}
```kotlin
interface OnItemClickListener {
    fun onIconClickListener(clickedItemPosition: Int)
 }
```
{% endcode %}

bu sefer viewholder tarafında da vermemiz gerekiyor ve tasarım kısmında imageview' a tıklanıldığında adapter içerisindeki posizyonu da vermiş oluyoruz.

{% code title="ExampleAdapter.kt" %}
```kotlin
class ExampleAdapter(
    private val listener: OnItemClickListener,
    private val list: List<Example>
) : RecyclerView.Adapter<ExampleAdapter.ExampleViewHolder>() {

class ExampleViewHolder(
        private val binding: CustomExampleRowBinding,
        private val listener: OnItemClickListener
    ) : ViewHolder(binding.root) {

        fun bind(example: Example) {
            with(binding) {
                exampleTextView.text = example.name

                exampleImageView.setOnClickListener {
                    listener.onIconClickListener(adapterPosition)
                }
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ExampleViewHolder {
        return ExampleViewHolder(
            CustomExampleRowBinding.inflate(
                LayoutInflater.from(parent.context), parent, false
            ), listener
        )
    }
}
```
{% endcode %}

Şimdi fragmente bu adapterdaki listeneri tanımlayarak override edelim

{% code title="HomeFragment.kt" %}
```kotlin
class HomeFragment : Fragment(), ExampleAdapter.OnItemClickListener {}
```
{% endcode %}

Geriye aldığımız pozisyona göre yapacağımız eylemleri yazmak kalıyor. İster logout yaparsın ister detay sayfasına gidersin. Spesifik bir item'a erişmeyi bu şekilde yapabildim.

{% code title="HomeFragment.kt" %}
```kotlin
 override fun onIconClickListener(clickedItemPosition: Int) {
        when (clickedItemPosition) {
            0 -> {
                Toast.makeText( requireContext(),"pozisyon $clickedItemPosition ",Toast.LENGTH_SHORT).show()
            }
            1 -> {
                Toast.makeText(requireContext(), "pozisyon $clickedItemPosition ", Toast.LENGTH_SHORT).show()
            }
            2 -> {
                Toast.makeText(requireContext(), "pozisyon $clickedItemPosition ", Toast.LENGTH_SHORT).show()
            }
            3 -> {
                Toast.makeText(requireContext(), "pozisyon $clickedItemPosition ",Toast.LENGTH_SHORT).show()
            }
        }
    }
```
{% endcode %}

#### Ad

### Adapterın Tamamı

```kotlin
class ExampleAdapter(
    private val listener: OnItemClickListener,
     private val list: List<Example>
) : RecyclerView.Adapter<ExampleAdapter.ExampleViewHolder>() {

    class ExampleViewHolder(
        private val binding: CustomExampleRowBinding,
         private val listener: OnItemClickListener
    ) : ViewHolder(binding.root) {

        fun bind(example: Example) {
            with(binding) {
                exampleTextView.text = example.name

                exampleImageView.setOnClickListener {
                    listener.onIconClickListener(adapterPosition)
                }
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ExampleViewHolder {
        return ExampleViewHolder(
            CustomExampleRowBinding.inflate(
                LayoutInflater.from(parent.context), parent, false
            ), listener
        )
    }

    override fun onBindViewHolder(holder: ExampleViewHolder, position: Int) {
        val currentList = list[position]
        holder.bind(currentList)
    }

    override fun getItemCount(): Int = list.size

    interface OnItemClickListener {
        fun onIconClickListener(clickedItemPosition: Int)
    }

}
```

### HomeFragment Tamamı

```kotlin
with(binding!!) {
            val list = listOf(
                Example(id = 1, name = "Deneme"),
                Example(id = 2, name = "Deneme"),
                Example(id = 3, name = "Deneme"),
                Example(id = 4, name = "Deneme"),
            )
            exampleAdapter = ExampleAdapter(this@HomeFragment, list)

            exampleRecyclerView.apply {
                adapter = exampleAdapter
                layoutManager = LinearLayoutManager(requireContext())
            }

```
