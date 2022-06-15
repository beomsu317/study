# Test Swipe to Delete Functionality

`ShoppingItemAdapter`를 생성한다.

```kotlin
class ShoppingItemAdapter @Inject constructor(
    private val glide: RequestManager
) : RecyclerView.Adapter<ShoppingItemAdapter.ShoppingItemViewHolder>() {

    class ShoppingItemViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)

    private val diffCallback = object : DiffUtil.ItemCallback<ShoppingItem>() {
        override fun areItemsTheSame(oldItem: ShoppingItem, newItem: ShoppingItem): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: ShoppingItem, newItem: ShoppingItem): Boolean {
            return oldItem.hashCode() == newItem.hashCode()
        }
    }

    private val differ = AsyncListDiffer(this, diffCallback)

    var shoppingItems: List<ShoppingItem>
        get() = differ.currentList
        set(value) = differ.submitList(value)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ShoppingItemViewHolder {
        return ShoppingItemViewHolder(
            LayoutInflater.from(parent.context).inflate(
                R.layout.item_shopping,
                parent,
                false
            )
        )
    }

    override fun getItemCount(): Int {
        return shoppingItems.size
    }

    override fun onBindViewHolder(holder: ShoppingItemViewHolder, position: Int) {
        val shoppingItem = shoppingItems[position]
        holder.itemView.apply {
            glide.load(shoppingItem.imageUrl).into(ivShoppingImage)

            tvName.text = shoppingItem.name
            val amountText = "${shoppingItem.amount}x"
            tvShoppingItemAmount.text = amountText
            val priceText = "${shoppingItem.price}€"
            tvShoppingItemPrice.text = priceText
        }
    }
}
```

`ShoppingFragment`에서 아이템이 스와이프되면 삭제되도록 구현한다.

```kotlin
class ShoppingFragment @Inject constructor(
    val shoppingItemAdapter: ShoppingItemAdapter,
    var viewModel: ShoppingViewModel ?= null
): Fragment(R.layout.fragment_shopping) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
				// viewModel이 null이 아닌 경우 test이므로 그 viewModel을 사용한다.
        viewModel = viewModel ?: ViewModelProvider(requireActivity()).get(ShoppingViewModel::class.java)
        subscribeToObservers()
        setupRecyclerView()

        fabAddShoppingItem.setOnClickListener {
            findNavController().navigate(
                ShoppingFragmentDirections.actionShoppingFragmentToAddShoppingItemFragment()
            )
        }
    }

    private val itemTouchCallback = object: ItemTouchHelper.SimpleCallback(
        0, LEFT or RIGHT
    ) {
        override fun onMove(
            recyclerView: RecyclerView,
            viewHolder: RecyclerView.ViewHolder,
            target: RecyclerView.ViewHolder
        ): Boolean = true

        override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
            val pos = viewHolder.layoutPosition
            val item = shoppingItemAdapter.shoppingItems[pos]
            viewModel?.deleteShoppingItem(item)
            Snackbar.make(requireView(), "Successfully deleted item", Snackbar.LENGTH_LONG).apply {
                setAction("Undo") {
                    viewModel?.insertShoppingIntoDb(item)
                }
                show()
            }
        }
    }

    private fun subscribeToObservers() {
        viewModel?.shoppingItems?.observe(viewLifecycleOwner, Observer {
            shoppingItemAdapter.shoppingItems = it
        })
        viewModel?.totalPrice?.observe(viewLifecycleOwner, Observer {
            val price = it ?: 0f
            val priceText = "Total Price: $price$"
            tvShoppingItemPrice.text = priceText

        })
    }

    private fun setupRecyclerView() {
        rvShoppingItems.apply {
            adapter = shoppingItemAdapter
            layoutManager = LinearLayoutManager(requireContext())
            ItemTouchHelper(itemTouchCallback).attachToRecyclerView(this)
        }
    }
}
```

`TestShoppingFragmentFactory`를 생성해 테스트에 사용될 `FragmentFactory`를 만든다.

```kotlin
class TestShoppingFragmentFactory @Inject constructor(
    private val imageAdapter: ImageAdapter,
    private val glide: RequestManager,
    private val shoppingItemAdapter: ShoppingItemAdapter
): FragmentFactory() {

    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        return when(className) {
            ImagePickFragment::class.java.name -> ImagePickFragment(imageAdapter)
            AddShoppingItemFragment::class.java.name -> AddShoppingItemFragment(glide)
            ShoppingFragment::class.java.name -> ShoppingFragment(
                shoppingItemAdapter,
                ShoppingViewModel(FakeShoppingRepositoryAndroidTest())
            )
            else -> super.instantiate(classLoader, className)
        }
    }
}
```

`ShoppingFragmentTest`에 `swipeShoppingItem_deleteItemInDb` 함수에 swipe 테스트 케이스를 만든다.

```kotlin
@MediumTest
@HiltAndroidTest
@ExperimentalCoroutinesApi
class ShoppingFragmentTest {

    @get:Rule
    var hiltRule = HiltAndroidRule(this)

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()

    @Inject
    lateinit var testFragmentFactory: TestShoppingFragmentFactory

    @Before
    fun setup() {
        hiltRule.inject()
    }

		...
    @Test
    fun swipeShoppingItem_deleteItemInDb() {
        val shoppingItem = ShoppingItem("TEST", 1, 1f, "TEST", 1)
        var testViewModel: ShoppingViewModel? = null
        launchFragmentInHiltContainer<ShoppingFragment>(
            fragmentFactory = testFragmentFactory
        ) {
            testViewModel = viewModel
            viewModel?.insertShoppingIntoDb(shoppingItem)
        }

        onView(withId(R.id.rvShoppingItems)).perform(
            RecyclerViewActions.actionOnItemAtPosition<ShoppingItemAdapter.ShoppingItemViewHolder>(
                0,
                swipeLeft()
            )
        )

        assertThat(testViewModel?.shoppingItems?.getOrAwaitValue()).isEmpty()
    }
}
```

## References

[Test Swipe to Delete Functionality - Testing on Android - Part 16](https://www.youtube.com/watch?v=V5S2-Mg8B2U&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=16)