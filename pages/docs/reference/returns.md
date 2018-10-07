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

* *return*. By default returns from the nearest enclosing function or [anonymous function](lambdas.md#anonymous-functions).
  `return`。預設情況下，從最近一層封閉函數 `{... return}` 返回或[匿名函數](lambdas.md#anonymous-functions) 。
* *break*. Terminates the nearest enclosing loop.
  `break`。中斷最近一層封閉循環。
* *continue*. Proceeds to the next step of the nearest enclosing loop.
  `continue`。前進最近一層封閉循環下一步。

All of these expressions can be used as part of larger expressions:

所有這些表達式可以用作較大表達式的一部分：

``` kotlin
val s = person.name ?: return
```
The type of these expressions is the [Nothing type](exceptions.md#the-nothing-type).

這些表達式類型是 [Nothing type](exceptions.md#the-nothing-type) 。

## Break and Continue Labels

Break and Continue Labels ：返回和繼續標籤

Any expression in Kotlin may be marked with a *label*. Labels have the form of an identifier followed by the `@` sign, for example: `abc@`, `fooBar@` are valid labels (see the [grammar](https://kotlinlang.org/docs/reference/grammar.html#labelReference)). To label an expression, we just put a label in front of it

在 Kotlin 任何表達式都可能使用 `label` 標記。標籤有識別名稱之後放 `@` 的符號形式，例如： `abc@` 、 `fooBar@` 是有效標籤 (請參閱 [grammar](https://kotlinlang.org/docs/reference/grammar.html#labelReference)) 。要標記表達式，我們只放一個標籤在它前面 `loop@ for (....)`

``` kotlin
loop@ for (i in 1..100) {
    // ...
}
```
Now, we can qualify a *break* or a *continue* with a label:

現在我們可以使用標籤修飾符 `break` 或 `continue` ：
``` kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```
A *break* qualified with a label jumps to the execution point right after the loop marked with that label.A *continue* proceeds to the next iteration of that loop.

使用標籤 `@` 修飾一個 break `break@loop` ，跳轉到 loop 被標記標籤 `@`  `loop@` 之後的執行點。 `continue` 繼續進行該循環的下一個遍歷。


## Return at Labels

Return at Labels：在標籤回傳

With function literals, local functions and object expression, functions can be nested in Kotlin. Qualified *return*s allow us to return from an outer function. The most important use case is returning from a lambda expression. Recall that when we write this:

使用函數文字、區域函數和物件表達式、函數可以被被內嵌在 Kotlin。修飾符 `return` 允許我們回傳從外部函數調用。最重要的使用案例是從 Lambda 表達式中回傳。當我們在這寫 `return` 時被回傳：

``` kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it) //ans:12
    }
    println("this point is unreachable")
}
//sampleEnd

fun main(args: Array<String>) {
    foo()
}
```
---

### Explicit Labels 顯式標籤 

The *return*-expression returns from the nearest enclosing function, i.e. `foo`. (Note that such non-local returns are supported only for lambda expressions passed to [inline functions](inline-functions.md).) If we need to return from a lambda expression, we have to label it and qualify the *return*:

`return-表達式` 從最近一層封閉函數 `{... return}` 回傳，即： `foo` 。 ( 請注意，只對表達式被傳遞到 [inline functions](inline-functions.md) 支援這樣非區域的回傳。 ) ，如果我們需要從表達式返回，我們必須使用標籤標記和修飾符 return ：

**`return@lit` 不會從函數跳出到 main 而是到另一個標籤 `lit@`**

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

//ans:1245 done with explicit label
```
---

### Implicit Labels 隱式標籤

Now, it returns only from the lambda expression. Oftentimes it is more convenient to use implicit labels:
such a label has the same name as the function to which the lambda is passed.

現在，它只從 Lambda 表達式回傳。通常使用隱式標籤更方便：這樣的標籤 `return@forEach` 與 Lambda `forEach{....}` 被傳遞的函數有相同名稱。

**`return@forEach` 不會從函數跳出到 main 而是到函數名稱 `forEach{...}`**

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

//ans:1245 done with implicit label
```
---

### Anonymous Function 匿名函數

Alternatively, we can replace the lambda expression with an [anonymous function](lambdas.md#anonymous-functions).A *return* statement in an anonymous function will return from the anonymous function itself.

或者，我們可以使用匿名函數回取代 Lambda 表達式。在匿名函數中使用 `return` 敘述將從匿名函數當中回傳。

**由於是匿名函數 `forEach(fun(value: Int) {... return ...})` 所以不會整個跳出到 main**

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

//ans:1245 done with anonymous function
```
---

### Return simulated Labels

Note that the use of local returns in previous three examples is similar to the use of *continue* in regular loops. There is no direct equivalent for *break*, but it can be simulated by adding another nesting lambda and non-locally returning from it:

注意：在前三個例子使用區域回傳 `forEach {... return}` 類似於在常規循環中使用 `continue` 。沒有直接等同於 `break` ，但 `break` 可以被模擬，增加另一個內嵌的 Lambda `run{}`和 從內嵌 Lambda 非區域性回傳：

**`return@loop` 傳到 `loop@`，再傳到 `run()`**

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

//ans:12 done with nested loop
```
When returning a value, the parser gives preference to the qualified return, i.e.

當回傳一個值時，解析器優先回傳修飾符 `return` 即

``` kotlin
return@a 1
```
means "return `1` at label `@a`" and not "return a labeled expression `(@a 1)`".

表示 "在標籤 `@a` 回傳 `1`" 而不是 "回傳標籤表達式 `(@a 1)`" 。