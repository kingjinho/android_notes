# How Recyclerview Works Internally

### Key Terms & Components

- RecyclerView
    - A ViewGroup for displaying a large data set
- LayoutManager
    - `Measuring and positioning item views in RecyclerView`
    - `Determine when to recycle views that are no longer visible`
- Adapter
    - `Responsible for providing views that represent items in a data set`
    - a.k.a `convert model into view`
- Position
    - Position of data item within an Adapter
- Index
    - Index of an attached child view used it a call to `ViewGroup.getChildAt`
    - Contrast with position
- Binding
    - `Process of preparing a child view to display data`
- Recycle(view)
    - `Previously used view in cache for later reuse` to display `same type of data`
    - `Drastically improve performance by skipping initial layout inflation or construction`
- Scrap(view)
    - `Child view in temporary detached state during layout`
    - `Maybe reused` without fully detached from parent recyclerview, either
      `unmodified` if no binding is required, or `modified` by the adapter if the view was considered dirty
- Dirty(view)
    - `View that must be rebound by the adapter`
- ViewHolder
    - Describes an item view and metadata about its place within the RecyclerView
- AdapterHelper
    - `Helper class that can enqueue and process adapter update operations`
    - Contains callback
        - Contract between AdapterHelper and RecyclerView
- ChildrenHelper
    - Helper class to manage children
    - Wraps a RecyclerView and adds ability to hide some children

### Basic usage of RecyclerView

```kotlin
class RecyclerViewAdapter(callback: DiffUtil.ItemCallback<ImageData>) :
    ListAdapter<ImageData, ItemViewHolder>(callback) {


    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemViewHolder {
        ...
    }

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        ...
    }
}

class ItemViewHolder(private val binding: ItemLayoutBinding) :
    RecyclerView.ViewHolder(binding.root) {

    fun bind(item: ImageData) {
        ...
    }
}

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    private val listAdapter by lazy {
        RecyclerViewAdapter(object : DiffUtil.ItemCallback<ImageData>() {
            override fun areItemsTheSame(oldItem: ImageData, newItem: ImageData): Boolean {
                return oldItem == newItem
            }

            override fun areContentsTheSame(oldItem: ImageData, newItem: ImageData): Boolean {
                return oldItem.id == newItem.id
            }
        })
    }

    private val layoutManager by lazy {
        LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false)
    }
    ...


    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        binding.list.also {
            it.adapter = listAdapter
            it.layoutManager = layoutManager
            it.addItemDecoration(object : RecyclerView.ItemDecoration() {
                override fun getItemOffsets(
                    outRect: Rect,
                    view: View,
                    parent: RecyclerView,
                    state: RecyclerView.State
                ) {
                    outRect.bottom += 32
                }
            })
        }
        listAdapter.submitList(data)
    }
}
```

### Things to know

- `RecyclerView Adapter uses Observer Pattern to notify item(item range) changed, moved, removed`
- Helper Pattern

### Flow

#### setAdapter(adapter: Adapter)

- function
    - Set new or null adapter to provide child views
- When adapter is changed, all existing views are recycled back to the pool
    - if the pool has only one adapter, it will be cleared

```java
public void setAdapter(@Nullable Adapter adapter){
        // bail out if layout is frozen

        /**
         * enable or disable layout and scroll
         * true to freeze, false to re-enable
         */
        setLayoutFrozen(false);

        /**
         * 1. replacement of adapter with new
         * 2. remove observer from old
         * 3. set new adapter
         * 4. register observer
         * 5. attach to recyclerview
         * 6. and callbacks
         *
         * Params:
         * adapter – The new adapter
         * compatibleWithPrevious – If true, the new adapter is using the same View Holders and
         *                          item types with the current adapter (helps us avoid cache invalidation).
         * removeAndRecycleViews – If true, we'll remove and recycle all existing views.
         *                          If compatibleWithPrevious is false, this parameter is ignored.
         */
        setAdapterInternal(adapter,false,true);

        /**
         * set flag value
         * add flag into holder(if exists)
         */
        processDataSetCompletelyChanged(false);
        requestLayout();
        }
```    

- ListAdapter
    - `Adapter including computing diffs between lists on background`
    - Wrapper class for `AsyncListdiffer`
    -

```java
public abstract class ListAdapter<T, VH extends RecyclerView.ViewHolder>
        extends RecyclerView.Adapter<VH> {
    final AsyncListDiffer<T> mDiffer;
    private final AsyncListDiffer.ListListener<T> mListener =
            new AsyncListDiffer.ListListener<T>() {
                @Override
                public void onCurrentListChanged(
                        @NonNull List<T> previousList, @NonNull List<T> currentList) {
                    ListAdapter.this.onCurrentListChanged(previousList, currentList);
                }
            };

    @SuppressWarnings("unused")
    protected ListAdapter(@NonNull DiffUtil.ItemCallback<T> diffCallback) {
        mDiffer = new AsyncListDiffer<>(new AdapterListUpdateCallback(this),
                new AsyncDifferConfig.Builder<>(diffCallback).build());
        mDiffer.addListListener(mListener);
    }

    @SuppressWarnings("unused")
    protected ListAdapter(@NonNull AsyncDifferConfig<T> config) {
        mDiffer = new AsyncListDiffer<>(new AdapterListUpdateCallback(this), config);
        mDiffer.addListListener(mListener);
    }

    ...
}

public class AsyncListDiffer<T> {
    /**
     * Implementation: AdapterListUpdateCallback
     * Called when updated, inserted, removed and moved
     */
    private final ListUpdateCallback mUpdateCallback;

    /**
     * contains diff callback(areItemsSame, Contents same)
     * Config object for ListAdapter, AsyncListDiffer, similar background-thread list diffing adapter
     * logic
     */
    @SuppressWarnings("WeakerAccess") /* synthetic access */
    final AsyncDifferConfig<T> mConfig;
    Executor mMainThreadExecutor;
  
  ...

}  
```

#### setLayoutManager(LayoutManager layout)

- Function
    - Controls custom layout arrangements provided by client code
    - Must be provided for RecyclerView to function

```java
public void setLayoutManager(LayoutManager layout){

        /**
         * 1. check whether it is same as current layout
         * 2. if current is not null,
         *                          Stop animation(if exists)
         *                          Stop scroll
         *                          remove and recycler all views
         *                          remove and recycle scrap Int
         * 3. set recyclerview to (throws an exception if already set)
         * 4.
         */
        ...

        }
```

#### addItemDecoration(ItemDecoration decor)

```java
public class RecyclerView extends ViewGroup implements ScrollingView,
        NestedScrollingChild2, NestedScrollingChild3 {
    ...
    final ArrayList<ItemDecoration> mItemDecorations
            = new ArrayList<>();
    ...
}

```

- Function
    - `Affects both measurement and drawing of individual item views`
- Things to know
    - It is an ordered list
        - Previously added item will occur first

```java
public void addItemDecoration(@NonNull ItemDecoration decor){
        addItemDecoration(decor,-1);
        }

public void addItemDecoration(@NonNull ItemDecoration decor,int index){

        /**
         * check if layoutmanager is in layout or scroll(throws an exception if set)
         * add to decoration to mItemDecorations
         */
        }
```  

#### submitList(item) in AsyncListDiffer class

1. Initial

- Check if newList is equal to currentList
- Check if newList is null :point_right: remove
- Check if currentList is null :point_right: first insert

```java
public void submitList(item,callback){
        // fast simple first insert 
        if(mList==null){
        mList=newList;
        mReadOnlyList=Collections.unmodifiableList(newList);
        /**
         * notify last, after list is updated
         * 1. mUpdateCallback = AdapterListUpdateCallback and this comes from when 
         *    initialzing ListAdapter
         * 2. mUpdateCallback.onInserted calls RecyclerView.Adapter's 
         *    notifyItemRangeInserted
         * 3. This calls AdapterDataObservable's onItemRangeInserted
         *    This uses observer pattern
         * 4. This calls RecyclerViewDataObserver's onItemRangeInserted
         *    Check if layout is in computing or scrolling
         * 5. Calls AdapterHelper's onItemRangeInserted
         *    Check item count
         *    add to pending update list
         * 6. Trigger Update Processor
         *    requestLayout()
         * 7. onCurrentListChanged
         *      calls mListener from ListAdapter and this call's ListAdapter's onCurrentListChanged
         * 8.      
         */
        mUpdateCallback.onInserted(0,newList.size());
        onCurrentListChanged(previousList,commitCallback);
        return;
        }
        ...
}
```
  