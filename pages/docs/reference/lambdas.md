---
type: doc
layout: reference
category: "Syntax"
title: "Higher-Order Functions and Lambdas"
---

# Higher-Order Functions and Lambdas

Higher-Order Functions and Lambdas ：高階函數和 Lambda 表達式

Kotlin functions are [*first-class*](https://en.wikipedia.org/wiki/First-class_function), which means that they can be stored in variables and data structures, passed as arguments to and returned from other [higher-order functions](#higher-order-functions). You can operate with functions in any way that is possible for other non-function values. 

Kotlin 函數是頭等函數，意味著它們可以儲存在變數和資料結構中，作為參數傳遞和從其他的[高階函數](#higher-order-functions)回傳。針對其他非函數值你可在任何情況下操作函數。

To facilitate this, Kotlin, as a statically typed programming language, uses a family of [function types](#function-types) to represent functions and provides a set of specialized language constructs, such as [lambda expressions](#lambda-expressions-and-anonymous-functions).

為了方便這點， Kotlin ，為靜態類型程式語言，使用一系列的函數類型來表達函數並提供一組專門語言結構，例如：[Lambda 表達式](#lambda-expressions-and-anonymous-functions)。

## Higher-Order Functions

Higher-Order Functions ：高階函數

A higher-order function is a function that takes functions as parameters, or returns a function.

高階函數是一種函數，可帶入函數為參數，或回傳一個函數。

A good example is the [functional programming idiom `fold`](https://en.wikipedia.org/wiki/Fold_(higher-order_function)) for collections, which takes an initial accumulator value and a combining function and builds its return value by consecutively combining current accumulator value with each collection element, replacing the accumulator:

一個很好的例子是 collections 的[函數程式慣語 `fold`](https://en.wikipedia.org/wiki/Fold_(higher-order_function)) ，`fold`函數帶入一個初始化累計器的值和一個結合函數並建立它的回傳值，回傳值透過每個集合元素連續結合目前累計值 (疊加) ，取代累計值。

``` kotlin
fun <T, R> Collection<T>.fold(
    initial: R, 
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

In the code above, the parameter `combine` has a [function type](#function-types) `(R, T) -> R`, so it accepts a function that takes two arguments of types `R` and `T` and returns a value of type `R`. It is [invoked](#invoking-a-function-type-instance) inside the *for*-loop, and the return value is then assigned to `accumulator`.

在上面的代碼中，參數 `combine` 有一個[函數類型](#function-types) `(R, T) -> R` ，因此它接受一個函數，函數帶有類型 `R` 和 `T` 的兩個參數並回傳一個 `R` 類型的值。在 for-循環內調用函數類型 `combine` ，接著分配給 `accumulator` 疊加當回傳值。

To call `fold`, we need to pass it an [instance of the function type](#instantiating-a-function-type) as an argument, and lambda expressions ([described in more detail below](#lambda-expressions-and-anonymous-functions)) are widely used for this purpose at higher-order function call sites:

要調用 `fold` ，我們需要傳給它一個[函數類型的實例](#instantiating-a-function-type)為參數，並且 Lambda 表達式 ([更詳細的描述在下面](#lambda-expressions-and-anonymous-functions)) 在高階函數調用場景中被廣泛用於此目的：

```kotlin
fun main(args: Array<String>) {
    //sampleStart
    val items = listOf(1, 2, 3, 4, 5)
    
    // Lambdas are code blocks enclosed in curly braces.
    items.fold(0, { 
        // When a lambda has parameters, they go first, followed by '->'
        acc: Int, i: Int -> 
        print("acc = $acc, i = $i, ") 
        val result = acc + i
        println("result = $result")
        // The last expression in a lambda is considered the return value:
        result
    })
    
    // Parameter types in a lambda are optional if they can be inferred:
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })
    
    // Function references can also be used for higher-order function calls:
    val product = items.fold(1, Int::times)
    //sampleEnd
    println("joinedToString = $joinedToString")
    println("product = $product")
}
//ans:
//acc = 0, i = 1, result = 1
//acc = 1, i = 2, result = 3
//acc = 3, i = 3, result = 6
//acc = 6, i = 4, result = 10
//acc = 10, i = 5, result = 15
//joinedToString = Elements: 1 2 3 4 5
//product = 120
```

The following sections explain in more detail the concepts mentioned so far.

以下章節更詳細的解釋到目前為止提到的概念。

## Function types

Function types ：函數類型

Kotlin uses a family of function types like `(Int) -> String` for declarations that deal with functions: `val onClick: () -> Unit = ...`.

Kotlin 使用一系列的函數類型，像是 `(Int) -> String` 處理函數宣告： `val onClick: () -> Unit = ...` 。

These types have a special notation that corresponds to the signatures of the functions, i.e. their parameters and return values:

這些類型有一個特殊符號對應到函數的簽名 `(para: Type) -> Unit` ，即是它們的參數和回傳：

* All function types have a parenthesized parameter types list and a return type: `(A, B) -> C` denotes a type that represents functions taking two arguments of types `A` and `B` and returning a value of type `C`. The parameter types list may be empty, as in `() -> A`. The [`Unit` return type](functions.md#unit-returning-functions) cannot be omitted. 
  所有函數類型都有一個帶括號的參數類型列表和一個回傳類型： `(A, B) -> C` 表示一個類型，代表函數帶有類型 `A` 和 `B` 兩個參數和回傳類型 `C` 的一個值。參數類型列表可以為空，如 `() -> A`。 [`Unit` 回傳類型](functions.md#unit-returning-functions) 不可以省略。

* Function types can optionally have an additional *receiver* type, which is specified before a dot in the notation: the type `A.(B) -> C` represents functions that can be called on a receiver object of `A` with a parameter of `B` and return a value of `C`. [Function literals with receiver](#function-literals-with-receiver) are often used along with these types.
  函數類型可以選擇有一個額外的 `receiver` 類型，它在符號中逗點前指定：類型 `A.(B) -> C` 代表函數可以在 `A` 的 `receiver` 物件調用和 `B` 的參數並回傳 `C` 的值。 [使用 receiver 的函數文字](#function-literals-with-receiver)通常與這些類型一起使用。

* [Suspending functions](coroutines.md#suspending-functions) belong to function types of a special kind, which have a *suspend* modifier in the notation, such as `suspend () -> Unit` or `suspend A.(B) -> C`.

  [縣掛式函數](coroutines.md#suspending-functions) 屬於特殊種類的函數類型。它在符號中有一個 `suspend` 修飾符，例如 `suspend () -> Unit` 或 `suspend A.(B) -> C` 。

The function type notation can optionally include names for the function parameters: `(x: Int, y: Int) -> Point`. These names can be used for documenting the meaning of the parameters.

函數類型符號可以選擇包括函數參數的名稱： `(x: Int, y: Int) -> Point` 。這些名稱可以用於記錄參數的意義。

> To specify that a function type is [nullable](null-safety.md#nullable-types-and-non-null-types), use parentheses: `((Int, Int) -> Int)?`.
>
> 指定函數類型是 [nullable](null-safety.md#nullable-types-and-non-null-types) ，使用括號： `((Int, Int) -> Int)?` 。把整個函數括起來並使用 `?` 符號。
>
> Function types can be combined using parentheses: `(Int) -> ((Int) -> Unit)`
>
> 函數類型使用括號結合： `(Int) -> ((Int) -> Unit)` ，參數 `(int)`，符號 `->` ，回傳 `((Int) -> Unit)`
>
> The arrow notation is right-associative, `(Int) -> (Int) -> Unit` is equivalent to the previous example, but not to `((Int) -> (Int)) -> Unit`.
>
> 箭頭符號是右關聯的， `(Int) -> (Int) -> Unit` 等同前一個範例，但不是 `((Int) -> (Int)) -> Unit` 。

You can also give a function type an alternative name by using [a type alias](type-aliases.md):

你也可以透過一個[類型別名](type-aliases.md)給一個函數類型備用名稱 `typealias ClickHandler`：

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

---

### Instantiating a function type

Instantiating a function type ：實例化一個函數類型

There are several ways to obtain an instance of a function type:

有幾種方式去獲取函數類型的實例：

* Using a code block within a function literal, in one of the forms: 
    在一個函數文字內使用一個代碼，使用其中一種形式：
    * a [lambda expression](#lambda-expressions-and-anonymous-functions): `{ a, b -> a + b }`,
      一個 [lambda 表達式](#lambda-expressions-and-anonymous-functions)： `{ a, b -> a + b }` ，
    * an [anonymous function](#anonymous-functions): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`
      一個[匿名函數](#anonymous-functions)： `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

   [Function literals with receiver](#function-literals-with-receiver) can be used as values of function types with receiver.

   [使用 receiver 的函數文字](#function-literals-with-receiver)可以用作 `receiver` 函數類型的值。

* Using a callable reference to an existing declaration:
    使用一個可調用的參照引用到已存在的宣告：
    * a top-level, local, member, or extension [function](reflection.md#function-references): `::isOdd`, `String::toInt`,
      一個高階、區域、成員、或擴展[函數](reflection.md#function-references) ： `::isOdd` ， `String::toInt` ，
    * a top-level, member, or extension [property](reflection.md#property-references): `List<Int>::size`,
      一個高階、成員、或擴展[屬性](reflection.md#property-references)： `List<Int>::size` ，
    * a [constructor](reflection.md#constructor-references): `::Regex`
      一個[建構元](reflection.md#constructor-references)： `::Regex`

   These include [bound callable references](reflection.md#bound-function-and-property-references-since-11) that point to a member of a particular instance: `foo::toString`.
這些包括[受約束可調用的參照](reflection.md#bound-function-and-property-references-since-11)指向一個特定實例的成員： `foo::toString`。

* Using instances of a custom class that implements a function type as an interface: 
    使函數類型為介面的自定義類別實例 `: (Int) -> Int` ：

```kotlin
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer() 
```

The compiler can infer the function types for variables if there is enough information:

如果有足夠的資訊，編譯器可以推斷變數的函數類型：

```kotlin
val a = { i: Int -> i + 1 } // The inferred type is (Int) -> Int
```

*Non-literal* values of function types with and without receiver are interchangeable, so that the receiver can stand in for the first parameter, and vice versa. For instance, a value of type `(A, B) -> C` can be passed or assigned where a `A.(B) -> C` is expected and the other way around:

有和沒有 `receiver` 函數類型的**非文字**值是可以互換的，這樣 `receiver` 可以位於第一個參數，反過來也是。例如：預期是 `A.(B) -> C` 函數類型可以傳遞或分配到類型 `(A, B) -> C` 的值並且反過來也可以：

``` kotlin
fun main(args: Array<String>) {
    //sampleStart
    
    //A.(B) -> C
    val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
    
    //(A,B) -> C 函數類型 接收 A.(B) - > C 函數類型
    val twoParameters: (String, Int) -> String = repeatFun // OK
    
    fun runTransformation(f: (String, Int) -> String): String {
        return f("hello", 3)
    }
    val result = runTransformation(repeatFun) // OK
    //sampleEnd
    println("result = $result")
}
//ans: result = hellohellohello
```

> Note that a function type with no receiver is inferred by default, even if a variable is initialized with a reference
> to an extension function. To alter that, specify the variable type explicitly.
>
> 注意：透過預設情況下, 會推斷沒有 `receiver` 的函數類型，即使使用參照引用到擴展函數來初始化變數。去改變它，請明確指定變數類型。

---

### Invoking a function type instance  

Invoking a function type instance  ：調用函數類型實例

A value of a function type can be invoked by using its [`invoke(...)` operator](operator-overloading.md#invoke): `f.invoke(x)` or just `f(x)`.

透過使用函數的 [`invoke(...)` 運算符](operator-overloading.md#invoke) 調用函數類型的值： `f.invoke(x)` 或只用 `f(x)` 。

If the value has a receiver type, the receiver object should be passed as the first argument. Another way to invoke a value of a function type with receiver is to prepend it with the receiver object, as if the value were an [extension function](extensions.md): `1.foo(2)`,

如果值有 `receiver` 類型， `receiver` 物件應該作為第一個參數傳遞，另一種方式調用 `receiver` 函數類型的值是使用 `receiver` 物件為前置，好像值是[擴展函數](extensions.md)： `1.foo(2)` ，

Example:

``` kotlin
fun main(args: Array<String>) {
    //sampleStart
    
    //宣告函數類型到變數
    val stringPlus: (String, String) -> String = String::plus
    
    //幾種調用函數類型的方式
    println(stringPlus.invoke("<-", "->"))//表示法：函數類型實例.invoke(...)
    println(stringPlus("Hello, ", "world!")) //表示法：函數類型實例(...)
    
    //建立類似的擴展函數
    val stringPlusExtension: String.(String) -> String = String::plus
    println("Hello, ".stringPlusExtension("world!")) //表示法：receiver.函數類型實例(...)
    //println(stringPlusExtension.invoke("<-", "->")) 也通用
    //println(stringPlusExtension("Hello, ", "world!")) 也通用
    
    //宣告函數類型到變數，多了擴展函數
    val intPlus: Int.(Int) -> Int = Int::plus

    //幾種調用函數類型的方式
    println(intPlus.invoke(1, 1)) //表示法：函數類型實例.invoke(...)
    println(intPlus(1, 2)) //上個章節兩種表示法通用，表示法：函數類型實例(...)
    println(2.intPlus(3)) // extension-like call ，表示法：receiver.函數類型實例(...)
    //sampleEnd
}

//ans:
//<-->
//Hello, world!
//Hello, world!
//2
//3
//5
```

---

### Inline functions

Inline functions ：行內置入函數

Sometimes it is beneficial to use [inline functions](inline-functions.md), which provide flexible control flow, for higher-order functions.

有時使用[行內置入函數](inline-functions.md)是有好處的，行內置入函數為高階函數提供靈活的控制流程。

## Lambda Expressions and Anonymous Functions

Lambda Expressions and Anonymous Functions ： Lambda 表達式和匿名函數

Lambda expressions and anonymous functions are 'function literals', i.e. functions that are not declared,
but passed immediately as an expression. Consider the following example:

Lambda 表達式和匿名函數是 "函數文字" ，即是未宣告的函數，而立即作為表達式傳遞。請考慮以下範例：

``` kotlin
max(strings, { a, b -> a.length < b.length })
```

Function `max` is a higher-order function, it takes a function value as the second argument. This second argument is an expression that is itself a function, i.e. a function literal, which is equivalent to the following named function:

函數 `max` 是一個高階函數，它帶入函數值為第二參數。第二個參數是一個表達式，表達式本身是一個函數，即是函數文字，函數文字相當以於以下已命名的函數：

``` kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

---

### Lambda expression syntax

Lambda expression syntax ： Lambda 表達式語法

The full syntactic form of lambda expressions is as follows:

從 Lambda 表達式完整的語法為以下：

``` kotlin
val sum = { x: Int, y: Int -> x + y }
```

A lambda expression is always surrounded by curly braces, parameter declarations in the full syntactic form go inside curly braces and have optional type annotations, the body goes after an `->` sign. If the inferred return type of the lambda is not `Unit`, the last (or possibly single) expression inside the lambda body is treated as the return value.

Lambda 表達式總是透過大括號圍繞，完整語法形式的參數宣告進入大括號內並有可選的類型註釋，在 `->` 符號之後為程式的內文。如果 Lambda 回傳類型推斷不是 `Unit` ，最後 (或可能單行) 表達式在 Lambda 內文被視為回傳值。

If we leave all the optional annotations out, what's left looks like this:

如果我們省去所有可選的註釋，留下來代碼的看起來像這樣：

``` kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

**比較兩段程式的不同：**

``` kotlin
//1.參數類型 (參數宣告) ：在大括號內宣告參數並指定類型 { x: Int, y: Int -> ... }
//2.表達式內文：在 -> 之後 { ... -> x + y }
//3.回傳值：依最後一行程式決定 { ... -> x + y }
val sum = { x: Int, y: Int -> x + y }

//1.參數類型 (參數宣告) ：在宣告變數時指定函數類型 val sum: (Int, Int) -> Int = ...
//2.表達式內文：在 -> 之後 { ... -> x + y }
//3.回傳值：必須為 Int 類型的回傳值 val sum: (Int, Int) -> Int = ...
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

---

### Passing a lambda to the last parameter

Passing a lambda to the last parameter ：傳遞 Lambda 表達式到最後一個參數

In Kotlin, there is a convention that if the last parameter of a function accepts a function, a lambda expression that is passed as the corresponding argument can be placed outside the parentheses:

在 Kotlin 中，有一個慣用語法，如果函數的最後參數接受一個函數，Lambda 表達式傳遞為對應的參數可以被放置在括號外：

**fold 的第一個參數是 (1) ，第二個參數是 { acc, e -> acc * e } ，所以可以在小括號外使用 Lambda 表達式**

``` kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

If the lambda is the only argument to that call, the parentheses can be omitted entirely: 

如果 Lambda 表達式是唯一調用的參數，可以完全省略括號：

``` kotlin
run { println("...") }
```

---

### `it`: implicit name of a single parameter

`it`: implicit name of a single parameter ： `it`: 單一參數的隱性名稱

It's very common that a lambda expression has only one parameter.

Lambda 表達式只有一個參數是很常見的。

If the compiler can figure the signature out itself, it is allowed not to declare the only parameter and omit `->`. The parameter will be implicitly declared under the name `it`:

如果編譯器可以計算出它本身的簽名，允許不宣告唯一的參數並省略 `->` 。參數將以 `it` 隱性宣告命名：

``` kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

---

### Returning a value from a lambda expression

Returning a value from a lambda expression ：從 Lambda 表達式回傳一個值

We can explicitly return a value from the lambda using the [qualified return](returns.md#return-at-labels) syntax. Otherwise, the value of the last expression is implicitly returned. 

我們可以使用[修飾符 return](returns.md#return-at-labels) 語法從 Lambda 回傳一個值。否則，隱性回傳最後一行表達式的值。

Therefore, the two following snippets are equivalent:

因此，以下兩個片段是相等的：

``` kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter //回傳值
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter //回傳值
}
```

This convention, along with [passing a lambda expression outside parentheses](#passing-a-lambda-to-the-last-parameter), allows for [LINQ-style](http://msdn.microsoft.com/en-us/library/bb308959.aspx) code:

這是慣用語法，以及[在括號外傳遞 Lambda 表達式](#passing-a-lambda-to-the-last-parameter)，允許 [LINQ-style](http://msdn.microsoft.com/en-us/library/bb308959.aspx) 的代碼：

``` kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

---

### Underscore for unused variables (since 1.1)

Underscore for unused variables (since 1.1) ：底線用於未使用的變數 (從 1.1 版)

If the lambda parameter is unused, you can place an underscore instead of its name:

如果 Lambda 參數是未使用的，你可以放置一個底線代替它的名稱：

``` kotlin
map.forEach { _, value -> println("$value!") }
```

---

### Destructuring in lambdas (since 1.1)

Destructuring in lambdas (since 1.1) ：在 Lambda 的解構 (從 1.1 版)

Destructuring in lambdas is described as a part of [destructuring declarations](multi-declarations.md#destructuring-in-lambdas-since-11).

在 Lambda的解構描述在解構宣告的一部份。

---

### Anonymous functions

Anonymous functions ：匿名函數

One thing missing from the lambda expression syntax presented above is the ability to specify the return type of the function. In most cases, this is unnecessary because the return type can be inferred automatically. However, if you do need to specify it explicitly, you can use an alternative syntax: an _anonymous function_.

從上面提到的 Lambda 表達式語法缺少一件事是指定函數回傳類型的能力。大多情況下，這是不必要的，因為超以自動地推斷回傳類型。然而，如果你需要明確指定它，你可以使用替代語法：匿名函數。

``` kotlin
fun(x: Int, y: Int): Int = x + y
```

An anonymous function looks very much like a regular function declaration, except that its name is omitted. Its body can be either an expression (as shown above) or a block:

匿名函數看起來非常像常規的函數宣告，只是省略了它的名稱。它的內文可以是表達式 (如上所述) 或一個區塊：

``` kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

The parameters and the return type are specified in the same way as for regular functions, except that the parameter types can be omitted if they can be inferred from context:

參數和回傳類型指定與常規函數相同，如果它們可以從程式的內容推斷，則可以省略參數類型：

``` kotlin
ints.filter(fun(item) = item > 0)
```

The return type inference for anonymous functions works just like for normal functions: the return type is inferred automatically for anonymous functions with an expression body and has to be specified explicitly (or is assumed to be `Unit`) for anonymous functions with a block body.

匿名函數的回傳類型推斷工作就像一般的函數：匿名函數的表達式內文自動地推斷回傳類型並且匿名函數的區塊內文必須明確指定 (或假設為 `Unit`) 。

Note that anonymous function parameters are always passed inside the parentheses. The shorthand syntax allowing to leave the function outside the parentheses works only for lambda expressions.

注意：匿名函數參數總是在括號內傳遞。允許在括號外留下函數的簡短語法只用於 Lambda 表達式。

One other difference between lambda expressions and anonymous functions is the behavior of [non-local returns](inline-functions.md#non-local-returns). A *return* statement without a label always returns from the function declared with the *fun* keyword. This means that a *return* inside a lambda expression will return from the enclosing function, whereas a *return* inside an anonymous function will return from the anonymous function itself.

Lambda 表達式和函名函數之間的一個不同是[非區域回傳](inline-functions.md#non-local-returns)的行為。沒有標籤 `@xxx` 的 `return` 敘述總是從使用 `fun` 關鍵字宣告的函數回傳。這意味著在 Lambda 表達式回傳將從閉包函數回傳，而在匿名函數內的 `return` 將從匿名函數本身回傳。

**Lambda 表達法的 `return` ，如果沒有使用標籤 `@xxx` 來指定跳出的地方會直接從整個函數回傳，而匿名函數不會**

---

### Closures

Closures ：閉包

A lambda expression or anonymous function (as well as a [local function](functions.md#local-functions) and an [object expression](object-declarations.md#object-expressions)) can access its _closure_, i.e. the variables declared in the outer scope. Unlike Java, the variables captured in the closure can be modified:

Lambda 表達式或匿名函數 (以及[區域函數](functions.md#local-functions)和[物件表達式](object-declarations.md#object-expressions)) 可以存取它的閉包，即是在外部範圍宣告的變數。不像 Java ，可以修改在閉包中捕獲的變數：

``` kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

---

### Function literals with receiver

Function literals with receiver ：使用 `receiver` 的函數文字

[Function types](#function-types) with receiver, such as `A.(B) -> C`, can be instantiated with a special form of function literals – function literals with receiver.

使用 `receiver` 的[函數類型](#function-types) ，例如： `A.(B) -> C` ，可以使用函數文字的特定形式初始化 - 使用 `receiver` 函數文字

As said above, Kotlin provides the ability [to call an instance](#invoking-a-function-type-instance) of a function type with receiver providing the _receiver object_.

如上所述， Kotlin 提供使用 `receiver` 提供 `receiver` 物件來[調用實例](#invoking-a-function-type-instance)函數類型實例的能力。

Inside the body of the function literal, the receiver object passed to a call becomes an *implicit* *this*, so that you can access the members of that receiver object without any additional qualifiers, or access the receiver object using a [`this` expression](this-expressions.md).

在函數文字的內文中， 傳遞給調用的 `receiver` 物件變成隱式的 `this` ，因此你可以沒有額外的修飾符存取 `receiver` 物件的成員，或存取 `receiver` 物件使用 [`this` 表達式](this-expressions.md) 。

This behavior is similar to [extension functions](extensions.md), which also allow you to access the members of the receiver object inside the body of the function.

這個行為類似於[擴展函數](extensions.md) ，也允許你在函數的內文中存取 `receiver` 的成員。

Here is an example of a function literal with receiver along with its type, where `plus` is called on the receiver object:

下面是一個使用 `receiver` 與它的類型一起的函數文字範例，當中 `plus` 在 `receiver` 物件被調用：

``` kotlin
//在 Kotlin 的 Int 類別已提供 plus 方法
//以下可以看成 { other -> this.plus(other) } ， receiver 就是 int 實例本身 this 
val sum: Int.(Int) -> Int = { other -> plus(other) } 

//實際調用： 200.sum(100)
//在函數內顯示 { other(代表參數 100) -> 200.plus(other) } 
//ans: 300
```

The anonymous function syntax allows you to specify the receiver type of a function literal directly. This can be useful if you need to declare a variable of a function type with receiver, and to use it later.

匿名函數語法允許你直接指定函式文字的 `receiver` 類型。如果你需要宣告一個使用 `receiver` 函數類型的變數，這將非常有用。

``` kotlin
//匿名函數以fun開頭 fun Int.(other: Int): Int
val sum = fun Int.(other: Int): Int = this + other

//實際調用： 200.sum(100)
//在函數內顯示 this(200) + other(100)
//ans: 300
```

Lambda expressions can be used as function literals with receiver when the receiver type can be inferred from context. One of the most important examples of their usage is [type-safe builders](type-safe-builders.md):

當 `receiver` 類型可以從代碼的內容中推斷， Lambda 表達式可以用作使用 `receiver` 的函數文字。它們的用法最重要的一個例子是[類型安全的建構者](type-safe-builders.md)：

``` kotlin
class HTML {
    fun body() { ... }
}

//init 是 receiver 類型也是一個函數類型
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // create the receiver object
    html.init()        // pass the receiver object to the lambda
    return html
}

//傳遞給 html() 一個函數類型 (小括號可以省略)
//在 {...} 中有隱性的 this 代表 html 這個實例
html {       // lambda with receiver begins here
    body()   // calling a method on the receiver object
}
```