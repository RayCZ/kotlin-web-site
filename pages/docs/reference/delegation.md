---
type: doc
layout: reference
category: "Syntax"
title: "Delegation"
---

# Delegation

Delegation ：委派

## Property Delegation

Property Delegation ：屬性委派

Delegated properties are described on a separate page: [Delegated Properties](delegated-properties.md).

委派屬性在單獨頁面上描述：[Delegated Properties](delegated-properties.md)

## Implementation by Delegation

The [Delegation pattern](https://en.wikipedia.org/wiki/Delegation_pattern) has proven to be a good alternative to implementation inheritance, and Kotlin supports it natively requiring zero boilerplate code. A class `Derived` can implement an interface `Base` by delegating all of its public members to a specified object:

**程式設計中的模組化程式碼，可以稱為boilerplate code，意即這些程式碼不會變**

已證明委派樣式是實現繼承的一個很好替代方案，且 Kotlin 原生支援委派樣式需要零樣版的代碼 。一個類別 `Derived` 可以通過委派它的所有公開成員給指定物件 `by b` 實作一個介面 `Base` ：


``` kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print()
}
```

The *by*-clause in the supertype list for `Derived` indicates that `b` will be stored internally in objects 
of `Derived` and the compiler will generate all the methods of `Base` that forward to `b`.

`Derived` 的超 (父) 類型列表中 `by` 子句，表示 `b` 內部儲存在 `Derived` 的物件，編輯器將生成轉發到 `b` 的所有 `Base` 方法。

### Overriding a member of an interface implemented by delegation 

[Overrides](classes.html#overriding-methods) work as you might expect: the compiler will use your `override` 
implementations instead of those in the delegate object. If we were to add `override fun print() { print("abc") }` to 
`Derived`, the program would print "abc" instead of "10" when `print` is called:

<div class="sample" markdown="1" theme="idea">

``` kotlin
interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    Derived(b).printMessageLine()
}
```
</div>

Note, however, that members overridden in this way do not get called from the members of the 
delegate object, which can only access its own implementations of the interface members:

<div class="sample" markdown="1" theme="idea">

``` kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // This property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```
</div>
