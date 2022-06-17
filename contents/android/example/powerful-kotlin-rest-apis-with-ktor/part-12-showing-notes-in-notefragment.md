# Showing notes in NoteFragment

이번엔 `NoteFragment`에 `RecyclerView`를 구현해 Note들이 보여지도록 구현해보자.

우선 `adapters` 패키지를 생성한 후 하위에 `NoteAdapter`를 생성 및 작성해준다.

```kotlin
class NoteAdapter : RecyclerView.Adapter<NoteAdapter.NoteViewHolder>() {

    inner class NoteViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {

    }

    private val diffCallback = object : DiffUtil.ItemCallback<Note>() {
        override fun areItemsTheSame(oldItem: Note, newItem: Note): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: Note, newItem: Note): Boolean {
            return oldItem.hashCode() == newItem.hashCode()
        }
    }

    private var onItemClickListener: ((Note) -> Unit)? = null

    private val differ = AsyncListDiffer(this, diffCallback)

    var notes: List<Note>
        get() = differ.currentList
        set(value) = differ.submitList(value)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): NoteViewHolder {
        return NoteViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.item_note, parent, false))
    }

    override fun onBindViewHolder(holder: NoteViewHolder, position: Int) {
        val note = notes[position]
        holder.itemView.apply {
            tvTitle.text = note.title
            if (!note.isSynced) {
                ivSynced.setImageResource(R.drawable.ic_cross)
                tvSynced.text = "Not Synced"
            } else {
                ivSynced.setImageResource(R.drawable.ic_check)
                tvSynced.text = "Synced"
            }

            val dateFormat = SimpleDateFormat("dd.MM.yy, HH:mm", Locale.getDefault())
            val dateString = dateFormat.format(note.date)
            tvDate.text = dateString

            val drawable = ResourcesCompat.getDrawable(resources, R.drawable.circle_shape, null)
            drawable?.let {
                val wrappedDrawable = DrawableCompat.wrap(it)
                val color = Color.parseColor("#${note.color}")
                DrawableCompat.setTint(wrappedDrawable, color)
                viewNoteColor.background = wrappedDrawable
            }

            setOnClickListener {
                onItemClickListener?.let { click ->
                    click(note)
                }
            }
        }
    }

    override fun getItemCount(): Int {
        return notes.size
    }

    fun setOnItemClickListener(onItemClick: (Note) -> Unit) {
        this.onItemClickListener = onItemClick
    }
}
```

### NotesFragment

```kotlin
@AndroidEntryPoint
class NotesFragment : BaseFragment(R.layout.fragment_notes) {

    // ...

    private lateinit var noteAdapter: NoteAdapter

    // ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        setupRecyclerView()
        // ...
    }

    private fun setupRecyclerView() = rvNotes.apply {
        noteAdapter = NoteAdapter()
        adapter = noteAdapter
        layoutManager = LinearLayoutManager(requireContext())
    }
    // ...
}
```

`NotesViewModel`를 다음과 같이 구현해준다.

```kotlin
class NotesViewModel @ViewModelInject constructor(
    private val repository: NoteRepository
) : ViewModel() {

    private val _forceUpdate = MutableLiveData<Boolean>(false)

    private val _allNotes = _forceUpdate.switchMap {
        // switchmap : whenever a value is post to _forceUpdate LiveData then we'll emit whatever comes in block here
        repository.getAllNotes().asLiveData(viewModelScope.coroutineContext)
    }.switchMap {
        MutableLiveData(Event(it))
    }
    val allNotes: LiveData<Event<Resource<List<Note>>>> = _allNotes

    fun syncAllNotes() = _forceUpdate.postValue(true)
}
```

그리고 다시 `NotesFragment`에서 Note 아이템 클릭 이벤트와 `allNotes`에 대한 observer를 구현해준다.

```kotlin
@AndroidEntryPoint
class NotesFragment : BaseFragment(R.layout.fragment_notes) {

    private val viewModel: NotesViewModel by viewModels()

    // ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // AuthFragment에서 화면 회전을 비활성화 했지만, 다시 활성화 시켜준다.
        requireActivity().requestedOrientation = SCREEN_ORIENTATION_USER
        setupRecyclerView()
        subscribeToObservers()

        noteAdapter.setOnItemClickListener {
            findNavController().navigate(
                NotesFragmentDirections.actionNotesFragmentToNoteDetailFragment(it.id)
            )
        }
        fabAddNote.setOnClickListener {
            findNavController().navigate(
                NotesFragmentDirections.actionNotesFragmentToAddEditNoteFragment(
                    ""
                )
            )
        }
    }

    private fun subscribeToObservers() {
        viewModel.allNotes.observe(viewLifecycleOwner, Observer {
            it?.let { event ->
                // peekContent
                val result = event.peekContent()
                when (result.status) {
                    Status.SUCCESS -> {
                        noteAdapter.notes = result.data!!
                        swipeRefreshLayout.isRefreshing = false
                    }
                    Status.ERROR -> {
                        // one time event
                        event.getContentIfNotHandled()?.let { errorResource ->
                            errorResource.message?.let { message ->
                                showSnackbar(message)
                            }
                        }
                        result.data?.let { notes ->
                            noteAdapter.notes = notes
                        }
                        swipeRefreshLayout.isRefreshing = false
                    }
                    Status.LOADING -> {
                        result.data?.let {
                            noteAdapter.notes = it
                        }
                        swipeRefreshLayout.isRefreshing = true
                    }
                }
            }
        })
    }

    private fun setupRecyclerView() = rvNotes.apply {
        noteAdapter = NoteAdapter()
        adapter = noteAdapter
        layoutManager = LinearLayoutManager(requireContext())
    }

    // ...
}
```