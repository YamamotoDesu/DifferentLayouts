# DifferentLayouts

activity_creature.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.raywenderlich.android.creatures.ui.CreatureActivity">

    <!-- 中略　-->

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/foodRecyclerView"
        android:layout_width="0dp"
        android:layout_height="@dimen/list_item_food_height"
        android:layout_marginTop="@dimen/padding_standard"
        android:layout_marginBottom="@dimen/padding_standard"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/planet" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

list_item_food.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="@dimen/list_item_food_height"
    android:layout_height="@dimen/list_item_food_height"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <ImageView
        android:id="@+id/foodImage"
        android:layout_width="@dimen/list_item_food_height"
        android:layout_height="@dimen/list_item_food_height"
        android:contentDescription="@string/content_description_food_image"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:srcCompat="@drawable/food_banana"
        />
</androidx.constraintlayout.widget.ConstraintLayout>
```

FoodAdapter
```kt
class FoodAdapter(private val foods: MutableList<Food>) : RecyclerView.Adapter<FoodAdapter.ViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): FoodAdapter.ViewHolder {
        return ViewHolder(parent.inflate(R.layout.list_item_food))
    }

    override fun getItemCount() = foods.size

    override fun onBindViewHolder(holder: FoodAdapter.ViewHolder, position: Int) {
        holder.bind(foods[position])
    }

    fun updateFoods(foods: List<Food>) {
        this.foods.clear()
        this.foods.addAll(foods)
        notifyDataSetChanged()
    }

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

        private lateinit var food: Food

        fun bind(food: Food) {
            this.food = food
            val context = itemView.context
            itemView.foodImage.setImageResource(
                context.resources.getIdentifier(food.thumbnail, null, context.packageName)
            )
        }
    }

}
```

CreatureActivity.kt
```kt
class CreatureActivity : AppCompatActivity() {

  private lateinit var creature: Creature
  private val adapter = FoodAdapter(mutableListOf())

  companion object {
    private const val EXTRA_CREATURE_ID = "EXTRA_CREATURE_ID"

    fun newIntent(context: Context, creatureId: Int): Intent {
      val intent = Intent(context, CreatureActivity::class.java)
      intent.putExtra(EXTRA_CREATURE_ID, creatureId)
      return intent
    }
  }

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_creature)

    setupCreature()
    setupTitle()
    setupViews()
    setupFavoriteButton()
    setupFoods()
  }

  // 中略

  private fun setupFoods(){
    foodRecyclerView.layoutManager = LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false)
    foodRecyclerView.adapter = adapter
    val foods = CreatureStore.getCreatureFoods(creature)
    adapter.updateFoods(foods)
  }

}

```

Food.kt
```kt
data class Food(val id: Int, val name: String, val image: String) {
    val thumbnail: String
        get() = "drawable/thumbnail_$image"
}
```
