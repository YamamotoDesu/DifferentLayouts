# DifferentLayouts

## Create a Horizontal RecyclerView
<img width="300" src="https://user-images.githubusercontent.com/47273077/191889762-fc49defc-75c3-46fb-a47e-e94a02592f58.png">

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

------

## Nest a RecyclerView
<img width="300" src="https://user-images.githubusercontent.com/47273077/191893708-87943d45-0755-4814-8383-eef85fcc5fe0.png">

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="@dimen/list_item_creature_height">

    <ImageView
        android:id="@+id/creatureImage"
        android:background="@color/colorAccent"
        android:layout_width="@dimen/list_item_creature_height"
        android:layout_height="@dimen/list_item_creature_height"
        android:contentDescription="@string/content_description_creature_image"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@drawable/creature_cat_derp" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/foodRecyclerView"
        android:layout_width="0dp"
        android:layout_height="@dimen/list_item_creature_height"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/creatureImage"
        app:layout_constraintTop_toTopOf="parent"
        />


</androidx.constraintlayout.widget.ConstraintLayout>
```

CreatureWithFoodAdapter
```kt
class CreatureWithFoodAdapter(private val creatures: MutableList<Creature>): RecyclerView.Adapter<CreatureWithFoodAdapter.ViewHolder>() {

    private val viewPool = RecyclerView.RecycledViewPool()

    class ViewHolder(itemView: View) : View.OnClickListener, RecyclerView.ViewHolder(itemView) {
        private lateinit var creature: Creature
        private val adapter = FoodAdapter(mutableListOf())

        init {
            itemView.setOnClickListener(this)
        }

        fun bind(creature: Creature) {
            this.creature = creature
            val context = itemView.context
            itemView.creatureImage.setImageResource(
                    context.resources.getIdentifier(creature.uri, null, context.packageName))
            setupFoods()
        }

        override fun onClick(view: View?) {
            view?.let {
                val context = it.context
                val intent = CreatureActivity.newIntent(context, creature.id)
                context.startActivity(intent)
            }
        }

        private fun setupFoods() {
            itemView.foodRecyclerView.layoutManager =
                LinearLayoutManager(itemView.context, LinearLayoutManager.HORIZONTAL, false)
            itemView.foodRecyclerView.adapter = adapter

            val foods = CreatureStore.getCreatureFoods(creature)
            adapter.updateFoods(foods)
        }

    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
//        return ViewHolder(parent.inflate(R.layout.list_item_creature_with_food))
        val holder = ViewHolder(parent.inflate(R.layout.list_item_creature_with_food))
        holder.itemView.foodRecyclerView.setRecycledViewPool(viewPool)
        return holder
    }

    override fun getItemCount() = creatures.size

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(creatures[position])
    }

}
```

### LinearSnapHelperを使う
<img width="300" src="https://user-images.githubusercontent.com/47273077/191894128-0a3d1a44-2f14-403b-b64b-b5fb928b113f.gif">

```kt
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
//        return ViewHolder(parent.inflate(R.layout.list_item_creature_with_food))
        val holder = ViewHolder(parent.inflate(R.layout.list_item_creature_with_food))
        holder.itemView.foodRecyclerView.setRecycledViewPool(viewPool)
        LinearSnapHelper().attachToRecyclerView(holder.itemView.foodRecyclerView)
        return holder
    }
```

### LinearSnapHelperを使わない場合
<img width="300" src="https://user-images.githubusercontent.com/47273077/191893914-14afb72a-1daa-49f7-b1b5-7e589cd7e999.gif">


----

## User a GridLayoutManager
<img width="300" alt="スクリーンショット 2022-09-23 15 44 47" src="https://user-images.githubusercontent.com/47273077/191905450-46aae19f-1239-4688-986e-cfa69ee48a99.png">

list_item_creature_card.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/creatureCard"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="@dimen/padding_half"
    card_view:cardCornerRadius="@dimen/card_corner_radius"
    card_view:cardElevation="@dimen/card_elevation">

    <ImageView
        android:id="@+id/creatureImage"
        android:layout_width="@dimen/card_image_size"
        android:layout_height="@dimen/card_image_size"
        android:layout_gravity="center"
        android:contentDescription="@string/content_description_creature_image"
        android:scaleType="fitXY"
        tools:srcCompat="@drawable/creature_cat_derp" />

    <LinearLayout
        android:id="@+id/cardRipple"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?android:selectableItemBackground"
        android:orientation="horizontal" />

    <LinearLayout
        android:id="@+id/nicknameHolder"
        android:layout_width="match_parent"
        android:layout_height="@dimen/card_nickname_holder_height"
        android:layout_gravity="bottom"
        android:alpha="0.9"
        android:background="@color/colorAccent"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/cardNickname"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:gravity="center_horizontal"
            android:paddingStart="@dimen/padding_half"
            android:paddingEnd="@dimen/padding_half"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:textColor="@android:color/white"
            android:textStyle="bold" />

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

CreatureCardAdapter
```kt
class CreatureCardAdapter(private val creatures: MutableList<Creature>): RecyclerView.Adapter<CreatureCardAdapter.ViewHolder>() {

    class ViewHolder(itemView: View) : View.OnClickListener, RecyclerView.ViewHolder(itemView) {
        private lateinit var creature: Creature

        init {
            itemView.setOnClickListener(this)
        }

        fun bind(creature: Creature) {
            this.creature = creature
            val context = itemView.context
            val imageResource = context.resources.getIdentifier(creature.uri, null, context.packageName)
            itemView.creatureImage.setImageResource(imageResource)
            itemView.cardNickname.text = creature.nickname
            setBackgroundColors(context, imageResource)
        }

        override fun onClick(view: View?) {
            view?.let {
                val context = it.context
                val intent = CreatureActivity.newIntent(context, creature.id)
                context.startActivity(intent)
            }
        }

        private fun setBackgroundColors(context: Context, imageResource: Int) {
            val image = BitmapFactory.decodeResource(context.resources, imageResource)
            Palette.from(image).generate { palette ->
                palette?.let {
                    val backgroundColor = it.getDominantColor(ContextCompat.getColor(context, R.color.colorPrimaryDark))
                    itemView.creatureCard.setBackgroundColor(backgroundColor)
                    itemView.nicknameHolder.setBackgroundColor(backgroundColor)
                    val textColor = if (isColorDark(backgroundColor)) Color.WHITE else Color.BLACK
                    itemView.cardNickname.setTextColor(textColor)
                }
            }
        }

        private fun isColorDark(color: Int) : Boolean {
            val darkness = 1 - (0.299 * Color.red(color) +
                    0.587 * Color.green(color) +
                    0.114 * Color.blue(color) / 255)

            return  darkness >= 0.5
        }

    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(parent.inflate(R.layout.list_item_creature_card))
    }

    override fun getItemCount() = creatures.size

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(creatures[position])
    }

}
```



