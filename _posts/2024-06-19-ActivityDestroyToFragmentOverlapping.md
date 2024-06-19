---
layout: post
title: Activity销毁导致Fragment重叠问题
tags: kotlin
author: hulu-knight
date: 2024-06-19 21:09 +0800


---



最近发现了一个离谱的问题，当我打开某软件并长时间锁屏后重新打开并切换Tab时发现竟然出现了Fragment的重叠问题，回味一会发现其实是Activity销毁重建后Fragment重新加载所导致的问题，本文章就如何解决上述问题提供一点小小的思路。




# 1.造成重叠的原因

当我们的应用长期处于后台时，**系统回收了主页Activity**，当我们重新解锁后，应用处于前台，主页Activity被销毁并重新创建了，并且在销毁之前执行了`onSaveInstanceState(Bundle outState)`这个方法，该方法会保存当前的视图层，之前被实例化过的 Fragment 依然会出现在 Activity 中，此时的 **FragmentTransaction** 中的相当于又再次 add 了 fragment 进去的，**hide()和show()方法对之前保存的fragment已经失效了**，这个时候点击底部导航的话会重新去切换fragment，于是发生了重叠的问题。

# 2.如何复现

复现方法很简单，进入手机设置 -> **开发者选项** -> 应用 -> **开启不保留活动**（离开后即销毁每个活动） 即可达成复现条件

我们现在模拟主页Acitivty，存在AFramgnet和BFragment。

```kotlin
kotlin复制代码class FragmentActivity : AppCompatActivity() {
    //用的是DataBinding
    private lateinit var binding: ActivityFragmentBinding
    private var aFragment: AFragment? = null
    private var bFragment: BFragment? = null
    private var currentId = R.id.tv_a //当前id
    ....
 }
kotlin复制代码override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = DataBindingUtil.setContentView(this, R.layout.activity_fragment)
    binding.apply {
        tvA.setOnClickListener(onClickListener)
        tvB.setOnClickListener(onClickListener)
    }
    //初始化AFragment
    changeFragment(currentId)
}
```

点击事件：

```ini
ini复制代码private var onClickListener = OnClickListener {
    changeFragment(it.id)
}
```

在Activity创建时将通过`changeFragment（id: Int）`方法将AFragment创建并添加到视图中去。

```scss
scss复制代码//切换Fragment
private fun changeFragment(id: Int) {
    supportFragmentManager.beginTransaction().apply {
        hideFragment(this)
        when (id) {
            R.id.tv_a -> {
                aFragment?.let {
                    show(it)
                } ?: run {
                    aFragment = AFragment()
                    add(R.id.fl_container, aFragment!!)
                }
                binding.tvA.isSelected = true
            }

            R.id.tv_b -> {
                bFragment?.let {
                    show(it)
                } ?: run {
                    bFragment = BFragment()
                    add(R.id.fl_container, bFragment!!)
                }
                binding.tvB.isSelected = true
            }
        }
    }.commitNowAllowingStateLoss()

}
```

切换之前先通过`hideFragment（transaction: FragmentTransaction）`隐藏所有的Fragment：

```kotlin
kotlin复制代码private fun hideFragment(transaction: FragmentTransaction) {
    binding.tvA.isSelected = false
    binding.tvB.isSelected = false
    aFragment?.let {
        transaction.hide(it)
    }
    bFragment?.let {
        transaction.hide(it)
    }
}
```

现在我们将项目跑起来，并且在开发者模式中打开不保留活动，进入后切换B fragment后切入后台再切回前台就可以复现重叠的情况了：

<img src="https://raw.githubusercontent.com/hulu-knight/Clouding-Pic/master/picture202406192200006.GIF" style="zoom:50%;" />

# 3.如何解决

我们在使用`FragmentTransaction`的`add`方法时需要一并存入tag，通过`FragmentManager`的`findFragmentByTag`来初始化Fragment，重写`onSaveInstanceState`在Activity销毁时存入Tag，再` onCreate`中通过Tag查找存入id再次选中Fragment。

更改后的`onCreate`：

```scss
scss复制代码    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_fragment)
        binding.apply {
            tvA.setOnClickListener(onClickListener)
            tvB.setOnClickListener(onClickListener)
        }
        aFragment = AFragment()
        //通过savedInstanceState来获取销毁时选中的id
        savedInstanceState?.let { bundle ->
            bundle.getString(TAG)?.let {
                currentId = getCurrentId(it)
            }
        }
        changeFragmentSafe(currentId)
    }
```

更改后的切换Fragment方法`changeFragmentSafe(id: Int) `：

```scss
scss复制代码private fun changeFragmentSafe(id: Int) {
    currentId = id
    //获取Fragment
    val fragment = initFragment(id)
    supportFragmentManager.beginTransaction().apply {
        hideFragment(this)
        //add时加入tag
        fragment?.let {
            if (!it.isAdded) {
                add(R.id.fl_container, it, getFragmentTag(id))
            }
            show(it)
        }
        findViewById<TextView>(id).isSelected = true
    }.commitNowAllowingStateLoss()
}
```

初始化：

```kotlin
kotlin复制代码private fun initFragment(id: Int): Fragment? {
    //通过id转为tag来找对应的Fragment
    var fragment = supportFragmentManager.findFragmentByTag(getFragmentTag(id))
    when (id) {
        R.id.tv_a -> {
            if (fragment == null) {
                fragment = AFragment.newInstance()
            }
            aFragment = fragment as AFragment
        }

        R.id.tv_b -> {
            if (fragment == null) {
                fragment = BFragment.newInstance()
            }
            bFragment = fragment as BFragment
        }
    }
    return fragment
}
```

这里用了个比较繁琐的方法来转换当前id和存入的tag：

```kotlin
kotlin复制代码//tag 转 id
private fun getCurrentId(tag: String): Int {
    return when (tag) {
        "tv_a" -> {
            R.id.tv_a
        }

        "tv_b" -> {
            R.id.tv_b
        }

        else -> {
            R.id.tv_a
        }
    }
}

//id 转 tag
private fun getFragmentTag(id: Int): String {
    return when (id) {
        R.id.tv_a -> {
            "tv_a"
        }

        R.id.tv_b -> {
            "tv_b"
        }

        else -> {
            "tv_a"
        }
    }
}
```

通过以上的方法，当Activity销毁重建后，会先去从`FragmentManager`中使用tag获取对应的Fragment对象，如果没有再去重新创建Fragment实例，这样就解决了Fragment的重叠问题：

# 4. 小结

以上就是关于Activity销毁重建后Fragment重新加载的分析和解决方法，除此之外以上方法更多情况应用于在activity嵌套多层fragment，或者activity + fragment + viewpager等类似这种在初始化fragment的时候需要加载大量数据的情况时，会出现明显的类似白屏，闪屏的现象，其实就是数据加载较慢导致ui”没来的及“加载出来。所以在首次创建之后，在切换的时候通过先通过tag来进行判断是否存在可以有效缓解摆平白屏效果。

