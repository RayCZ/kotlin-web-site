---
type: doc
layout: reference
category: "Other"
title: "Destructuring Declarations"
---

# Destructuring Declarations

Sometimes it is convenient to _destructure_ an object into a number of variables, for example:

有時解構一個物件到數個變數是很方便的，例如：

``` kotlin
val (name, age) = person 
```

This syntax is called a _destructuring declaration_. A destructuring declaration creates multiple variables at once. We have declared two new variables: `name` and `age`, and can use them independently:

這個語法稱為解構宣告。解構宣告馬上建立多個變數。我們宣告兩個新的變數： `name` 和 `age` ，並且可以獨立的使用它們：

``` kotlin
println(name)
println(age)
```

A destructuring declaration is compiled down to the following code:

解構宣告被編譯為以下代碼：

``` kotlin
val name = person.component1()
val age = person.component2()
```

The `component1()` and `component2()` functions are another example of the _principle of conventions_ widely used in Kotlin (see operators like `+` and `*`, *for*-loops etc.). Anything can be on the right-hand side of a destructuring declaration, as long as the required number of component functions can be called on it. And, of course, there can be `component3()` and `component4()` and so on.

`component1()` 和 `component2()` 函數在 Kotlin 廣泛使用是慣例原則的另一個例子 (查看運算符像是 `+` 和 `*` ， `for`-loop 等等慣例。) 。任何東西都可以放在解構宣告的右手邊，只要在解構宣告上有所需要的數個元件函數，當然，可以有 `component3()` 和 `component4()` 等等。

Note that the `componentN()` functions need to be marked with the `operator` keyword to allow using them in a destructuring declaration.

注意： `componentN()` 函數需要使用 `operator` 關鍵字標記使解構宣告允許使用它們。

Destructuring declarations also work in *for*-loops: when you say:

解構宣告也用於 `for`-loop ：當你說：

``` kotlin
for ((a, b) in collection) { ... }
```

Variables `a` and `b` get the values returned by `component1()` and `component2()` called on elements of the collection. 

變數 `a` 和 `b` 在集合的元素調用 `component1()` 和 `component2()` 回傳值。

## Example: Returning Two Values from a Function

Example: Returning Two Values from a Function ：例子：從一個函數回傳二個值

Let's say we need to return two things from a function. For example, a result object and a status of some sort. A compact way of doing this in Kotlin is to declare a [_data class_](data-classes.md) and return its instance:

比如說我們需要從函數回傳兩個東西。例如，結果的物件和某排序的狀態。在 Kotlin 做這件事的合併方法是宣告一個 [`data class`](data-classes.md) 和回傳它的實例：

``` kotlin
data class Result(val result: Int, val status: Status)
fun function(...): Result {
    // computations

    return Result(result, status)
}

//解構宣告使用方式，直接回傳函數進行解構
// Now, to use this function:
val (result, status) = function(...)
```

Since data classes automatically declare `componentN()` functions, destructuring declarations work here.

由於 data class 自動的宣告 `componentN()` 函數，因此解構宣告在這裡起作用。

**NOTE**: we could also use the standard class `Pair` and have `function()` return `Pair<Int, Status>`, but it's often better to have your data named properly.  

注意：我們也可以使用標準類別 `Pair` 並有 `function()` 回傳 `Pair<Int, Status>` ，但通常最好將資料正確命名。

## Example: Destructuring Declarations and Maps

Example: Destructuring Declarations and Maps ：例子：解構宣告和 Maps

Probably the nicest way to traverse a map is this:

或許遍歷一個 map 最好的方式是這樣：

``` kotlin
for ((key, value) in map) {
   // do something with the key and the value
}
```

To make this work, we should 

為了這樣工作，我們應該

* present the map as a sequence of values by providing an `iterator()` function;
  透過提供 `iterator()` 函數表示 map 為值的序列；
* present each of the elements as a pair by providing functions `component1()` and `component2()`.
  透過函數 `component1()` 和 `component2()` 表示每個元素為一對。
And indeed, the standard library provides such extensions:

事實上，標準函式庫提功這樣的擴展函數：

``` kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```

So you can freely use destructuring declarations in *for*-loops with maps (as well as collections of data class instances etc).

因此你可以在使用 maps 的 `for`-loop 中自由的使用解構宣告 (以及 data class 實例的集合等等) 。

## Underscore for unused variables (since 1.1)

Underscore for unused variables (since 1.1) ：未使用變數的底線 (從 1.1)

If you don't need a variable in the destructuring declaration, you can place an underscore instead of its name:

如果你在解構宣告中不需要變數，你可以放置一個底線代替它的名稱：

``` kotlin
val (_, status) = getResult()
```

The `componentN()` operator functions are not called for the components that are skipped in this way.

在這樣的方式會跳過元件不會調用 `componentN()` 運算符函數。

## Destructuring in Lambdas (since 1.1)

Destructuring in Lambdas (since 1.1) ： Lambda 中解構 (從 1.1)

You can use the destructuring declarations syntax for lambda parameters. If a lambda has a parameter of the `Pair` type (or `Map.Entry`, or any other type that has the appropriate `componentN` functions), you can introduce several new parameters instead of one by putting them in parentheses:   

你可以對 Lambda 參數使用解構宣告。如果 Lambda 有 `Pair` 類型參數 (或 `Map.Entry` ，或任何其他類型有適當的 `componentN` 函數) ，你可以透過放在括號中引入幾個新參數代替一個：

``` kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" } //引入多個參數
```

Note the difference between declaring two parameters and declaring a destructuring pair instead of a parameter:  

注意：宣告兩個參數和宣告解構對列替代參數的不同：

``` kotlin
{ a -> ... } // one parameter
{ a, b -> ... } // two parameters
{ (a, b) -> ... } // a destructured pair
{ (a, b), c -> ... } // a destructured pair and another parameter
```

If a component of the destructured parameter is unused, you can replace it with the underscore to avoid inventing its name:

如果未使用解構參數的元件，你可以使用底線代碼它來避免造訪名稱：

``` kotlin
map.mapValues { (_, value) -> "$value!" }
```

You can specify the type for the whole destructured parameter or for a specific component separately:

你可以分別指定整個解構參數或特定元件的類型：

``` kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" } //整個參數的類型

map.mapValues { (_, value: String) -> "$value!" } //特定元件的類型
```