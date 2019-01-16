---
type: doc
layout: reference
category: "Syntax"
title: "Inline Functions and Reified Type Parameters"
---

# Inline Functions

Inline Functions ：行內置入函數

Using [higher-order functions](lambdas.md) imposes certain runtime penalties: each function is an object, and it captures a closure, i.e. those variables that are accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead.

使用高階函數會強加於某部分運行時的損失：每個函數都是一個物件，並且物件捕獲一個閉包，即是在函數內文中存取的那些變數。記憶體配置 (用於函數物件和類別) 並且虛擬的調用都會引入運行時的開銷。

But it appears that in many cases this kind of overhead can be eliminated by inlining the lambda expressions. The functions shown below are good examples of this situation. I.e., the `lock()` function could be easily inlined at call-sites. Consider the following case:

但似乎在很多情況下透過行內置入 Lambda 表達式可以消除這種開銷。下面顯示的功能是這種情況的好範例。即是 `lock()` 函數可以在調用場景容易的行內置入。考慮以下情況：

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

可行內置入的 Lambda 表達式只可以在行內置入的函數或作為可行內置入參數傳遞內調用，但 `noinline` 可以在任何我們想要的方式操作：在欄位儲存、周圍傳遞等等。

Note that if an inline function has no inlinable function parameters and no [reified type parameters](#reified-type-parameters), the compiler will issue a warning, since inlining such functions is very unlikely to be beneficial (you can suppress the warning if you are sure the inlining is needed using the annotation `@Suppress("NOTHING_TO_INLINE")`).

注意：如果行內置入函數沒有標記 `inline` 的參數和 [`reified` 類型的參數](#reified-type-parameters) ，編譯器將發出警告，因為行內置入這樣函數是不太有益的 (如果你確定需要行內置入，使用註釋 `@Suppress("NOTHING_TO_INLINE")` ，你可以壓制警告) 。

**用 `inline` 的函數會在編譯時期，複製並取代整段代碼到調用的地方，而 `noinline` 則不會，變成是正常的函數調用方式，保該原本函數的特性**

## Non-local returns

Non-local returns ：非-區域回傳

In Kotlin, we can only use a normal, unqualified `return` to exit a named function or an anonymous function. This means that to exit a lambda, we have to use a [label](returns.md#return-at-labels), and a bare `return` is forbidden inside a lambda, because a lambda cannot make the enclosing function return:

在 Kotlin 中，我們只可以使用常規、沒有修飾符的 `return` 來離開已命名的函數或匿名函數。這意味著離開 Lambda 表達式，我們必須使用 [label](returns.md#return-at-labels) `@...`，和在 Lambda 表達式內禁止赤裸的 `return` ，因為 Lambda 表達式不可以使封閉的函數回傳：

``` kotlin
fun ordinaryFunction(block: () -> Unit) {
    println("hi!")
}
//sampleStart
fun foo() {
    ordinaryFunction {
        //這裡是 Lambda 表達式中，所以不可以直接使用 return
        return // ERROR: cannot make `foo` return here
    }
}
//sampleEnd
fun main(args: Array<String>) {
    foo()
}

//ans: 'return' is not allowed here
```

But if the function the lambda is passed to is inlined, the return can be inlined as well, so it is allowed:

但如果 Lambda 函數傳遞給以下的 `inlined` ，回傳也可以為 `inlined` ，因此被允許：
``` kotlin
inline fun inlined(block: () -> Unit) {
    println("hi!")
}

//sampleStart
fun foo() {
    inlined {
        //這裡是 Lambda 表達式中， inlined 函數有加 inline 所以可以 return
        return // OK: the lambda is inlined
    }
}
//sampleEnd
fun main(args: Array<String>) {
    foo()
}
```

Such returns (located in a lambda, but exiting the enclosing function) are called *non-local* returns. We are used to this sort of construct in loops, which inline functions often enclose:

這樣的回傳 (位置 Lambda 中，但離開封閉的函數) 被稱為非-區域回傳。我們習慣於循環中這種構造的排序，行內置入函數通常包含：

``` kotlin
fun hasZeros(ints: List<Int>): Boolean {
    //foreach 是 inline 函數
    ints.forEach {
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

Note that some inline functions may call the lambdas passed to them as parameters not directly from the function body, but from another execution context, such as a local object or a nested function. In such cases, non-local control flow is also not allowed in the lambdas. To indicate that, the lambda parameter needs to be marked with the `crossinline` modifier:

注意：一些行內置入函數可能調用傳遞給函數作為參數的 Lambda 表達式不直接從函數內文，而從另一個執行內容，例如一個區域物件或一個內嵌函數。在這樣的情況，非-區域控制流程在 Lambda 表達式也是不允許。為了指明這種情況， Lambda 參數需要使用 `crossinline` 修飾符標記：

``` kotlin
//在 inline 函數中
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```

> `break` and `continue` are not yet available in inlined lambdas, but we are planning to support them too.
>
> `break` 和 `continue` 在行內置入 Lambda 表達式中不可以使用，但我們正在計劃支援它們。

## Reified type parameters

Reified type parameters ：具體化類型參數

Sometimes we need to access a type passed to us as a parameter:

有時我們需要存取傳遞給我們為參數的類型：

``` kotlin
//存取具體的 Class<T> 類型
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```

Here, we walk up a tree and use reflection to check if a node has a certain type. It’s all fine, but the call site is not very pretty:

在這裡，我們走訪樹的演算法並且使用反射去檢查節點是否有某種類型，一切都很好，但調用場景不是很漂亮：

``` kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

What we actually want is simply pass a type to this function, i.e. call it like this:

我們真正想要的只是傳遞類型給函數，就是調用它像這樣：

``` kotlin
treeNode.findParentOfType<MyTreeNode>()
```

To enable this, inline functions support *reified type parameters*, so we can write something like this:

要啟用此功能，行內置入函數支援 `reified` 類型參數，所以我們可以寫入內容像這樣：

``` kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

We qualified the type parameter with the `reified` modifier, now it’s accessible inside the function, almost as if it were a normal class. Since the function is inlined, no reflection is needed, normal operators like `!is` and `as` are working now. Also, we can call it as mentioned above: `myTree.findParentOfType<MyTreeNodeType>()`.

我們使用 `reified` 修飾符來修飾類型參數，現在它可以在函數內存取，就像是一般的函數。因為函數是行內置入的，不需要反射，一般的運算符 `!is` 和 `as` 像現在一樣工作。另外，我們可以如上所述的調用它： `myTree.findParentOfType<MyTreeNodeType>()`

Though reflection may not be needed in many cases, we can still use it with a reified type parameter:

雖然在很多情況下可能不需要反射，我們還是可以使用 `reified` 類型參數來使用它：

``` kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```

Normal functions (not marked as inline) cannot have reified parameters. A type that does not have a run-time representation (e.g. a non-reified type parameter or a fictitious type like `Nothing`) cannot be used as an argument for a reified type parameter.

一般函數 (沒有標記 `inline`) 不可以有 `reified` 修飾符的參數。沒有運行時表示的類型，不能用作具體化類型參數 (例如：非-具體化類型參數或像 `Nothing` 虛構的類型) 。

For a low-level description, see the [spec document](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md).

有關低階的描述，請參閱[規格文件](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md)。

## Inline properties (since 1.1)

Inline properties (since 1.1) ：行內置入屬性 (從 1.1 版)

The `inline` modifier can be used on accessors of properties that don't have a backing field. You can annotate individual property accessors:

`inline` 修飾符可以在屬性的存取器使用，但不會有支援欄位。你可以註釋私人屬性存取器：

``` kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

You can also annotate an entire property, which marks both of its accessors as inline:

你也可以註釋整個屬性，標記兩個它的存取器 (setter/getter) 為 `inline` ：

``` kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

At the call site, inline accessors are inlined as regular inline functions.

在調用場景，行內置入存取器被置入為常規行內置入函數。

## Restrictions for public API inline functions

Restrictions for public API inline functions ：公開 API 行內置入函數的限制

When an inline function is `public` or `protected` and is not a part of a `private` or `internal` declaration, it is considered a [module](visibility-modifiers.md#modules)'s public API. It can be called in other modules and is inlined at such call sites as well.

當行內置入函數是 `public` 或 `protected` 並且不是 `private` 或 `internal` 宣告的一部分，它被視為[模組](visibility-modifiers.md#modules)的公開 API 。它可以在其他模組調用，並且也在這樣的調用場景置入。

This imposes certain risks of binary incompatibility caused by changes in the module that declares an inline function in case the calling module is not re-compiled after the change.

假使在更改之後未重新編譯調用模組，在模組宣告行內置入函數中，更改會導致二進制不兼容的某些風險。

To eliminate the risk of such incompatibility being introduced by a change in **non**-public API of a module, the public API inline functions are not allowed to use non-public-API declarations, i.e. `private` and `internal` declarations and their parts, in their bodies.

為了消除由模組的非-公開 API 更改導致這樣不兼容的風險，公開 API 行內置入函數不允許在非-公開-API 宣告使用，即是 `private` 和 `internal` 宣告和它們的部份，以及內文中。

An `internal` declaration can be annotated with `@PublishedApi`, which allows its use in public API inline functions. When an `internal` inline function is marked as `@PublishedApi`, its body is checked too, as if it were public.

`internal` 宣告可以使用 `@PublishedApi` 註釋，註釋允許它在公開 API 行內置入函數使用。當 `internal` 行內置入函數標記為 `@PublishedApi` ，也會檢查它的內文，就像它是公開的。

