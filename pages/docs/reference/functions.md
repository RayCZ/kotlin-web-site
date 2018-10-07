---
type: doc
layout: reference
category: "Syntax"
title: "Functions: infix, vararg, tailrec"
---

# Functions

## Function Declarations

Function Declarations ：函數宣告

Functions in Kotlin are declared using the *fun* keyword:

在 Kotlin 中的函數使用 `fun` 關鍵字宣告：

``` kotlin
fun double(x: Int): Int {
    return 2 * x
}
```

## Function Usage

Function Usage ：函數用法

Calling functions uses the traditional approach:

調用函數使用傳統的方式：

``` kotlin
val result = double(2)
```

Calling member functions uses the dot notation:

調用成員函數使用點符號：

``` kotlin
Sample().foo() // create instance of class Sample and call foo
```

### Parameters

Parameters ：參數

Function parameters are defined using Pascal notation, i.e. *name*: *type*. Parameters are separated using commas. Each parameter must be explicitly typed:

函數參數使用 Pascal 表示法定義，即是名稱 : 類型。參數使用逗號分隔。每個參數必須明確指定類型：

``` kotlin
fun powerOf(number: Int, exponent: Int) { ... }
```

### Default Arguments

Default Arguments ：預設參數

Function parameters can have default values, which are used when a corresponding argument is omitted. This allows for a reduced number of overloads compared to other languages:

函數參數可以有預設值，當對應參數被省略時使用預設值。與其他語言比較，這樣可以減少多載的數量。

``` kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) { ... }
```

Default values are defined using the **=** after type along with the value.

在類型之後使用 **=** 與值一起定義預設值。

Overriding methods always use the same default parameter values as the base method. When overriding a method with default parameters values, the default parameter values must be omitted from the signature:

覆寫方法總是使用與基礎方法相同的預設參數值。當使用預設參數值覆寫方法時，必須從簽名省略預設參數值：

``` kotlin
open class A {
    open fun foo(i: Int = 10) { ... }
}
class B : A() {
    override fun foo(i: Int) { ... }  // no default value allowed
}

```

If a default parameter precedes a parameter with no default value, the default value can be used only by calling the function with [named arguments](#named-arguments):

如果預設參數在沒有預設值參數的前面，只能透過使用[已命名的參數](#named-arguments)調用函數來使用預設值：

``` kotlin
fun foo(bar: Int = 0, baz: Int) { ... }

foo(baz = 1) // The default value bar = 0 is used
```

But if a last argument [lambda](lambdas.md#lambda-expression-syntax) is passed to a function call outside the parentheses, passing no values for the default parameters is allowed:

但如果最後一個 [Lambda](lambdas.md#lambda-expression-syntax) 參數在括號外傳遞給函數調用，允許不傳遞預設參數的值。

``` kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { ... }

foo(1) { println("hello") } // Uses the default value baz = 1 
foo { println("hello") }    // Uses both default values bar = 0 and baz = 1
```

### Named Arguments

Named Arguments ：調用已命名的參數

Function parameters can be named when calling functions. This is very convenient when a function has a high number of parameters or default ones.

當調用函數時可以命名函數參數。當函數有大量參數或預設參數時，這非常方便。

Given the following function:

給予以下函數：

``` kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
...
}
```

we could call this using default arguments:

我們可以使用預設參數調用它：

``` kotlin
reformat(str)
```

However, when calling it with non-default, the call would look something like:

然後，當我們使用非預設值調用它時，調用看起來像：

``` kotlin
reformat(str, true, true, false, '_')
```

With named arguments we can make the code much more readable:

使用已命名參數，我們可以使代碼更可讀的：

``` kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

and if we do not need all arguments:

並且如果我們不需要所有參數：

``` kotlin
reformat(str, wordSeparator = '_')
```

When a function is called with both positional and named arguments, all the positional arguments should be placed before the first named one. For example, the call `f(1, y = 2)` is allowed, but `f(x = 1, 2)` is not.

當使用位置和已命名參數調用函數時，所有位置參數應該放在第一個已命名參數之前。例如，允許調用 `f(1, y = 2)` ，但不允許 `f(x = 1, 2)` 。

[Variable number of arguments (*vararg*)](#variable-number-of-arguments-varargs) can be passed in the named form by using the **spread** operator:

透過使用 `spread` 運算符在命名表單中傳遞[可變數量的參數 (`vararg`)](#variable-number-of-arguments-varargs) ：

**`spread` (伸展) 運算符使用星號 `*` ，針對陣列做伸展，用於陣列的傳遞**

``` kotlin
fun foo(vararg strings: String) { ... }

foo(strings = *arrayOf("a", "b", "c"))
```

Note that the named argument syntax cannot be used when calling Java functions, because Java bytecode does not
always preserve names of function parameters.

注意：當調用 Java 函數時不能使用已命名參數語法，因為 Java bytecode 總是不會保留函數參數的名稱。

### Unit-returning functions

Unit-returning functions ：Unit 類型回傳函數

If a function does not return any useful value, its return type is `Unit`. `Unit` is a type with only one value - `Unit`. This value does not have to be returned explicitly:

如果一個函數不會回傳任何有用的值，函數回傳類型是 `Unit` 。 `Unit` 是只有一種值的類型 - `Unit` 。 這個值不須明確的回傳：

``` kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` or `return` is optional
}
```

The `Unit` return type declaration is also optional. The above code is equivalent to:

`Unit` 回傳類型宣告也是可選的。上面的代碼相當於：

``` kotlin
fun printHello(name: String?) { ... }
```

### Single-Expression functions

Single-Expression functions ：單行表達式函數，直接在函數後使用 `=` 給值

When a function returns a single expression, the curly braces can be omitted and the body is specified after a **=** symbol:

當一個函數回傳一個單行表達式，大括號 `{}` 可以被省略並且內文指定在 `=` 符號之後：

``` kotlin
fun double(x: Int): Int = x * 2
```

Explicitly declaring the return type is [optional](#explicit-return-types) when this can be inferred by the compiler:

當內文可以由編譯器推斷時，明確的宣告回傳類型是[可選的](#explicit-return-types) `: Int`：

``` kotlin
fun double(x: Int) = x * 2
```

### Explicit return types

Explicit return types ：明確的回傳類型

Functions with block body must always specify return types explicitly, unless it's intended for them to return `Unit`, [in which case it is optional](#unit-returning-functions). Kotlin does not infer return types for functions with block bodies because such functions may have complex control flow in the body, and the return type will be non-obvious to the reader (and sometimes even for the compiler). 

使用區塊內文的函數 `{...}` 必須明確指定回傳類型，除非它們用於回傳 `Unit` ，[在這種情況下是可選的](#unit-returning-functions)。 Kotlin不會推斷函數區塊內文的回傳類型，因為這些函數在內文可能有複雜的控制流程，並且回傳類型對讀著是不明顯的 (有時對編譯器也是如此) 。

### Variable number of arguments (Varargs)

Variable number of arguments (Varargs) ：可變數量的參數

A parameter of a function (normally the last one) may be marked with `vararg` modifier:

一個函數的參數 (通常致少一個) 可以用 `vararg` 修飾符標記：

``` kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

allowing a variable number of arguments to be passed to the function:

允許可變數量的參數傳遞給函數：

``` kotlin
val list = asList(1, 2, 3)
```

Inside a function a `vararg`-parameter of type `T` is visible as an array of `T`, i.e. the `ts` variable in the example above has type `Array<out T>`.

在函數內類型 `T` 的 `vararg`-參數看作 `T` 陣列，即是在上面範例中的 `ts` 變數是類有類型 `Array<out T>` 。

Only one parameter may be marked as `vararg`. If a `vararg` parameter is not the last one in the list, values for the following parameters can be passed using the named argument syntax, or, if the parameter has a function type, by passing a lambda outside parentheses.

只有一個參數可以被標記為 `vararg` 。如果一個 `vararg` 參數不是在列表中的最後一個參數，可以使用已命名參數語法傳遞以下參數的值，或者，如果參數有函數類型，透過括號 `()` 外 Lambda 傳遞給參數。

When we call a `vararg`-function, we can pass arguments one-by-one, e.g. `asList(1, 2, 3)`, or, if we already have an array and want to pass its contents to the function, we use the **spread** operator (prefix the array with `*`):

當我們調用一個 `vararg`-函數時，我們可以一個接一個傳遞參數，例如： `asList(1, 2, 3)` ，或者，如果我們已經有一個陣列或想要傳遞它的內容到函數，我們使用 `spread` 運算符 (在陣列前前使用 `*`) ：

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

### Infix notation

Infix notation ：中綴符號

Functions marked with the *infix* keyword can also be called using the infix notation (omitting the dot and the parentheses for the call). Infix functions must satisfy the following requirements:

使用 `infix` 關鍵字標記函數也可以使用中綴符號調用 (省略點符號和調用的圓括號) 。中綴函數必須滿足以下需求：

* They must be member functions or [extension functions](extensions.md);
  它們必須提成員函數或[擴展函數](extensions.md)；
* They must have a single parameter;
  它們必須有一個參數；
* The parameter must not [accept variable number of arguments](#variable-number-of-arguments-varargs) and must have no [default value](#default-arguments).
  參數不必[接受可變數量的參數](#variable-number-of-arguments-varargs)且必須沒有[預設值](#default-arguments)。

``` kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

> Infix function calls have lower precedence than the arithmetic operators, type casts, and the `rangeTo` operator.
> The following expressions are equivalent:
> 中綴函數的優先級低於算述運算符、類型強轉、 `rangeTo` 運算符。以下表達式是相等的：
>
> shl 代表位元像左移
>
> * `1 shl 2 + 3` and `1 shl (2 + 3)` ans: `1 shl 5` = `32(10000)` and `1 shl (5)` = `32(10000)`，算述運算符先
> * `0 until n * 2` and `0 until (n * 2)` until不包括最後一個，算述運算符先
> * `xs union ys as Set<*>` and `xs union (ys as Set<*>)` as 強轉運算符先
>
> On the other hand, infix function call's precedence is higher than that of the boolean operators `&&` and `||`, `is`- and `in`-checks, and some other operators. These expressions are equivalent as well:
> 另一方面，中綴函數優先級的調用高於布林運算符 `&&` 和 `||` 、 `is` 、 `in` 檢查和某些其他的運算符。這些表達式也相當於：
>
> * `a && b xor c` and `a && (b xor c)` 中綴運算符 `xor` 先，之後才 `&&`
> * `a xor b in c` and `(a xor b) in c` 中綴運算符 `xor` 先，之後才 `in`
>
> See the [Grammar reference](https://kotlinlang.org/docs/reference/grammar.html#precedence) for the complete operators precedence hierarchy.
>
> 完整運算符優先層級請參閱 [Grammar reference](https://kotlinlang.org/docs/reference/grammar.html#precedence) 。

Note that infix functions always require both the receiver and the parameter to be specified. When you're calling a method on the current receiver using the infix notation, you need to use `this` explicitly; unlike regular method calls, it cannot be omitted. This is required to ensure unambiguous parsing.

注意：中綴函數總是需要指定 receiver 和參數。當你在目前 receiver 使用中綴符號調用一個方法，你需要明確使用 `this` ；不像常規的方法調用， `this` 不可以被省略。這要求是確保清楚的解析。

```kotlin
class MyStringCollection {
    infix fun add(s: String) { ... }

    fun build() {
        this add "abc"   // Correct
        add("abc")       // Correct
        add "abc"        // Incorrect: the receiver must be specified
    }
}
```

## Function Scope

Function Scope ：函數的範圍

In Kotlin functions can be declared at top level in a file, meaning you do not need to create a class to hold a function, which you are required to do in languages such as Java, C# or Scala. In addition to top level functions, Kotlin functions can also be declared local, as member functions and extension functions.

在 Kotlin 的函數可以宣告在檔案內的最高層級，意味著你不需要建立一個新類別去持有一個函數，你需要在  Java 、 C#  或 Scala 等語言執行此操作。除了最高層級的函數，Kotlin 函數也可以宣告區域函數，像成員函數和擴展函數。

### Local Functions

Local Functions ：區域函數

Kotlin supports local functions, i.e. a function inside another function:

Kotlin 支援區域函數，即是一個函數在其他函數內 ` fun dfs(current: Vertex, visited: Set<Vertex>)` ：

``` kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) { //Local Functions
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

Local function can access local variables of outer functions (i.e. the closure), so in the case above, the *visited* can be a local variable:

區域函數可以存取外部函數的區域變數 `visited` (即是閉包) ，所以在上面情況， `visited` 可以是區域變數：

``` kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>() //Local Variable
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

### Member Functions

Member Functions ：成員函數

A member function is a function that is defined inside a class or object:

成員函數是一個在類別或物件內定義的函數：

``` kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

Member functions are called with dot notation:

成員函數使用點符號調用：

``` kotlin
Sample().foo() // creates instance of class Sample and calls foo
```

For more information on classes and overriding members see [Classes](classes.md) and [Inheritance](classes.md#inheritance).

在類別和覆寫成員更多資訊請參閱[類別](classes.md)和[繼承](classes.md#inheritance) 。

## Generic Functions

Generic Functions ：泛型函數

Functions can have generic parameters which are specified using angle brackets before the function name:

函數可以有泛型參數，在函數名稱前使用尖括號 `<T>` 指定泛型參數：

``` kotlin
fun <T> singletonList(item: T): List<T> { ... }
```

For more information on generic functions see [Generics](generics.md).

在泛型函數更多資訊請參閱[泛型](generics.md)。

## Inline Functions

Inline Functions ：內置函數

Inline functions are explained [here](inline-functions.md).

內置函數解譯[在這裡](inline-functions.md)。

## Extension Functions

Extension Functions ：擴展函數

Extension functions are explained in [their own section](extensions.md).

擴展函數解譯在[它們擁有的章節](extensions.md)。

## Higher-Order Functions and Lambdas

Higher-Order Functions and Lambdas ：高階函數和 Lambda

Higher-Order functions and Lambdas are explained in [their own section](lambdas.md).

高階函數和 Lambda 解譯在[它們擁有的章節](lambdas.md)。

## Tail recursive functions

Tail recursive functions ：尾段遞迴函數

Kotlin supports a style of functional programming known as [tail recursion](https://en.wikipedia.org/wiki/Tail_call). This allows some algorithms that would normally be written using loops to instead be written using a recursive function, but without the risk of stack overflow.  When a function is marked with the `tailrec` modifier and meets the required form, the compiler optimises out the recursion, leaving behind a fast and efficient loop based version instead:

Kotlin 支援稱為[尾段遞迴](https://en.wikipedia.org/wiki/Tail_call)的函數式編碼風格。這允許一些通常使用循環編寫的演算法改為使用遞迴函數編寫，但沒有堆疊溢出的風險。當使用 `tailrec` 修飾符標記函數且滿足所需的形式，編譯器優化出遞迴，從而留下基本版本快速和有效的循環：

``` kotlin
val eps = 1E-10 // "good enough", could be 10^-15

tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (Math.abs(x - Math.cos(x)) < eps) x else findFixPoint(Math.cos(x))
```

This code calculates the fixpoint of cosine, which is a mathematical constant. It simply calls Math.cos repeatedly starting at 1.0 until the result doesn't change any more, yielding a result of 0.7390851332151611 for the specified `eps` precision. The resulting code is equivalent to this more traditional style:

此代碼計算餘弦的定點，這是一個數學常數。它只是從 1.0 開始重覆調用 Math.cos，直到結果不再發生任何改變，為 eps精度產生 0.7390851332151611 的結果。生成的代碼相當於這種更傳統的風格：

``` kotlin
val eps = 1E-10 // "good enough", could be 10^-15

private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (Math.abs(x - y) < eps) return x
        x = Math.cos(x)
    }
}
```

To be eligible for the `tailrec` modifier, a function must call itself as the last operation it performs. You cannot use tail recursion when there is more code after the recursive call, and you cannot use it within try/catch/finally blocks. Currently tail recursion is only supported in the JVM backend.

要獲得 `tailrec` 修飾符的資格，函數必須調用自己為它執行的最後一個操作。在遞迴調用之後當有更多代碼，你不可以使用尾段遞迴，並且你不可以使用它在 try/catch/finally 區域內，目前尾段遞迴只在 JVM 後端支援。