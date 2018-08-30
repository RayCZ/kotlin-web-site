---
type: doc
layout: reference
category: "Basics"
title: "Basic Syntax"
---

# Basic Syntax

## Defining packages

**Package：在 Java 或 Kotlin 做檔案結構的分類或是打包**

Package specification should be at the top of the source file:

Package 規範應該在來源檔案的最上面：

``` kotlin
package my.demo

import java.util.*

// ...
```
It is not required to match directories and packages: source files can be placed arbitrarily in the file system.

package 需不要匹配目錄與 package ：在檔案系統中來源檔案可以隨意的放置

See [Packages](packages.md).

## Defining functions (定義函數)

Function having two `Int` parameters with `Int` return type:

以下函數有兩個 `Int` 參數與 `Int` 回傳類型：

``` kotlin
//sampleStart
fun sum(a: Int, b: Int): Int {
    return a + b
}
//sampleEnd

fun main(args: Array<String>) {
    print("sum of 3 and 5 is ")
    println(sum(3, 5))//ans:sum of 3 and 5 is 8
}
```

----

Function with an expression body and inferred return type:

使用表達式 (Lambda) 內文的函數或推斷回傳類型：

``` kotlin
//sampleStart
fun sum(a: Int, b: Int) = a + b
//sampleEnd

fun main(args: Array<String>) {
    println("sum of 19 and 23 is ${sum(19, 23)}")//ans:sum of 19 and 23 is 42
}
```
----

Function returning no meaningful value:

函數回傳無意義的值 (Unit) ：

``` kotlin
//sampleStart
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main(args: Array<String>) {
    printSum(-1, 8)//ans:sum of -1 and 8 is 7
}
```
----

`Unit` return type can be omitted:

`Unit` 回傳類型可以被省略 (與上面比較Unit被省略) ：

``` kotlin
//sampleStart
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main(args: Array<String>) {
    printSum(-1, 8)//ans:sum of -1 and 8 is 7
}
```
See [Functions](functions.md).

## Defining variables (定義變數)

Assign-once (read-only) local variable:

分配一次性 (唯讀) 區域變數 (val) ：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val a: Int = 1  // immediate assignment
    val b = 2   // `Int` type is inferred
    val c: Int  // Type required when no initializer is provided
    c = 3       // deferred assignment
//sampleEnd
    println("a = $a, b = $b, c = $c")//ans:a = 1, b = 2, c = 3
}
```

---

Mutable variable:

可變的變數 (var) ：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    var x = 5 // `Int` type is inferred
    x += 1
//sampleEnd
    println("x = $x")//ans:x = 6
}
```
---

Top-level variables:

最高層級的變數 (不在類別或是函數內) ：

``` kotlin
//sampleStart
val PI = 3.14
var x = 0

fun incrementX() { 
    x += 1 
}
//sampleEnd

fun main(args: Array<String>) {
    println("x = $x; PI = $PI")//ans:x = 0; PI = 3.14
    incrementX()
    println("incrementX()")//ans:incrementX()
    println("x = $x; PI = $PI")//ans:x = 1; PI = 3.14
}
```
See also [Properties And Fields](properties.md).


## Comments (註解)

Just like Java and JavaScript, Kotlin supports end-of-line and block comments.

就像 Java 或 JavaScript， Kotlin 支援行尾和區塊註解

``` kotlin
// This is an end-of-line comment

/* This is a block comment
   on multiple lines. */
```

Unlike Java, block comments in Kotlin can be nested.

不像 Java，在 Kotlin 區塊註解可以被內嵌

See [Documenting Kotlin Code](kotlin-doc.md) for information on the documentation comment syntax.

有關文件註解語法，請看 [Documenting Kotlin Code](kotlin-doc.md) 

## Using string templates (字串模版)

**String templates：利用 `$` 或 `${}` 符號在字串內 `""` 使用變數或程式區塊**

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    var a = 1
    // simple name in template:
    val s1 = "a is $a" 
    
    a = 2
    // arbitrary expression in template:
    val s2 = "${s1.replace("is", "was")}, but now is $a"
//sampleEnd
    println(s2)//ans:a was 1, but now is 2
}
```
See [String templates](basic-types.html#string-templates).

## Using conditional expressions (條件表達式)


**Conditional expressions：條件表達式，if、else**

``` kotlin
//sampleStart
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
//sampleEnd

fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")//ans:max of 0 and 42 is 42
}
```
Using `if` as an expression:

使用 `if` 為一個單行表達式 (Lambda) ，差別在於方法可以用 `=` 指定表達式

``` kotlin
//sampleStart
fun maxOf(a: Int, b: Int) = if (a > b) a else b
//sampleEnd

fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")//ans:max of 0 and 42 is 42
}
```
See [*if*-expressions](control-flow.md#if-expression).

## Using nullable values and checking for `null` (使用可空的變數和null檢查)

A reference must be explicitly marked as nullable when `null` value is possible.

當可能為 `null` 值時，一個參照必須明確標記為可空的

Return `null` if `str` does not hold an integer:

如果 `str` 不為整數回傳 `null` (Int?)：

``` kotlin
fun parseInt(str: String): Int? {
    // ...
}
```
Use a function returning nullable value:

使用函數回傳可空的值：

``` kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

//sampleStart
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // Using `x * y` yields error because they may hold nulls.
    if (x != null && y != null) {
        // x and y are automatically cast to non-nullable after null check
        println(x * y)
    }
    else {
        println("either '$arg1' or '$arg2' is not a number")
    }    
}
//sampleEnd


fun main(args: Array<String>) {
    printProduct("6", "7")//ans:42
    printProduct("a", "7")//ans:either 'a' or '7' is not a number
    printProduct("a", "b")//ans:either 'a' or 'b' is not a number
}
```
or

``` kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)
    
//sampleStart
    // ...
    if (x == null) {
        println("Wrong number format in arg1: '$arg1'")
        return
    }
    if (y == null) {
        println("Wrong number format in arg2: '$arg2'")
        return
    }

    // x and y are automatically cast to non-nullable after null check
    println(x * y)
//sampleEnd
}

fun main(args: Array<String>) {
    printProduct("6", "7")//ans:42
    printProduct("a", "7")//ans:Wrong number format in arg1: 'a'
    printProduct("99", "b")//ans:Wrong number format in arg2: 'b'
}
```
See [Null-safety](null-safety.md).

## Using type checks and automatic casts (使用類型檢查與自動強轉)

The `is` operator checks if an expression is an instance of a type.
If an immutable local variable or property is checked for a specific type, there's no need to cast it explicitly:

`is` 運算符檢查是否為類型的實例，如果一個不可變的區域變數或屬或被檢查為特定類型，沒有明確強轉的必要：


**以下例子：```if (obj is String)``` 已經先檢查過自動強轉，所以在 ```{ return obj.length }``` 區塊內不再使用強轉 `as`**

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` is automatically cast to `String` in this branch
        return obj.length
    }

    // `obj` is still of type `Any` outside of the type-checked branch
    return null
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")//ans:'Incomprehensibilities' string length is 21 
    printLength(1000)//ans:'1000' string length is ... err, not a string 
    printLength(listOf(Any()))//ans:'[java.lang.Object@3af49f1c]' string length is ... err, not a string 
}
```
or

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` is automatically cast to `String` in this branch
    return obj.length
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")//ans:'Incomprehensibilities' string length is 21 
    printLength(1000)//ans:'1000' string length is ... err, not a string 
    printLength(listOf(Any()))//ans:'[java.lang.Object@3af49f1c]' string length is ... err, not a string 
}
```
or even

甚至

**```if (obj is String && obj.length > 0)``` 在判斷式的右手邊  ```obj.length > 0``` 自動轉為 String 不用強轉 `as`**

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    // `obj` is automatically cast to `String` on the right-hand side of `&&`
    if (obj is String && obj.length > 0) {
        return obj.length
    }

    return null
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, is empty or not a string at all"} ")
    }
    printLength("Incomprehensibilities")//ans:'Incomprehensibilities' string length is 21 
    printLength("")//ans:'' string length is ... err, is empty or not a string at all 
    printLength(1000)//ans:'1000' string length is ... err, is empty or not a string at all 
}
```
See [Classes](classes.md) and [Type casts](typecasts.md).

## Using a `for` loop (使用 `for` 循環)

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    for (item in items) {
        println(item)
    }
//sampleEnd
}

//ans:
//apple
//banana
//kiwifruit
```
or

或

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
//sampleEnd
}

//ans:
//item at 0 is apple
//item at 1 is banana
//item at 2 is kiwifruit
```

See [for loop](control-flow.md#for-loops).

## Using a `while` loop (使用 `while` 循環)

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
//sampleEnd
}

//ans:
//item at 0 is apple
//item at 1 is banana
//item at 2 is kiwifruit
```

See [while loop](control-flow.md#while-loops).

## Using `when` expression (使用 `when` 表達式)

``` kotlin
//sampleStart
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
    }
//sampleEnd

fun main(args: Array<String>) {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}

//ans:
//One
//Greeting
//Long
//Not a string
//Unknown
```

See [when expression](control-flow.md#when-expression).

## Using ranges (使用範圍)

Check if a number is within a range using `in` operator:

使用 `in` 運算符檢查一個數值是否在範圍內：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val x = 10
    val y = 9
    if (x in 1..y+1) {
        println("fits in range")
    }
//sampleEnd
}
//ans:fits in range
```
----

Check if a number is out of range:

檢查一個數值是否超出範圍：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val list = listOf("a", "b", "c")
    
    if (-1 !in 0..list.lastIndex) {
        println("-1 is out of range")
    }
    if (list.size !in list.indices) {
        println("list size is out of valid list indices range too")
    }
//sampleEnd
}

//ans:
//-1 is out of range
//list size is out of valid list indices range too
```
----

Iterating over a range:

遍歷(走訪、循環)一個範圍

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    for (x in 1..5) {
        print(x)
    }
//sampleEnd
}

//ans:12345
```
----

or over a progression:

或超出進展(步數)

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    for (x in 1..10 step 2) {
        print(x)//ans:13579
    }
    println()
    for (x in 9 downTo 0 step 3) {
        print(x)//ans:9630
    }
//sampleEnd
}
```
See [Ranges](ranges.html).

## Using collections (使用集合)

Iterating over a collection:

遍歷(走訪、循環)集合：

``` kotlin
fun main(args: Array<String>) {
    val items = listOf("apple", "banana", "kiwifruit")
//sampleStart
    for (item in items) {
        println(item)
    }
//sampleEnd
}

//ans:
//apple
//banana
//kiwifruit
```
----

Checking if a collection contains an object using `in` operator:

使用 `in` 運算符檢查集合是否包含物件

``` kotlin
fun main(args: Array<String>) {
    val items = setOf("apple", "banana", "kiwifruit")
//sampleStart
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
//sampleEnd
}

//ans:apple is fine too
```
---

Using lambda expressions to filter and map collections:


使用表達式過濾和映射集合：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
  val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
  fruits
      .filter { it.startsWith("a") }//avocado,apple
      .sortedBy { it }//apple,avocado
      .map { it.toUpperCase() }//APPLE,AVOCADO
      .forEach { println(it) }
//sampleEnd
}

//ans:
//APPLE
//AVOCADO
```
See [Higher-order functions and Lambdas](lambdas.md).

## Creating basic classes and their instances: (建立基本類別和它的實例)

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val rectangle = Rectangle(5.0, 2.0) //no 'new' keyword required
    val triangle = Triangle(3.0, 4.0, 5.0)
//sampleEnd
    println("Area of rectangle is ${rectangle.calculateArea()}, its perimeter is ${rectangle.perimeter}")
    println("Area of triangle is ${triangle.calculateArea()}, its perimeter is ${triangle.perimeter}")
}

abstract class Shape(val sides: List<Double>) {
    val perimeter: Double get() = sides.sum()
    abstract fun calculateArea(): Double
}

interface RectangleProperties {
    val isSquare: Boolean
}

class Rectangle(
    var height: Double,
    var length: Double
) : Shape(listOf(height, length, height, length)), RectangleProperties {
    override val isSquare: Boolean get() = length == height
    override fun calculateArea(): Double = height * length
}

class Triangle(
    var sideA: Double,
    var sideB: Double,
    var sideC: Double
) : Shape(listOf(sideA, sideB, sideC)) {
    override fun calculateArea(): Double {
        val s = perimeter / 2
        return Math.sqrt(s * (s - sideA) * (s - sideB) * (s - sideC))
    }
}
```
See [classes](classes.md) and [objects and instances](object-declarations.md).
