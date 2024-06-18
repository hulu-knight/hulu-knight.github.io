---
layout: post
title: android RecyclerView刷新机制详解
tags: android
author: hulu-knight
date: 2024-06-18 18:33 +0800


---





# RecyclerView刷新机制详解

RecyclerView是我们常用的列表控件，一般来说当Item的数据改变的时候我们需要刷新当前的Item 。

如何刷新 RV 的列表？基本上有这几种方式：

> notifyDataSetChanged
>
> notifyItemChanged(int position)
>
> notifyItemChanged(int position, @Nullable Object payload)

一般来说一个 item 是由多个控件组成的，如 Button、CheckBox、TextView、ImageView、ViewGroup 等组合。当我们点击item的某个控件时，RV 需要重新计算布局、刷新视图来响应交互。假设一个 item 包含了N多个控件，如果调用notifyItemChanged(int position) 时，item 中的每个控件都需要重新布局显示，无形中加大了内存和性能的损耗。

就算我们不考虑内存和性能的问题，那么一些效果也是我们无法接受的，当 item 刷新的时候会导致内部的图片或item 出现一次闪烁。

以我的精神粮食起点为例,如下

![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/uPic/4d694872df6545f5b2d6279a4debfadb.webp)

所以我们才需要用到类似 Payload 与 Diff 之类的刷新方式。

下面我们就一起看看它们是怎么使用的。

## 1.Payload的刷新

我们通过 notifyItemChanged(int position, @Nullable Object payload)来刷新指定索引的item。

在 RV 的 Adapter 中的onBindViewHolder可以接收到 payloads 参数，这个 payloads 参数是一个 List 对象，该对象不是 null 但可能是空的。

```java
public void onBindViewHolder(VH holder, intposition, List<Object> payloads) {
	onBindViewHolder(holder, position);
}
```

通过 Adapter 的 notifyXXX 函数的带有 payload 参数的函数可以设置 payload 对象，如：

```java
public final void notifyItemChanged(intposition, Object payload) {
	mObservable.notifyItemRangeChanged(position, 1, payload);
}
```

由于onBindViewHolder 有重载的方法，如果使用了 payloads 的方式，那么我们需要做兼容，如果没有 payloads 就去走整个 item 的刷新，如果有 payloads 那么我们就根据指定的 payload 去刷新指定的数据。
```java
@Override
public void onBindViewHolder(ViewHolder holder, intposition, List< Object> payloads) {
	if(!payloads.isEmpty && payloads. get( 0).equals( "like")) {
    //如果是我们指定的 payloads ，那么就可以指定刷新
    TipsBean item = mData. get(position);
    holder.tvLikeNum.setText(R.id.tv_tips_like_num, item.likes_count > 0? item.likes_count + "": "Like");
    holder.tvLikeNum.setTextColor(R.id.tv_tips_like_num, item.likes_count > 0? CommUtils.getColor(R.color.black) : CommUtils.getColor(R.color.home_item_text_light_gray));
	} else{
		// 如果没有 payloads ，或者不是我们指定的，还是返回默认的整个刷新
		super.onBindViewHolder(holder, position);
	}
}
```

如果没有 payload ，当调用notifyItemChanged 时，RV 会通过回调 onBindViewHolder(holder, position) 来更新当前数据变化的 item ，此时会触发 整个 item 中 view 的重新布局和计算位置，这样的话只要是其中一个 View 状态变化了，最终会导致整个 item 都需要重新布局一遍。

例如上述的例子，在评论的列表中，我只点赞了第一个 item ，我们就通过 payload 来告诉 RV 这个 item 中的 like 文本变化了，那么我们就只需要处理 Like 文本的变化。

## 2. Diff的刷新与快速实现方法

每一个 payloads 都要写一次，然后我们在调用notifyItemChanged 时万一写错了怎么办？有没有一种方式能让程序自动管理，让程序帮我们记录 payloads ？最好还能帮助我们自动排序！

有，我们先看看自动排序的方式： **SortedList 的方式。**

SortedList，顾名思义就是排序列表，它适用于列表有序且不重复的场景。并且SortedList会帮助你比较数据的差异，定向刷新数据。而不是简单粗暴的notifyDataSetChanged。

例如我们定义一个城市排序的对象：

```java
public class City {
  private int id;
  private String cityName;
  private String firstLetter;
  ...
}
```

我们需要进行排序的规则定义。

```java
public class SortedListCallback extends SortedListAdapterCallback<City> {
	public SortedListCallback(RecyclerView.Adapter adapter){
		super(adapter);
	}

  /**
  * 排序条件
  */
  @Override
  public int compare(City o1, City o2){
    return o1.getFirstLetter.compareTo(o2.getFirstLetter);
  }

  /**
  * 用来判断两个对象是否是相同的Item。
  */
  @Override
  public boolean areItemsTheSame(City item1, City item2){
    returnitem1.getId == item2.getId;
  }

  /**
  * 用来判断两个对象是否是内容的Item。
  */
  @Override
  public boolean areContentsTheSame(City oldItem, City newItem){
    if(oldItem.getId != newItem.getId) {
      returnfalse;
    }
    return oldItem.getCityName.equals(newItem.getCityName);
	}
	
}
```

再然后在Adapte中使用的时候，不需要Arrylist，要用排序的集合，新的对象SortedList排序集合。

```java
public class SortedAdapter extends RecyclerView.Adapter< SortedAdapter.ViewHolder> {

  // 数据源使用SortedList
  private SortedList<City> mSortedList;

  private LayoutInflater mInflater;

  public SortedAdapter(Context mContext){
    mInflater = LayoutInflater.from(mContext);
  }

  public void setSortedList(SortedList<City> mSortedList){
    this.mSortedList = mSortedList;
  }
  /**
  * 批量更新操作，例如：
  * <pre>
  * mSortedList.beginBatchedUpdates;
  * try {
  * 	mSortedList.add(item1)
  * 	mSortedList.add(item2)
  * 	mSortedList.remove(item3)
  * 	...
  * } finally {
  * 	mSortedList.endBatchedUpdates;
  * }
  * </pre>
  */
	public void addData(List<City> mData){
    mSortedList.beginBatchedUpdates;
    mSortedList.addAll(mData);
    mSortedList.endBatchedUpdates;
  }

  /**
  * 移除item
  */
  public void removeData(int index) {
  	mSortedList.removeItemAt(index);
  }

  /**
  * 清除集合
  */
  public void clear{
  	mSortedList.clear;
  }

  @Override
  @NonNull
  public SortedAdapter.ViewHolder onCreateViewHolder(@NonNull  ViewGroup parent, int viewType) {
  return new ViewHolder(mInflater.inflate(R.layout.item_test, parent, false));
  }

  @Override
  public void onBindViewHolder(@NonNull SortedAdapter.ViewHolder holder, finalintposition) {
  // 。。。
  }

  @Override
  public int getItemCount{
  	return mSortedList.size;
  }

  public class ViewHolder extends RecyclerView.ViewHolder{
    public ViewHolder(View itemView){
      super(itemView);
    }
  }

}
```

使用的时候：

```java
RecyclerView mRecyclerView = findViewById(R.id.rv);
mRecyclerView.setLayoutManager(newLinearLayoutManager(this));
mSortedAdapter = newSortedAdapter(this);

// SortedList初始化
SortedListCallback mSortedListCallback = newSortedListCallback(mSortedAdapter);
SortedList mSortedList = newSortedList<>(City.class, mSortedListCallback);
mSortedAdapter.setSortedList(mSortedList);
mRecyclerView.setAdapter(mSortedAdapter);

//添加数据
...
```

这样确实就能实现自动排序，刷新列表了，相对 notifyDataSetChanged的暴力刷新，优雅一点。但是它没有 payload 的功能，这个刷新只是刷新的是整个 RV 中的部分Item，但还是刷新整个 item 啊。

有没有办法既能排序又能 payload差异化刷新的方式呢？

肯定有哇， DiffUtil 的方式就此诞生，常用的相关的几个类为 DiffUtil , AsyncListDiffer, ListAdapter（用于快速实现的封装类）

**DiffUtil** 能实现排序加payload局部刷新的功能:

- 当某个 item 的位置变化，触发排序逻辑，有移除和添加的动画（可去掉）。
- 当某个 item 的位置不变，内容变化，触发 payload 局部刷新。
- 在子线程中计算DiffResult，在主线程中刷新RecyclerView。

**AsyncListDiffer** 又是什么东西，为什么需要它。

其实AsyncListDiffer 就是集成了 AsyncListUtil + DiffUtil 的功能，由于 DiffUtil在计算数据差异DiffUtil.calculateDiff(mDiffCallback) 是一个 **耗时操作**，需要我们放到子线程去处理，最后在主线程刷新，为了我们开发者更加的方便，谷歌直接提供了AsyncListDiffer方便我们直接使用。看来谷歌是怕我们开发者不会使用子线程，直接给我们写好了。

**ListAdapter** 又是个什么鬼？怎么越来越复杂了？

其实谷歌就喜欢把简单的东西复杂化，如果我们使用AsyncListDiffer 去实现的话，虽然不用我们操心子线程了，但是还是需要我们定义对象、集合、添加数据的方法 ，如addNewData ,里面调用mDiffer.submitList 才能实现。

谷歌还是怕我们不会用吧！直接把AsyncListDiffer 的使用都给简化了，直接提供了 ListAdapter 包装类，内部对AsyncListDiffer 的使用做了一系列的封装，使用的时候我们的 RV-Adapter 直接继承 ListAdapter 即可实现AsyncListDiffer的功能，内部连设置数据的方法都给我们提供好了。谷歌真的是我们的好爸爸！为我们开发者操碎了心。

那它们都到底 **怎么使用**呢？

不管什么方式，我们都需要定义好自己的DiffUtil.CallBack，毕竟就算让程序帮我们排序和差分，我们也得告诉程序排序的规则和diff的规则，是吧！

### DiffUtil

```java
public class MyDiffUtilItemCallback extends DiffUtil. ItemCallback< TestBean> {
  /**
  * 是否是同一个对象
  */
  @Override
  public boolean areItemsTheSame( @NonNullTestBean oldItem, @NonNullTestBean newItem) {
    returnoldItem.getId == newItem.getId;
  }

  /**
  * 是否是相同内容
  */
  @Override
  public boolean areContentsTheSame( @NonNullTestBean oldItem, @NonNullTestBean newItem) {
    returnoldItem.getName.equals(newItem.getName);
  }

  /**
  * areItemsTheSame返回true而areContentsTheSame返回false时调用,也就是说两个对象代表的数据是一条，
  * 但是内容更新了。此方法为定向刷新使用，可选。
  */
  @Nullable
  @Override
  public Object getChangePayload( @NonNullTestBean oldItem, @NonNullTestBean newItem) {
    Bundle payload = new Bundle;
    if(!oldItem.getName.equals(newItem.getName)) {
      payload.putString( "KEY_NAME", newItem.getName);
    }
    if(payload.size == 0){
    //如果没有变化 就传空
      returnnull;
    }
    return payload;
  }

}
```

那个Diff的具体实现我们选用哪一种方案呢？其实三种方式都是可以实现的，这里我们先使用AsyncListDiffer的方式来实现。

上面关于AsyncListDiffer的介绍我们说过了，虽然不需要我们实现异步操作了，但是我们还是需要实现对象、集合、添加数据的方法等。

示例如下：

```java
public class AsyncListDifferAdapter extends RecyclerView.Adapter< AsyncListDifferAdapter.ViewHolder> {

  private LayoutInflater mInflater;
  // 数据的操作由AsyncListDiffer实现
  private AsyncListDiffer<TestBean> mDiffer;
  public AsyncListDifferAdapter(Context mContext){
    // 初始化AsyncListDiffe
    mDiffer = new AsyncListDiffer<>(this, newMyDiffUtilItemCallback);
    mInflater = LayoutInflater.from(mContext);
  }

  public void addData(TestBean mData){
  	//添加数据传对象和对象集合都可以
    List<TestBean> mList = new ArrayList<>;
    mList.addAll(mDiffer.getCurrentList);
    mList.add(mData);
    mDiffer.submitList(mList);
  }

  public void addData(List<TestBean> mData){
    // 由于DiffUtil是对比新旧数据，所以需要创建新的集合来存放新数据。
    // 实际情况下，每次都是重新获取的新数据，所以无需这步。
    List<TestBean> mList = new ArrayList<>;
    mList.addAll(mData);
    mDiffer.submitList(mList);
  }

  //删除数据，要先获取全部集合，再删除指定的集合，再提交删除之后的集合
  public void removeData(intindex) {
    List<TestBean> mList = new ArrayList<>;
    mList.addAll(mDiffer.getCurrentList);
    mList.remove(index);
    mDiffer.submitList(mList);
  }

  publicvoidclear{
  	mDiffer.submitList( null);
  }

  @Override
  @NonNull
  public AsyncListDifferAdapter.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, intviewType) {
  	return new ViewHolder(mInflater.inflate(R.layout.item_test, parent, false));
  }

  @Override
  public void onBindViewHolder(@NonNull ViewHolder holder, intposition, @NonNull List<Object> payloads) {
    if(payloads.isEmpty) {
      onBindViewHolder(holder, position);
    } else{
      Bundle bundle = (Bundle) payloads.get(0);
      holder.mTvName.setText(bundle.getString("KEY_NAME"));
    }
  }

  @Override
  public void onBindViewHolder(@NonNull AsyncListDifferAdapter.ViewHolder holder, final int position) {
    TestBean bean = mDiffer.getCurrentList.get(position);
    holder.mTvName.setText(bean.getName);
  }

  @Override
  public int getItemCount{
  	return mDiffer.getCurrentList.size;
  }

  static class ViewHolder extends RecyclerView. ViewHolder{
  	......
  }

}
```

由于Diff.Callback我们已经在 Adapter 内部已经初始化了，所以使用的时候我们直接像普通的 RV 设置 Adapter 一样即可。

在更新数据的时候我们使用 Adapter 定义的addData和removeData 即可完成Diff刷新。

**使用 ListAdapter 会怎样？**

如果觉得使用AsyncListDiffer都嫌弃麻烦的话，我们直接使用ListAdapter 也能实现。

由于我们还是需要一个List集合去保存我们的数据，我们就能对ListAdapter 再做一个简单的基类封装。
```java
public abstract class BaseRVDifferAdapter<T, VH extends RecyclerView.ViewHolder> extends ListAdapter<T, VH> {

  protected Context mContext;
  protected LayoutInflater mInflater;
  protected List<T> mDatas = newArrayList<>;
  
  public BaseRVDifferAdapter(Context context, DiffUtil.ItemCallback<T> callback){
    super(callback);
    mContext = context;
    mInflater = LayoutInflater.from(mContext);
  }

  //设置数据源
  protected void setData(List<T> list){
  	mDatas.clear;
  	mDatas.addAll(list);
		List<T> mList = new ArrayList<>;
		mList.addAll(mDatas);
		submitList(mList);
  }

  //添加数据源
  protected void addData(List<T> list){
    mDatas.addAll(list);
    List<T> mList = new ArrayList<>;
    mList.addAll(mDatas);
    submitList(mList);
  }

  //删除指定索引数据源
  protected void removeData(int index) {
    mDatas.remove(index);
    List<T> mList = new ArrayList<>;
    mList.addAll(mDatas);
    submitList(mList);
  }

  //清除全部数据
  protected void clear{
    mDatas.clear;
    submitList( null);
  }

  //获取adapter维护的数据集合
  protected List<T> getAdapterData{
  	returnmDatas;
  }

}
```

我们实现Adater的方法就如下：

```java
public class MyListAdapter extends BaseRVDifferAdapter<TestBean, MyListAdapter.ViewHolder> {
  public MyListAdapter(Context context){
  	super(context, new MyDiffUtilItemCallback);
  }

  @Override
  @NonNull
  public MyListAdapter.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, intviewType) {
  	return new ViewHolder(mInflater.inflate(R.layout.item_test, parent, false));
  }

  @Override
  public void onBindViewHolder(@NonNull ViewHolder holder, intposition, @NonNull List<Object> payloads) {
    if(payloads.isEmpty) {
    	onBindViewHolder(holder, position);
    } else{
			Bundle bundle = (Bundle) payloads.get( 0);
			holder.mTvName.setText(bundle.getString( "KEY_NAME"));
    }
  }

  @Override
  public void onBindViewHolder(@NonNull MyListAdapter.ViewHolder holder, final int position) {
    TestBean bean = getItem(position);
    holder.mTvName.setText(bean.getName);
  }

  static class ViewHolder extends RecyclerView.ViewHolder{
  	TextView mTvName;
  	ViewHolder(View itemView) {
  		super(itemView);
			mTvName = itemView.findViewById(R.id.tv_name);
  	}
  }

}
```

在我们自己的 Adpater 中，我们还是需要初始化我们自己的Diff.Callback的。那么在使用的时候也就和普通的RV设置 Adapter 是一样的。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_sorted_list);
  RecyclerView mRecyclerView = findViewById(R.id.rv);
  mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
  mAsyncListDifferAdapter = new AsyncListDifferAdapter(this);
  mRecyclerView.setAdapter(mAsyncListDifferAdapter);
  initData;
}

private void addData {
  List<TestBean> mList = new ArrayList;
  for(int i = 10; i < 20; i++){
  	mList. add(new TestBean(i, "Item "+ i));
	}
	mMyListAdapter.addData(mList);
}

private void initData {
  List<TestBean> mList = new ArrayList;
  for(int i = 0; i < 10; i++){
 	 mList.add(new TestBean(i, "Item "+ i));
	}
	mMyListAdapter.setData(mList);
}

private void updateData{
  List<TestBean> mList = new ArrayList;
  for(int i = 20; i < 30; i++){
  	mList.add(newTestBean(i, "Item "+ i));
  }
  mMyListAdapter.addData(mList);
}
```

是不是可以无脑使用Diff的方式呢？也不是，当一些普通长列表我们使用默认的RV-Adapter就行了，加载更多，往源数据上面加数据，并不涉及到很多数据源改变的也没必要使用Diff。

那么哪里使用？比如IM的会话列表，长期变动的数据，比如评论区域热评区的数据切换时，比如我们的Android掘金App：

![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/uPic/b6031a7d638046d1a0d49cf9e109493b.webp)

直接暴力的切换数据源然后 notifyDataSetChanged，这样就会整个页面闪烁。而对于这些特定的一些场景，我们使用 Diff 的功能，就会感觉更加的流畅，体验会好一点哦 ! 

## 3. DiffUtil的封装

既然AsyncListDiffer 和ListAdapter 都是快速实现的方式，那我们直接使用基本 DiffUtil 行不行？

当然可以，我不想直接使用ListAdapter，我就想用RV.Adapter,因为我有其他的封装与扩展，我自己会使用异步线程，我不需要你帮我管理，我不需要使用你的 AsyncListDiffer 帮我管理我的 List 对象 。

我们可以直接使用基本的 DiffUtil 。真实使用下来也不是很复杂，只需要做亿点点的封装也能很方便的实现逻辑。

先写一个Differ的配置文件,内部可配置主线程，异步线程，和必备的DiffUtil.Callback：

```kotlin
class MyAsyncDifferConfig<T> (
  @SuppressLint( "SupportAnnotationUsage")
  @RestrictTo(RestrictTo.Scope.LIBRARY)
  val mainThreadExecutor: Executor?,
  val backgroundThreadExecutor: Executor,
	val diffCallback: DiffUtil.ItemCallback<T>
) {
  class Builder<T> (private val mDiffCallback: DiffUtil.ItemCallback<T>) {
    companion object{
      private val sExecutorLock = Any
      private var sDiffExecutor: Executor? = null
    }

    private var mMainThreadExecutor: Executor? = null
    private var mBackgroundThreadExecutor: Executor? = null
    fun setMainThreadExecutor(executor: Executor?) : Builder<T> {
      mMainThreadExecutor = executor
      return this
    }

    fun setBackgroundThreadExecutor(executor: Executor?) : Builder<T> {
      mBackgroundThreadExecutor = executor
      return this
    }

    fun build: MyAsyncDifferConfig<T> {
      if(mBackgroundThreadExecutor == null) {
        synchronized(sExecutorLock) {
          if(sDiffExecutor == null) {
          	sDiffExecutor = Executors.newSingleThreadExecutor
          }
        }
      	mBackgroundThreadExecutor = sDiffExecutor
      }

      return MyAsyncDifferConfig(
      mMainThreadExecutor,
      mBackgroundThreadExecutor!!,
      mDiffCallback)
    }
  }

}
```

重点是定义一个自己的AsyncDiffer，内部使用 DiffUtil 来计算差分。

```kotlin
class MyAsyncDiffer<T> (
  private val adapter: RecyclerView.Adapter<*>,
  private val config: MyAsyncDifferConfig<T>
) {
	private val mUpdateCallback: ListUpdateCallback = AdapterListUpdateCallback(adapter)
	private var mMainThreadExecutor: Executor = config.mainThreadExecutor ?: MainThreadExecutor
  private val mListeners: MutableList<ListChangeListener<T>> = CopyOnWriteArrayList
  private var mMaxScheduledGeneration = 0
  private var mList: List<T>? = null
  private var mReadOnlyList = emptyList<T>

  private class MainThreadExecutorinternal constructor: Executor {
    val mHandler = Handler(Looper.getMainLooper)
    override fun execute(command: Runnable) {
      mHandler.post(command)
    }
  }

  fun getCurrentList: List<T> {
    return mReadOnlyList
  }

  @JvmOverloads
  fun submitList(newList: MutableList<T>?, commitCallback: Runnable? = null) {
    val runGeneration: Int= ++mMaxScheduledGeneration
    if(newList == mList) {
      commitCallback?.run
      return
    }
    val previousList = mReadOnlyList
    if(newList == null) {
      val countRemoved = mList?.size ?: 0
      mList = null
      mReadOnlyList = emptyList
      mUpdateCallback.onRemoved(0, countRemoved)
      onCurrentListChanged(previousList, commitCallback)
      return
    }
    if(mList == null) {
      mList = newList
      mReadOnlyList = Collections.unmodifiableList(newList)
      mUpdateCallback.onInserted(0, newList.size)
      onCurrentListChanged(previousList, commitCallback)
      return
    }
    val oldList: List<T> = mList asList<T>
    config.backgroundThreadExecutor.execute {
      valresult = DiffUtil.calculateDiff(object: DiffUtil.Callback {
        override fun getOldListSize: Int{
          returnoldList.size
        }
        override fun getNewListSize: Int{
          returnnewList.size
        }

        override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int) : Boolean{
          valoldItem: T? = oldList[oldItemPosition]
          valnewItem: T? = newList[newItemPosition]
          return if(oldItem != null&& newItem != null) {
            config.diffCallback.areItemsTheSame(oldItem, newItem)
          } else
            oldItem == null&& newItem == null
        }

        override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int) : Boolean{
          valoldItem: T? = oldList[oldItemPosition]
          valnewItem: T? = newList[newItemPosition]
          if(oldItem != null&& newItem != null) {
            return config.diffCallback.areContentsTheSame(oldItem, newItem)
          }
          if(oldItem == null&& newItem == null) {
            return true
          }
          throwAsserti
        }

        override fun getChangePayload(oldItemPosition: Int, newItemPosition: Int) : Any? {
          valoldItem: T? = oldList[oldItemPosition]
          valnewItem: T? = newList[newItemPosition]
          if(oldItem != null&& newItem != null) {
            return config.diffCallback.getChangePayload(oldItem, newItem)
          }
          throwAsserti
        }
			})

      mMainThreadExecutor.execute {
        if(mMaxScheduledGeneration == runGeneration) {
          latchList(newList, result, commitCallback)
        }
      }
  	}
  }
  
  private fun latchList(
    newList: MutableList<T>,
    diffResult: DiffUtil. DiffResult,
    commitCallback: Runnable?
  ) {
    val previousList = mReadOnlyList
    mList = newList
    mReadOnlyList = Collections.unmodifiableList(newList)
    diffResult.dispatchUpdatesTo(mUpdateCallback)
    onCurrentListChanged(previousList, commitCallback)
  }

	private fun onCurrentListChanged(
    previousList: List< T>,
    commitCallback: Runnable?
  ) {
    for(listener inmListeners) {
      listener.onCurrentListChanged(previousList, mReadOnlyList)
    }
    commitCallback?.run
  }

  //定义接口
  interface ListChangeListener<T> {
  	fun onCurrentListChanged(previousList: List< T>, currentList: List T>)
  }
  
  fun addListListener(listener: ListChangeListener< T>) {
  	mListeners.add(listener)
  }

  fun removeListListener(listener: ListChangeListener< T>) {
  	mListeners.remove(listener)
  }

  fun clearAllListListener{
  	mListeners.clear
  }

}
```

然后我们可以用委托的方式配置，可以让普通的RecyclerView.Adapter 也能通过配置的方式选择是否使用Differ。

实现我们的控制类接口。

```kotlin
typealias IDiffer<T> = IMyDifferController<T>
fun <T> differ: MyDifferController<T> = MyDifferController
interface IMyDifferController< T> {
	fun RecyclerView.Adapter <*>.initDiffer(config: MyAsyncDifferConfig< T>) : MyAsyncDiffer<T>
	fun getDiffer: MyAsyncDiffer<T>?
	fun getCurrentList: List<T>
	fun setDiffNewData(list: MutableList< T>, commitCallback: Runnable? = null)
	fun addDiffNewData(list: MutableList< T>, commitCallback: Runnable? = null)
	fun addDiffNewData(t: T, commitCallback: Runnable? = null)
	fun removeDiffData(index: Int)
	fun clearDiffData
	fun RecyclerView.Adapter <*>. onCurrentListChanged(previousList: List< T>, currentList: List< T>)
}
```

在对控制类接口实例化，做一些具体的操作逻辑。

```kotlin
class MyDifferController<T> : IMyDifferController< T> {
	private var mDiffer: MyAsyncDiffer<T>? = null
	override fun RecyclerView.Adapter <*>. initDiffer(config: MyAsyncDifferConfig< T>) : MyAsyncDiffer<T> {
	mDiffer = MyAsyncDiffer( this, config)
	val mListener: MyAsyncDiffer.ListChangeListener<T> = object: MyAsyncDiffer.ListChangeListener<T> {
	
    override fun onCurrentListChanged(previousList: List< T>, currentList: List< T>) {
      this@initDiffer.onCurrentListChanged(previousList, currentList)
    }
	}

  mDiffer?.addListListener(mListener)
 	 return mDiffer!!
  }

  override fun getDiffer: MyAsyncDiffer<T>? {
    return mDiffer
  }

  override fun getCurrentList: List<T> {
    return mDiffer?.getCurrentList ?: emptyList
  }

	override fun setDiffNewData(list: MutableList< T>, commitCallback: Runnable?) {
		mDiffer?.submitList(list, commitCallback)
	}

	override fun addDiffNewData(list: MutableList< T>, commitCallback: Runnable?) {
		val newList = mutableListOf<T>
		newList.addAll(mDiffer?.getCurrentList ?: emptyList)
		newList.addAll(list)
		mDiffer?.submitList(newList, commitCallback)
	}

	override fun addDiffNewData(t: T, commitCallback: Runnable?) {
		val newList = mutableListOf<T>
		newList.addAll(mDiffer?.getCurrentList ?: emptyList)
		newList.add(t)
		mDiffer?.submitList(newList, commitCallback)
	}

	override fun removeDiffData(index: Int) {
		val newList = mutableListOf<T>
		newList.addAll(mDiffer?.getCurrentList ?: emptyList)
		newList.removeAt(index)
		mDiffer?.submitList(newList)
	}

	override fun clearDiffData{
		mDiffer?.submitList( null)
	}

  override fun RecyclerView.Adapter <*>. onCurrentListChanged(previousList: List< T>, currentList: List< T>) {
  }

}
```

到此我们就能封装一个DiffUtil的工具类了，我们可以选择是否启用Diff,例如我们不使用Diff，我们使用Adapter就是一个普通的Adaper。

```kotlin
class MyDiffAdapter: RecyclerView.Adapter<BaseViewHolder> {

  private val mDatas = arrayListOf<DemoDiffBean>

  fun addData(list : List< DemoDiffBean>) {
    mDatas.addAll(list)
    notifyDataSetChanged
  }

  override fun onBindViewHolder(holder: BaseViewHolder, position: Int, payloads: MutableList< Any>) {
    if(!CheckUtil.isEmpty(payloads) && (payloads[ 0] asString) == "text") {
      YYLogUtils.w( "差分刷新 -------- 文本更新")
      holder.setText(R.id.tv_job_text, mDatas[position].content)
    } else{
    	onBindViewHolder(holder, position)
    }
  }

  override fun onBindViewHolder(holder: BaseViewHolder, position: Int) {
    YYLogUtils.w( "默认数据赋值 --------")
    holder.setText(R.id.tv_job_text, mDatas[position].content)
  }

  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int) : BaseViewHolder {
    return BaseViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.item_diff_jobs, parent, false))
  }

  override fun getItemCount: Int{
    return mDatas.size
  }

}
```

如果我们想启动Diff的功能的时候，实现这个接口并委托实现即可启用Diff。

```kotlin
class MyDiffAdapter: RecyclerView.Adapter<BaseViewHolder>, IDiffer<DemoDiffBean> by differ {

  init {
    initDiffer(MyAsyncDifferConfig.Builder(DiffDemoCallback).build)
  }

  override fun onBindViewHolder(holder: BaseViewHolder, position: Int, payloads: MutableList< Any>) {
    if(!CheckUtil.isEmpty(payloads) && (payloads[ 0] asString) == "text") {
      YYLogUtils.w( "差分刷新 -------- 文本更新")
      holder.setText(R.id.tv_job_text, getCurrentList[position].content)
    } else{
      onBindViewHolder(holder, position)
    }
  }

  override fun onBindViewHolder(holder: BaseViewHolder, position: Int) {
    YYLogUtils.w( "默认数据赋值 --------")
    holder.setText(R.id.tv_job_text, getCurrentList[position].content)
  }

  overridefunonCreateViewHolder(parent: ViewGroup, viewType: Int) : BaseViewHolder {
    return BaseViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.item_diff_jobs, parent, false))
  }

  overridefungetItemCount: Int{
    return getCurrentList.size
  }

}
```

使用的时候也是超方便的。

```kotlin
private fun initRV{
  mAdapter = MyDiffAdapter
  findViewById<RecyclerView>(R.id.recyclerView).vertical.apply {
    adapter = mAdapter
    divider(Color.BLACK)
  }
}

private fun initData{
  mDatas.clear
  for(i in1..10) {
  	mDatas.add(DemoDiffBean(i, "conetnt: $i" ))
  }
  mAdapter.setDiffNewData(mDatas)
}

private fun initListener{
	findViewById<View>(R.id.diff_1).click {
    vallist = mutableListOf<DemoDiffBean>
		for(i in1..10) {
			list add(DemoDiffBean(i, "Diff1 conetnt: $i" ))
		}
		mAdapter.setDiffNewData(list)
	}
	findViewById<View>(R.id.diff_2).click {
		vallist = mutableListOf<DemoDiffBean>
		for(i in1..10) {
			list.add(DemoDiffBean(i, "Diff3 conetnt: $i" ))
		}
		list.removeAt( 0)
		list.removeAt( 1)
		list.removeAt( 2)
		list[ 3].content = "自定义乱改的数据"
		mAdapter.setDiffNewData(list)
	}

}
```

运行的效果如下：

![](https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/uPic/305249a9c49047d5a0b2a00b6e1446fb.webp)

是不是很方便呢？可能有人会问，这么封装有什么好处？

其实这样通过配置的方式，我们可以使用在任意的RV.Adapter上面，包括我们自己定义的BaseAdapter，LoadMoreAdapter等。相对比较灵活吧，方便在原有的效果上快速修改。

当然了，其实关于 Diff 的实现，有这么多种方式可以让大家使用，每一种方案都能实现同样的效果，只是看大家愿不愿意优化而已，每一种方式使用起来都不算难。

## 4. 小结

关于 paylpoad 和 Diff 的问题，既然 Diff 是在 payload 的基础上实现的，那是不是有 Diff 功能之后我们就不需要手动 payload 了呢？

也不是，上面的介绍中已经讲过了，如果数据频繁的切换，最好是使用Diff，如果就是类似普通的评论列表点赞的效果，我们手动 payload 即可。他们有各自的使用场景。

需要注意的是，如果使用 Diff 要留意对象指针的问题，DiffUtil 首先检查新提交的 newList 与内部持有的 mList 的引用是否相同, 如果相同, 就直接返回。如果不同的引用，才会对 newList 和 mList 做 Diff 算法比较。

可以看的我的 Demo 都是直接另 new 一个 List 来进行操作的，正常开发场景一般我们都是从服务器拿的不同的 List，也不会有问题。而如果从本地拿的数据去比对的时候就需要注意，对比的对象和原有的对象是否是同一个对象，此时可以考虑对象的深拷贝来实现新对象，再拿去和原有对象进行差分对比。