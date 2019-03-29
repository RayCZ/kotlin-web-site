---
type: doc
layout: reference
category: "Syntax"
title: "Nested and Inner Classes"
---

# Nested and Inner Classes

Nested and Inner Classes ：內嵌和內部類別

Classes can be nested in other classes:

類別可以嵌套在其他類別：

``` kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## Inner classes

Inner classes ：內部類別

A class may be marked as *inner* to be able to access members of outer class. Inner classes carry a reference to an object of an outer class:

一個類別可以被標記為 `inner` 能夠去存取外部類別的成員。內部類別帶有一個參照引用一個外部類別的物件：

``` kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

**foo 方法是指向 Outer 類別的 bar**

See [Qualified *this* expressions](this-expressions.md) to learn about disambiguation of *this* in inner classes.

請參閱 [Qualified *this* expressions](this-expressions.md) 學習關於 `this` 在內部類別的消除分歧。

## Anonymous inner classes

Anonymous inner classes ：匿名內部類別

Anonymous inner class instances are created using an [object expression](object-declarations.md#object-expressions):

使用 [object expression](object-declarations.md#object-expressions) `object: MouseAdapter() {..}` 創建匿名內部類別實例：

``` kotlin
window.addMouseListener(object: MouseAdapter() {

    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
})
```

If the object is an instance of a functional Java interface (i.e. a Java interface with a single abstract method), you can create it using a lambda expression prefixed with the type of the interface:

如果物件是一個函數化 Java 介面的實例 (換句話說，使用一個單一抽象方法的 Java 介面) ，你可以使用介面的類型當前綴 `ActionListener` 使用一個 Lambda 表達式 `{}` 去建立它：

``` kotlin
val listener = ActionListener { println("clicked") }
```
