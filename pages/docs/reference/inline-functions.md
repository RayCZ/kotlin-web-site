---
type: doc
layout: reference
category: "Syntax"
title: "Inline Functions and Reified Type Parameters"
---

# Inline Functions

Inline Functions ：行內置入函數

Using [higher-order functions](lambdas.md) imposes certain runtime penalties: each function is an object, and it captures a closure, i.e. those variables that are accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead.

使用高階函數會強加於某部分運行時的損失：每個函數都是一個物件，並且它捕獲一個閉包，即是在函數內文中存取的那些變數。記憶體配置 (用於函數物件和類別) 並且虛擬的調用都會引入運行時的開銷。

But it appears that in many cases this kind of overhead can be eliminated by inlining the lambda expressions. The functions shown below are good examples of this situation. I.e., the `lock()` function could be easily inlined at call-sites. Consider the following case:

但似乎在很多情況下透過行內置入 Lambda 表達可以消除這種開銷。下面顯示的功能是這種情況的好範例。即是 `lock()` 函數可以在調用場景容易的行內置入。考慮以下情況：

``` kotlin
lock(l) { foo() }
```

Instead of creating a function object for the parameter and generating a call, the compiler could emit the following code:

編譯器可以發送以下代碼，而不是為參數建立函數物件並生成調用：

``` kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```

Isn't it what we wanted from the very beginning?

從一開始就不是我們想要的嗎？

To make the compiler do this, we need to mark the `lock()` function with the `inline` modifier:

為了使編譯器做到這件事，我們需要使用 `inline` 修飾符標記 `lock` 函數：

``` kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T { ... }
```

The `inline` modifier affects both the function itself and the lambdas passed to it: all of those will be inlined into the call site.

`inline` 修飾符影響函數本身和傳遞給它的 Lambda 表達法：所有這些將行內置入到調用場景。

Inlining may cause the generated code to grow; however, if we do it in a reasonable way (i.e. avoiding inlining large functions), it will pay off in performance, especially at "megamorphic" call-sites inside loops.

行內置入可能會造成生成代碼的增長；然而，如果我們在合理的方式進行 (即是避免行內置入大量函數) ，它將在性能上回報，特別是在循環內的 "變形" 調用場景。

**凡是標記 `inline` 的函數，會在編譯時期，在調用的地方由函數內的代碼直接轉換**

## noinline

noinline ：非行內置入，保留函數原本的特性，而不會取代置入代碼

In case you want only some of the lambdas passed to an inline function to be inlined, you can mark some of your function parameters with the `noinline` modifier:

假使你只想要部份的 Lambda 表達式傳遞給行內置入函數為以下的 `inlined: () -> Unit` ，你可以使用 `noinline` 修飾符標些部份你的函數參數 `noinline notInlined: () -> Unit`：

``` kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... }
```

Inlinable lambdas can only be called inside the inline functions or passed as inlinable arguments, but `noinline` ones can be manipulated in any way we like: stored in fields, passed around etc.

可行內置入的 Lambda 表達式只可以在行內置入的函數或傳遞作為可行內置入參數內調用，但 `noinline` 可以在任何我們想要的方式操作：在欄位儲存、周圍傳遞等等。

Note that if an inline function has no inlinable function parameters and no [reified type parameters](#reified-type-parameters), the compiler will issue a warning, since inlining such functions is very unlikely to be beneficial (you can suppress the warning if you are sure the inlining is needed using the annotation `@Suppress("NOTHING_TO_INLINE")`).

注意：如果行內置入函數沒有標記 `inline` 的參數和 [`reified` 類型的參數](#reified-type-parameters) ，編譯器將發出警告，因為行內置入這樣函數是不太有益的 (如果你確定需要行內置入，使用註釋 `@Suppress("NOTHING_TO_INLINE")` ，你可以壓制警告) 。

**用 `inline` 的函數會在編譯時期，複製並取代整段代碼到調用的地方，而 `noinline` 則不會，變成是正常的函數調用方式，保該原本函數的特性**

## Non-local returns

Non-local returns ：非區域回傳

In Kotlin, we can only use a normal, unqualified `return` to exit a named function or an anonymous function. This means that to exit a lambda, we have to use a [label](returns.md#return-at-labels), and a bare `return` is forbidden inside a lambda, because a lambda cannot make the enclosing function return:

在 Kotlin 中，我們只可以使用一般的、不合格的 `return` 來離開已命名的函數或匿名函數。這意味著離開 Lambda 表達式，我們必須使用 [label](returns.md#return-at-labels) `@...`，和在 Lambda 表達式中

``` kotlin
fun ordinaryFunction(block: () -> Unit) {
    println("hi!")
}
//sampleStart
fun foo() {
    ordinaryFunction {
        return // ERROR: cannot make `foo` return here
    }
}
//sampleEnd
fun main(args: Array<String>) {
    foo()
}
```

But if the function the lambda is passed to is inlined, the return can be inlined as well, so it is allowed:

inline fun inlined(block: () -> Unit) {
    println("hi!")
}
``` kotlin
//sampleStart
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
//sampleEnd
fun main(args: Array<String>) {
    foo()
}
```

Such returns (located in a lambda, but exiting the enclosing function) are called *non-local* returns. We are used to this sort of construct in loops, which inline functions often enclose:

``` kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

Note that some inline functions may call the lambdas passed to them as parameters not directly from the function body, but from another execution context, such as a local object or a nested function. In such cases, non-local control flow is also not allowed in the lambdas. To indicate that, the lambda parameter needs to be marked with the `crossinline` modifier:

``` kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```

> `break` and `continue` are not yet available in inlined lambdas, but we are planning to support them too.

## Reified type parameters

Sometimes we need to access a type passed to us as a parameter:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```
</div>

Here, we walk up a tree and use reflection to check if a node has a certain type.
It’s all fine, but the call site is not very pretty:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```
</div>

What we actually want is simply pass a type to this function, i.e. call it like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
treeNode.findParentOfType<MyTreeNode>()
```
</div>

To enable this, inline functions support *reified type parameters*, so we can write something like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```
</div>

We qualified the type parameter with the `reified` modifier, now it’s accessible inside the function,
almost as if it were a normal class. Since the function is inlined, no reflection is needed, normal operators like `!is`
and `as` are working now. Also, we can call it as mentioned above: `myTree.findParentOfType<MyTreeNodeType>()`.

Though reflection may not be needed in many cases, we can still use it with a reified type parameter:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```
</div>

Normal functions (not marked as inline) cannot have reified parameters.
A type that does not have a run-time representation (e.g. a non-reified type parameter or a fictitious type like `Nothing`)
cannot be used as an argument for a reified type parameter.

For a low-level description, see the [spec document](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md).

{:#inline-properties}

## Inline properties (since 1.1)

The `inline` modifier can be used on accessors of properties that don't have a backing field.
You can annotate individual property accessors:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
​``` kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```
</div>

You can also annotate an entire property, which marks both of its accessors as inline:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```
</div>

At the call site, inline accessors are inlined as regular inline functions.

{:#public-inline-restrictions}

## Restrictions for public API inline functions

When an inline function is `public` or `protected` and is not a part of a `private` or `internal` declaration, it is considered a [module](visibility-modifiers.html#modules)'s public API. It can be called in other modules and is inlined at such call sites as well.

This imposes certain risks of binary incompatibility caused by changes in the module that declares an inline function in case the calling module is not re-compiled after the change.

To eliminate the risk of such incompatibility being introduced by a change in **non**-public API of a module, the public API inline functions are not allowed to use non-public-API declarations, i.e. `private` and `internal` declarations and their parts, in their bodies.

An `internal` declaration can be annotated with `@PublishedApi`, which allows its use in public API inline functions. When an `internal` inline function is marked as `@PublishedApi`, its body is checked too, as if it were public.

