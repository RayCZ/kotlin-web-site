---
type: doc
layout: reference
category: "Syntax"
title: "This expressions"
---

# This Expression

This Expression ： `this` 表達式

To denote the current _receiver_, we use *this* expressions:

表示目前的「receiver」，我們使用 `this` 表達式：

* In a member of a [class](classes.md#inheritance), *this* refers to the current object of that class.
  在一個[類別](classes.md#inheritance)的一個成員， `this` 引用該類別目前的物件。
* In an [extension function](extensions.md) or a [function literal with receiver](lambdas.md#function-literals-with-receiver) *this* denotes the _receiver_ parameter that is passed on the left-hand side of a dot.
  在[擴展函數](extensions.md)或[使用 receiver 的函數文字](lambdas.md#function-literals-with-receiver) `this` 表示逗點的左手邊傳遞 receiver 參數。

If *this* has no qualifiers, it refers to the _innermost enclosing scope_. To refer to *this* in other scopes, _label qualifiers_ are used:

如果 `this` 沒有修飾符，它引用到最裡面的封閉範圍。在其他範圍引用 `this` ，使用標籤修飾符 `@` 。

## Qualified *this*
Qualified *this* ：修飾 `this`

To access *this* from an outer scope (a [class](classes.md), or [extension function](extensions.md), or labeled [function literal with receiver](lambdas.md#function-literals-with-receiver)) we write `this@label` where `@label` is a [label](returns.md) on the scope *this* is meant to be from:

從外部範圍存取 `this` (類別，或擴展函數，標記標籤 `@` 使用 receiver 函數文字 ) ，我們寫 `this@label` 之中的 `@label` 是在範圍中的[標籤](returns.md)意味著 `this` 來自於：

``` kotlin
class A { // implicit label @A
    inner class B { // implicit label @B
        fun Int.foo() { // implicit label @foo
            val a = this@A // A's this , 指定 class A
            val b = this@B // B's this , 指定 class B

            val c = this // foo()'s receiver, an Int , 指定當下範圍的物件
            val c1 = this@foo // foo()'s receiver, an Int , 指定擴展函數調用的物件
    
            val funLit = lambda@ fun String.() {
                val d = this // funLit's receiver
            }
            
            //funLit 調用時
            "可以在裡面調用 this".funLit() //上面的 this 會是 "可以在裡面調用 this"


            val funLit2 = { s: String ->
                // foo()'s receiver, since enclosing lambda expression
                // doesn't have any receiver , this 會指向 Int.foo() 的 receiver
                val d1 = this
            }
            
            //funLit2 調用時
            funLit2("this 是 Int.foo()") //Int.foo() 的 receiver
        }
    }
}

```