---
layout: post
title: 【转】Kotlin 中 opt-in 的使用说明
tags: kotlin
author: hulu-knight
date: 2023-05-23 20:52 +0800
---


之前在给公司项目封装库的时候，领导告诉我封装的漂亮一点，等以后公司发展起来了可能需要把这个库提供给第三方接入使用。

此时，就有这么一个问题：某些功能函数使用条件比较苛刻，直接使用可能会出现意想不到的后果，如果想要使用，需要结合其他状态判断是否可以使用。

为了避免第三方接入时误操作，我为这个使用条件苛刻的函数另外封装了一个可以直接使用的新函数。

但是，即使如此，出于测试和维护需求，我也不能移除或者将原函数设置为私有（private）函数。

那么问题来了，我要怎么避免其他同事或者第三方在使用时不会误调用这个函数，同时又能在知晓直接使用可能导致的后果时依旧能够使用呢？

靠文档声明？显然这是不可靠的，就算你在文档中大写加粗标红强调这类函数的危险性，依旧会有人视而不见。

当时我在谷歌苦苦搜寻了好久，终于在 Kotlin 官方文档中找到一个完美契合我的需求的功能，那就是 Opt-in 。

当时关于 Opt-in 的资料，除了官方文档几乎没有其他资料，我也没有在实际中见到有什么库或者程序使用这个功能，所以我用起来还是觉得心里发怵。

直到今年我开始接触了 Compose ，我才发现，原来 Compose 中已经大量应用了这个功能：

![s1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ef1909b3d5d45ab9d26cf041c7fb6df~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

![s2.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20daa146bb4b4bd6b538bc7e30d68df7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

并且在今年的年中发布的 Kotlin 1.7.0 中，该功能终于发布了正式版本。

所以，是时候介绍一下这个功能了。

# 正文

## 什么是 Opt-in

根据官方文档介绍。

Opt-in 是 Kotlin 标准库中的一个方法，用于声明使用某些 API 需要明确的同意。该功能可以让开发者告知 API 使用者使用某些 API 需要一些特定的条件，如果使用者已经知晓则需要明确声明依旧使用（Opt-in）才能继续使用该 API。

例如，某些 API 尚处于测试阶段，未来可能会发生变化；亦或是我前言中提到的场景，都非常适合使用该方法。

如果我们声明了某个方法（functiuon）或类(class)需要 Opt-in ，则IDE或编译器会发出警告，要求使用者明确标注需要使用（Opt-in）。

## 如何使用

在介绍怎么编写 Opt-in 注解之前，我们先简单介绍一下如何使用。

这里我们就以 Compose 中 `LazyColunm` 的 `stickyHeader` 函数举例，我们不需要关心这个函数的具体实现，只需要知道这个函数被标记为了 Opt-in ：

```kotlin
@ExperimentalFoundationApi
fun stickyHeader(
    key: Any? = null,
    contentType: Any? = null,
    content: @Composable LazyItemScope.() -> Unit
)

@RequiresOptIn(
    "This foundation API is experimental and is likely to change or be removed in the " +
        "future."
)
annotation class ExperimentalFoundationApi
```

其中 `ExperimentalFoundationApi` 即用于标记需要选择加入的注解名称。

可以看到， `stickyHeader` 被加上了 `@ExperimentalFoundationApi` 的注解。

此时，如果我们直接调用 `stickyHeader` ，将会收到如下的 IDE 错误：

![s3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26da56f8a2004ff4b53debe6ea1924d9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

如果我们无视这个警告，直接编译的话也会编译出错。

为了消除这个警告，我们可以选择加上注解： `@OptIn(ExperimentalFoundationApi::class)` 表示我们已知晓使用 `stickyHeader` 的风险，并且依旧需要使用。

加上上述注解后，错误消失，也可以正常编译运行了。

`@OptIn` 的作用域可以是方法（函数）、类、文件：

```kotlin
// 1. 注解方法
@OptIn(ExperimentalFoundationApi::class)
fun Test() {

}


// 2. 注解类
@OptIn(ExperimentalFoundationApi::class)
class Test {

}

// 3. 注解整个文件
@file:OptIn(ExperimentalFoundationApi::class)

```

分别对应这个方法选择加入、这个类中的所有方法都选择加入、整个文件中的所有方法，函数，类都加入。

需要注意的是，直接使用 `OptIn(ExperimentalFoundationApi::class)` 表示的是不传递选择加入，即如果我们在 `Test()` 函数中注解了 `OptIn(ExperimentalFoundationApi::class)` ，则调用 `Test()` 的地方不需要再声明 `OptIn(xxx)`：

![s4.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da77641d062d49f9875878c443189579~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

如果我们想要让选择加入传递下去，则可以更改 `Test()` 的注解为 `@ExperimentalFoundationApi`，此时调用了 `Test()` 的地方也需要声明选择加入：

![s5.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8db3167ab06149f88b68124303f525f6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

最后，可能有人想问，我不想到处都写上 `OptIn(xxxx)` 怎么办？就算可以注解给整个文件我也觉得很麻烦啊。

那么，你可以选择直接给整个模块（Moudle）都加上注解，我们需要给当前模块的编译配置加上以下编译选项：

```groovy
tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).configureEach {
    kotlinOptions {
        freeCompilerArgs += "-opt-in=org.mylibrary.OptInAnnotation"
    }
}
```

需要注意的是对于 Kotlin 1.6.0 之前的版本，请将 `-opt-in`  替换为 `-Xopt-in` 。

另外，如果使用的是 kts，则为：

```kotlin
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
    kotlinOptions.freeCompilerArgs += "-opt-in=org.mylibrary.OptInAnnotation"
}
```

## 如何自己编写

上面我们简单讲解了如何使用 Opt-in 。大家现在应该对 Opt-in 有了一个大致的理解，所以接下来我们讲解如何自己写一个 Opt-in 注解。

创建选择加入注解和创建普通注解一样，只是多了一个额外的配置选项：

```kotlin
@RequiresOptIn("直接使用该方法可能会导致意想不到的错误，请确认已知晓该方法可能会产生的问题后再使用", RequiresOptIn.Level.ERROR)
annotation class NotSafeForUse
```

在上面的代码中我们创建了一个名为 `NotSafeForUse` 的注解。

并且为 `NotSafeForUse` 添加了一个 `@RequiresOptIn` 注解，该注解用于声明 `NotSafeForUse` 是一个选择加入的注解。

`@RequiresOptIn` 接收两个参数：

- `message` 即使用时的提示文本
- `level` 警告级别

警告级别可选择 `RequiresOptIn.Level.ERROR` 或 `RequiresOptIn.Level.WARNING` 。

ERROR 表示被注解的地方强制启用选择加入，如果不声明选择加入，则编译将不通过：

```kotlin
@NotSafeForUse
fun testFun() {

}

@RequiresOptIn("直接使用该方法可能会导致意想不到的错误，请确认已知晓该方法可能会产生的问题后再使用", 
annotation class NotSafeForUse
```

以上代码在IDE会被标注红色下划线警告，并且编译时将报错：

![s6.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83dc2444a644aa7ab2417eb7432e582~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

如果把级别改为 WARNING 则仅警告而不会导致编译失败，同时 IDE 也只会提示弱化的警告：

![s7.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cac10929f0e412fa66a72290fafed0c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

同样的，普通注解可以使用的配置参数，选择加入也可以使用：

```kotlin
@RequiresOptIn("直接使用该方法可能会导致意想不到的错误，请确认已知晓该方法可能会产生的问题后再使用", RequiresOptIn.Level.ERROR)
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class NotSafeForUse
```

需要注意的是，`@Retention` 需要为 `BINARY` 或 `RUNTIME`

另外，对于选择加入的注解，还需要满足以下条件：

- No EXPRESSION, FILE, TYPE, or TYPE_PARAMETER among targets
- No parameters.

## 其他问题

在 Kotlin 1.7.0 之前，Opt-in 自身也处于 Opt-in 状态，所以如果我们的 Kotlin 版本在 1.7.0 之前，想要使用 Opt-in 必须先声明 Opt-in Opt-in（绕口令呢？）

为了声明使用选择加入，我们需要添加编译配置：

```groovy
tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).all {
    kotlinOptions.freeCompilerArgs += "-opt-in=kotlin.RequiresOptIn"
}
```

对于 kts 则使用：

```kotlin
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
    kotlinOptions.freeCompilerArgs += "-Xopt-in=kotlin.RequiresOptIn"
}
```

如果不添加这个编译选项的话。直接使用 Opt-in 会警告：

![s8.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66f6a7745454422281ff6206350886d1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

# 参考资料

1. [Opt-in requirements](https://link.juejin.cn?target=https%3A%2F%2Fkotlinlang.org%2Fdocs%2Fopt-in-requirements.html)
2. [What's new in Kotlin 1.7.0](https://link.juejin.cn?target=https%3A%2F%2Fkotlinlang.org%2Fdocs%2Fwhatsnew17.html)

转载作者：equationl
转载链接：https://juejin.cn/post/7157950373250990093
来源：稀土掘金