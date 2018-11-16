---
type: doc
layout: reference
category: "Other"
title: "Type aliases (since 1.1)"
---

## Type aliases

Type aliases ：類型別名

Type aliases provide alternative names for existing types. If the type name is too long you can introduce a different shorter name and use the new one instead.

類型別名對已存在的類型提供代替名稱。如果類型名稱太長，你可以引入不同更短的名稱以及使用新的名稱。

It's useful to shorten long generic types. For instance, it's often tempting to shrink collection types:

縮短長泛型的類型是有用的。例如：縮小集合類型通常很誘人：

``` kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

You can provide different aliases for function types:

你可以為函數類型提供不同別名：

``` kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```

You can have new names for inner and nested classes:

你可以為內部或內嵌類型有新的名稱：

``` kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

Type aliases do not introduce new types. They are equivalent to the corresponding underlying types. When you add `typealias Predicate<T>` and use `Predicate<Int>` in your code, the Kotlin compiler always expand it to `(Int) -> Boolean`. Thus you can pass a variable of your type whenever a general function type is required and vice versa:

類型別名不會引入新的類型。他們相當於對應的底層類型。當你添加 `typealias Predicate<T>` 並在你的代碼中使用 `Predicate<Int>` ， Kotlin 編譯器總是擴展它是 `(Int) -> Boolean` 。因此每當需要泛型函數時，你可以傳遞你類型的變數，反過來也是如此。

``` kotlin
typealias Predicate<T> = (T) -> Boolean // 函數類型

fun foo(p: Predicate<Int>) = p(42) // 相當於 p : (T) -> Boolean , 調用的 f(42)

fun main(args: Array<String>) {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // prints "true" , 42 > 0 , foo { it > 0 }

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // prints "[1]"
}
```
