---
type: doc
layout: reference
category: "Syntax"
title: "Returns and Jumps: break and continue"
---

# Returns and Jumps

Returns and Jumps ：回傳和區段跳轉

Kotlin has three structural jump expressions:

Kotlin 有三種結構跳轉表示法：

* *return*. By default returns from the nearest enclosing function or [anonymous function](lambdas.html#anonymous-functions).
  `return`。預設情況下，從最近一層封閉函數 `{... return}` 返回或匿名函數
* *break*. Terminates the nearest enclosing loop.
  `break`。中斷最近一層封閉循環
* *continue*. Proceeds to the next step of the nearest enclosing loop.
  `continue`。前進最近一層封閉循環下一步

All of these expressions can be used as part of larger expressions:

所有這些表達式可以用作較大表達式的一部分：

``` kotlin
val s = person.name ?: return
```
The type of these expressions is the [Nothing type](exceptions.html#the-nothing-type).

這些表達式類型是 [Nothing type](exceptions.md#the-nothing-type).

## Break and Continue Labels

Break and Continue Labels ：返回和繼續標籤

Any expression in Kotlin may be marked with a *label*. Labels have the form of an identifier followed by the `@` sign, for example: `abc@`, `fooBar@` are valid labels (see the [grammar](grammar.html#labelReference)). To label an expression, we just put a label in front of it

在 Kotlin 任何表達式都可能使用 `label` 標記，標籤有識別名稱之後放 `@` 的符號形式，例如： `abc@` 、 `fooBar@` 是有效標籤 (請參閱 [grammar for *grammar*](https://kotlinlang.org/docs/reference/grammar.html#labelReference))，要標記表達式，我們只放一個標籤在它前面 `loop@ for (....)`

``` kotlin
loop@ for (i in 1..100) {
    // ...
}
```
Now, we can qualify a *break* or a *continue* with a label:

現在我們可以使用標籤修飾符 `break` 或 `continue`
``` kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```
A *break* qualified with a label jumps to the execution point right after the loop marked with that label.A *continue* proceeds to the next iteration of that loop.

 `break` 修飾符


## Return at Labels

With function literals, local functions and object expression, functions can be nested in Kotlin. 
Qualified *return*{: .keyword }s allow us to return from an outer function. 
The most important use case is returning from a lambda expression. Recall that when we write this:

<div class="sample" markdown="1" theme="idea">

``` kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}
//sampleEnd

fun main(args: Array<String>) {
    foo()
}
```
</div>

The *return*{: .keyword }-expression returns from the nearest enclosing function, i.e. `foo`.
(Note that such non-local returns are supported only for lambda expressions passed to [inline functions](inline-functions.html).)
If we need to return from a lambda expression, we have to label it and qualify the *return*{: .keyword }:

<div class="sample" markdown="1" theme="idea">

``` kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with explicit label")
}
//sampleEnd

fun main(args: Array<String>) {
    foo()
}
```
</div>

Now, it returns only from the lambda expression. Oftentimes it is more convenient to use implicit labels:
such a label has the same name as the function to which the lambda is passed.

<div class="sample" markdown="1" theme="idea">

``` kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with implicit label")
}
//sampleEnd

fun main(args: Array<String>) {
    foo()
}
```
</div>

Alternatively, we can replace the lambda expression with an [anonymous function](lambdas.html#anonymous-functions).
A *return*{: .keyword } statement in an anonymous function will return from the anonymous function itself.

<div class="sample" markdown="1" theme="idea">

``` kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}
//sampleEnd

fun main(args: Array<String>) {
    foo()
}
```
</div>

Note that the use of local returns in previous three examples is similar to the use of *continue*{: .keyword } in regular loops. There is no direct equivalent for *break*{: .keyword }, but it can be simulated by adding another nesting lambda and non-locally returning from it:

<div class="sample" markdown="1" theme="idea">

``` kotlin
//sampleStart
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}
//sampleEnd

fun main(args: Array<String>) {
    foo()
}
```
</div>

When returning a value, the parser gives preference to the qualified return, i.e.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
return@a 1
```
</div>

means "return `1` at label `@a`" and not "return a labeled expression `(@a 1)`".
