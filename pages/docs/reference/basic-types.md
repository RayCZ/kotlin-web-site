---
type: doc
layout: reference
category: "Syntax"
title: "Basic Types: Numbers, Strings, Arrays"
---

# Basic Types

Basic Types ：基本類型

In Kotlin, everything is an object in the sense that we can call member functions and properties on any variable. Some of the types can have a special internal representation - for example, numbers, characters and booleans can be represented as primitive values at runtime - but to the user they look like ordinary classes. In this section we describe the basic types used in Kotlin: numbers, characters, booleans, arrays, and strings.

在 Kotlin 中，在某種意義上每件事都是一個物件，我們可以在任何變數調用成員函數或屬性。某些類型可以有特別的內部表示 - 例如，數值、字元、布林在運行時可以被表示為原生數值 - 但對於使用者，他們看起來像普通類別。在這個章節我們介紹在 Kotlin 中的基本類型：數值、字元、布林、陣列、字串。

## Numbers

Numbers ：數值

Kotlin handles numbers in a way close to Java, but not exactly the same. For example, there are no implicit widening conversions for numbers, and literals are slightly different in some cases.

在 Kotlin 以接近 Java 的方式處理數字，但不完全相同。例如：數字沒有明確的擴展轉換，並且在某些情況下文字略有不同。

Kotlin provides the following built-in types representing numbers (this is close to Java):

Kotlin 提供以下內建類型表示數值 (這與 Java 接近) ：

| Type (類型) | Bit width (位元長度) |
| --------- | ---------------- |
| Double    | 64               |
| Float     | 32               |
| Long      | 64               |
| Int       | 32               |
| Short     | 16               |
| Byte      | 8                |

Note that characters are not numbers in Kotlin.

注意在 Kotlin 字元不是數值。

---

### Literal Constants

Literal Constants ：文字常數

There are the following kinds of literal constants for integral values:

有以下幾種整數值的文字常數表示法：

* Decimals: `123`
  十進位：`123`
  * Longs are tagged by a capital `L`: `123L`
    透過大寫 `L` 標記長數值： `123L`
* Hexadecimals: `0x0F`
  十六進位： `0x0F`
* Binaries: `0b00001011`
  二進位： `0b00001011`

NOTE: Octal literals are not supported.

注意：八進位文字不支援。

Kotlin also supports a conventional notation for floating-point numbers:

Kotlin 也支援浮點數的常規符號：

* Doubles by default: `123.5`, `123.5e10`
  預設雙精準度： `123.5`、`123.5e10`
* Floats are tagged by `f` or `F`: `123.5f`
  標記 `f` 或 `F` 的浮點數： `123.5f`

---

### Underscores in numeric literals (since 1.1)

Underscores in numeric literals (since 1.1) ：數值文字中的底線  (從 1.1 版支援)

You can use underscores to make number constants more readable:

你可以使用底線使得數值常數更易讀：

``` kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```
---

### Representation

Representation ：表示法

On the Java platform, numbers are physically stored as JVM primitive types, unless we need a nullable number reference (e.g. `Int?`) or generics are involved. In the latter cases numbers are boxed.

在 Java 平台，數值在物理上存為 JVM 原生類型，除非你需要可空的數值引用 (例如： `Int?` ) 或泛型的涉及。在後一種情況會已自動裝箱。

Note that boxing of numbers does not necessarily preserve identity:

請注意：數字裝箱不一定會保留識別 (參照) ：

**===：使用這種表示法判斷是不是同一種參照、引用**

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val a: Int = 10000
    println(a === a) // Prints 'true'
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    println(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
//sampleEnd
}
```
On the other hand, it preserves equality:

另一方面，它保持數值的相等：

**==：使用這種表示法判斷是不是相同結構內容**
``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val a: Int = 10000
    println(a == a) // Prints 'true'
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    println(boxedA == anotherBoxedA) // Prints 'true'
//sampleEnd
}
```
---

### Explicit Conversions

Explicit Conversions ：明顯的轉換

Due to different representations, smaller types are not subtypes of bigger ones.
If they were, we would have troubles of the following sort:

由於表示法不同，較小的類型不是較大類型的子類型。如果是，我們會遇到以下排序麻煩：

``` kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(b == a) // Surprise! This prints "false" as Long's equals() checks whether the other is Long as well
```
So equality would have been lost silently all over the place, not to mention identity.

所以相同就會在整個地方默默的消失，更不用說識別 (參照) 。

As a consequence, smaller types are NOT implicitly converted to bigger types. This means that we cannot assign a value of type `Byte` to an `Int` variable without an explicit conversion

結果，較小的類型不會隱式轉換到大類型。這意味著我們不會在沒有明顯的轉換情況下將 `Byte` 類型值分配給 `Int` 變數
``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val b: Byte = 1 // OK, literals are checked statically
    val i: Int = b // ERROR
//sampleEnd
}
```
We can use explicit conversions to widen numbers

我們可以使用明顯的轉換來擴大數值
``` kotlin
fun main(args: Array<String>) {
    val b: Byte = 1
//sampleStart
    val i: Int = b.toInt() // OK: explicitly widened
    print(i)
//sampleEnd
}

```
Every number type supports the following conversions:

每個數值類型支援以下轉換：

* `toByte(): Byte`
* `toShort(): Short`
* `toInt(): Int`
* `toLong(): Long`
* `toFloat(): Float`
* `toDouble(): Double`
* `toChar(): Char`

Absence of implicit conversions is rarely noticeable because the type is inferred from the context, and arithmetical operations are overloaded for appropriate conversions, for example

缺乏隱式轉換很少會引起注意，因為類型是從上下文中推斷出來，並且適當的轉換為算術運算符多載，例如
``` kotlin
val l = 1L + 3 // Long + Int => Long
```
---

### Operations

Operations ：運算符

Kotlin supports the standard set of arithmetical operations over numbers, which are declared as members of appropriate classes (but the compiler optimizes the calls down to the corresponding instructions).
See [Operator overloading](operator-overloading.md).

Kotlin支援數字上的標準算術運算符組，它們被宣告為適當類別的成員 (但編譯器將優化調用對應的指令) 。
請參閱 [Operator overloading](operator-overloading.md) 。

As of bitwise operations, there're no special characters for them, but just named functions that can be called in infix form, for example:

作為二元運算符，他們沒有特別字元，只是在中綴形式可以調用命名函數，例如：

``` kotlin
val x = (1 shl 2) and 0x000FF000
```
Here is the complete list of bitwise operations (available for `Int` and `Long` only):

這裡是完整的二元運算符列表 (只能 `Int` 和 `Long` 可用)

* `shl(bits)` – signed shift left (Java's `<<`)
* `shr(bits)` – signed shift right (Java's `>>`)
* `ushr(bits)` – unsigned shift right (Java's `>>>`)
* `and(bits)` – bitwise and
* `or(bits)` – bitwise or
* `xor(bits)` – bitwise xor
* `inv()` – bitwise inversion

---

### Floating Point Numbers Comparison

Floating Point Numbers Comparison ：浮點數比對

The operations on floating point numbers discussed in this section are:

在這個章節討論浮點數運算符：

* Equality checks: `a == b` and `a != b`
  檢查相等： `a == b` 和 `a != b`
* Comparison operators: `a < b`, `a > b`, `a <= b`, `a >= b`
  比對運算符： `a < b` , `a > b` , `a <= b` , `a >= b`
* Range instantiation and range checks: `a..b`, `x in a..b`, `x !in a..b`
  範圍實例和檢查範圍： `a..b` , `x in a..b` , `x !in a..b`

When the operands `a` and `b` are statically known to be `Float` or `Double` or their nullable counterparts (the type is declared or inferred or is a result of a [smart cast](typecasts.md#smart-casts)), the operations on the numbers and the range that they form follow the IEEE 754 Standard for Floating-Point Arithmetic. 

當靜態已經知道運算符 `a` 和 `b` 是 `Float` 或 `Double` 或它們可空的對應物 (從宣告或推斷類型或是智能轉換的結果) ，在數值和範圍的運算符它們遵循浮點數算術的 IEEE 754 標準。

However, to support generic use cases and provide total ordering, when the operands are **not** statically typed as floating point numbers (e.g. `Any`, `Comparable<...>`, a type parameter), the operations use the `equals` and `compareTo` implementations for `Float` and `Double`, which disagree with the standard, so that:

但是，為了支援泛型使用案例和提供總排序，當運算符不是浮點數靜態類型 (例如： `Any` 、 `Comparable<...>` 、 參數類型)，運算符使用 `Float` 和 `Double` 的 `equals` 和 `compareTo` 實作，不同意標準，因此：

* `NaN` is considered equal to itself
  `NaN` 被認為與自身相同
* `NaN` is considered greater than any other element including `POSITIVE_INFINITY`
  `NaN` 被認為比其他元素都要大，包括 `POSITIVE_INFINITY`
* `-0.0` is considered less than `0.0`
  `-0.0` 被認為比 `0.0` 少

## Characters

Characters ：字元

Characters are represented by the type `Char`. They can not be treated directly as numbers

`Char` 類型表示為字元。你不可以直接視為數值

``` kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```
Character literals go in single quotes: `'1'`. Special characters can be escaped using a backslash. The following escape sequences are supported: `\t`, `\b`, `\n`, `\r`, `\'`, `\"`, `\\` and `\$`. To encode any other character, use the Unicode escape sequence syntax: `'\uFF00'`.

字元進入單引號：`'1'` 。可以使用反斜線 `\` 轉義 (跳脫) 為特別字元。支援以下轉義 (跳脫) 序列： `\t` , `\b `, `\n` , `\r` , `\'` , `\"` , `\\` 和 `\$` 。對任何其他字元進行編碼請使用 Unicode 轉義 (跳脫) 序列 `\uFF00` 。

We can explicitly convert a character to an `Int` number:

我們可以明顯的轉換字元到 `Int` 數值：

``` kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```
Like numbers, characters are boxed when a nullable reference is needed. Identity is not preserved by the boxing operation.

像數值一樣，當需要一個可空的參照自動裝箱字元。自動裝箱操作不會保留識別。

## Booleans

Booleans ：布林值

The type `Boolean` represents booleans, and has two values: *true* and *false*.

`Boolean` 類型表示為布林值，且有兩個值： `true` 和 `false` 。

Booleans are boxed if a nullable reference is needed.

如果需要可空參照自動裝箱布林值。

Built-in operations on booleans include

布林值的內建操作包括

* `||` – lazy disjunction (懶惰的分離)
* `&&` – lazy conjunction (懶惰的結合)
* `!` - negation (否定)

## Arrays

Arrays ：陣列

Arrays in Kotlin are represented by the `Array` class, that has `get` and `set` functions (that turn into `[]` by operator overloading conventions), and `size` property, along with a few other useful member functions:

`Array` 類別表示 Kotlin 中的陣列，有 `get` 和 `set` 函數 ( 透過運算符多載慣例轉換為 [] )，並有 `size` 屬性，以及一些其他有用的成員函數：

``` kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```
To create an array, we can use a library function `arrayOf()` and pass the item values to it, so that `arrayOf(1, 2, 3)` creates an array `[1, 2, 3]` . Alternatively, the `arrayOfNulls()` library function can be used to create an array of a given size filled with null elements.

創建陣列，我們可以使用函式庫數函數 `arrayOf` 並且傳遞項目值給它，以便 `arrayOf(1, 2, 3,)` 創建一個陣列 [ 1 , 2 , 3 ] 。或者 `arrayOfNulls()` 函式庫函數可以用於創建使用 null 元素填滿已給大小的陣列。

Another option is to use the `Array` constructor that takes the array size and the function that can return the initial value of each array element given its index:

另外一種選擇使用 `Array` 建構元，並帶入 陣列大小 `5` 和函數 `{....}` ，函數給予它的索引參數 `{ i -> ...}` 可以回傳每個陣列元素的初始值

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    // Creates an Array<String> with values ["0", "1", "4", "9", "16"]
    val asc = Array(5, { i -> (i * i).toString() })
    asc.forEach { println(it) }
//sampleEnd
}
```
As we said above, the `[]` operation stands for calls to member functions `get()` and `set()`.

如上所述，`[]` 操作代表調用成員函數 `get()` 和 `set()` 。

Note: unlike Java, arrays in Kotlin are invariant. This means that Kotlin does not let us assign an `Array<String>` to an `Array<Any>`, which prevents a possible runtime failure (but you can use `Array<out Any>`,  see [Type Projections](generics.md#type-projections)).

注意：不像 Java，在 Kotlin 陣列是不可變的。這意味著 Kotlin 不會讓我們分配一個 `Array<String>` 給 `Array<Any>` ，這可防止可能的運行時失敗 (但你可以使用 `Array<out Any>` ，詳細請看 [Type Projections](generics.md#type-projections)) 。

Kotlin also has specialized classes to represent arrays of primitive types without boxing overhead: `ByteArray`, `ShortArray`, `IntArray` and so on. These classes have no inheritance relation to the `Array` class, but they have the same set of methods and properties. Each of them also has a corresponding factory function:

Kotlin 也有專門類別來表示原生類型的陣列並沒有自動裝箱的開銷： `ByteArray` , `ShortArray` , `IntArray` 等等。這些類別沒有與 `Array` 繼承關係，但他們有相同方法和屬性集合。他們每個類型都有相應的工廠函數：

``` kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

## Unsigned integers

Unsigned integers ：無符號 (正負號) 的整數

> Unsigned types are available only since Kotlin 1.3 and currently are *experimental*. See details [below](#experimental-status-of-unsigned-integers) 
> 無符號類型只在 Kotlin 1.3 之後可用並且目前是實驗性的。詳細見[下文](#experimental-status-of-unsigned-integers)

Kotlin introduces following types for unsigned integers:

Kotlin 為無符號整數引入以下類型：

* `kotlin.UByte`: an unsigned 8-bit integer, ranges from 0 to 255
* `kotlin.UShort`: an unsigned 16-bit integer, ranges from 0 to 65535
* `kotlin.UInt`: an unsigned 32-bit integer, ranges from 0 to 2^32 - 1
* `kotlin.ULong`: an unsigned 64-bit integer, ranges from 0 to 2^64 - 1

Unsigned types support most of the operations of their signed counterparts.

無符號類型支援大多相對應有符號的操作。

> Note that changing type from unsigned type to signed counterpart (and vice versa) is a *binary incompatible* change
> 注意：改變類型從無符號到對應的有符號是二進制不兼容的改變

Unsigned types are implemented using another experimental feature, namely [inline classes](inline-classes.md).

使用另一個實驗性的功能實作無符號類型，即是[內置類別](inline-classes.md)

### Specialized classes 

Specialized classes  ：專門的類別

Same as for primitives, each of unsigned type has corresponding type that represents array, specialized for that unsigned type:

與原生類型相同，每個無符號類型有對應的表示陣列類型，專用於無符號類型：

* `kotlin.UByteArray`: an array of unsigned bytes
* `kotlin.UShortArray`: an array of unsigned shorts
* `kotlin.UIntArray`: an array of unsigned ints
* `kotlin.ULongArray`: an array of unsigned longs

Same as for signed integer arrays, they provide similar API to `Array` class without boxing overhead. 

與有符號整數陣列相同，它們提供相類似的 API 給 `Array` 類別沒有自動裝箱的開銷。

Also, [ranges and progressions](ranges.md) supported for `UInt` and `ULong` by classes `kotlin.ranges.UIntRange`, `kotlin.ranges.UIntProgression`, `kotlin.ranges.ULongRange`, `kotlin.ranges.ULongProgression` 

而且，透過類別  `kotlin.ranges.UIntRange` 、 `kotlin.ranges.UIntProgression` 、 `kotlin.ranges.ULongRange` 、 `kotlin.ranges.ULongProgression` 支援 `UInt` 和 `ULong` 的範圍和進度。

### Literals

Literals ：數值文字

To make unsigned integers easier to use, Kotlin provides an ability to tag an integer literal with a suffix indicating a specific unsigned type (similarly to Float/Long):

為了使無符號整數可容易使用， Kotlin 提供一個標記整數文字的功能，使用後綴表示一個專門無符號的類型 (類似 Float/Long) ：

* suffixes `u` and `U` tag literal as unsigned. Exact type will be determined based on the expected type. If no expected type is provided, `UInt` or `ULong` will be chosen based on the size of literal 
  後綴 `u` 和 `U` 標記文字為無符號，根據期望的類型決定確切的類型。如果沒有提供期望的類型，根據數值文字的大小選擇 `Uint` 或 `ULong`

``` kotlin
//有提供期望的類型 : Type
val b: UByte = 1u  // UByte, expected type provided
val s: UShort = 1u // UShort, expected type provided
val l: ULong = 1u  // ULong, expected type provided

//沒提供期望的類型
val a1 = 42u // UInt: no expected type provided, constant fits in UInt
val a2 = 0xFFFF_FFFF_FFFFu // ULong: no expected type provided, constant doesn't fit in UInt
```

* suffixes `uL` and `UL` explicitly tag literal as unsigned long.
  後綴 `uL` 和 `UL` 明確標記數值文字為無符號的長整數。

``` kotlin
val a = 1UL // ULong, even though no expected type provided and constant fits into UInt
```

### Experimental status of unsigned integers

The design of unsigned types is experimental, meaning that this feature is moving fast and no compatibility guarantees are given. When using unsigned arithmetics in Kotlin 1.3+, warning will be reported, indicating that this feature is experimental. To remove warning, you have to opt-in for experimental usage of unsigned types. 

There are two possible ways to opt-in for unsigned types: with marking your API as experimental too, or without doing that.

- to propagate experimentality, either annotate declarations which use unsigned integers with `@ExperimentalUnsignedTypes` or pass `-Xexperimental=kotlin.ExperimentalUnsignedTypes` to the compiler (note that the latter will make *all* declaration in compiled module experimental)
- to opt-in without propagating experimentality, either annotate declarations with `@UseExperimental(ExperimentalUnsignedTypes::class)` or pass `-Xuse-experimental=kotlin.ExperimentalUnsignedTypes`

It's up to you to decide if your clients have to explicitly opt-in into usage of your API, but bear in mind that unsigned types are an experimental feature, so API which uses them can be suddenly broken due to changes in language. 

See also or Experimental API [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/experimental.md) for technical details.

### Further discussion

See [language proposal for unsigned types](https://github.com/Kotlin/KEEP/blob/master/proposals/unsigned-types.md) for technical details and further discussion.

## Strings

Strings ：字串

Strings are represented by the type `String`. Strings are immutable. Elements of a string are characters that can be accessed by the indexing operation: `s[i]`. A string can be iterated over with a *for*-loop:

`String` 類型表示為字串。字串是不可變的。字串元素是字元可以透過索引操作存取： `s[i]` 。一個字串可以使用 `for-loop` 遍歷：

``` kotlin
fun main(args: Array<String>) {
val str = "abcd"
//sampleStart
for (c in str) {
    println(c)
}
//sampleEnd
}
```
You can concatenate strings using the `+` operator. This also works for concatenating strings with values of other types, as long as the first element in the expression is a string:

你可以使用 `+` 運算符連接字串。這也可用於使用其他類型的數值連接，以及在表示法中第一個元素是一個字串 `val s = "abc" + 1` ：
``` kotlin
fun main(args: Array<String>) {
//sampleStart
val s = "abc" + 1
println(s + "def")
//sampleEnd
}
```
Note that in most cases using [string templates](#string-templates) or raw strings is preferable to string concatenation.

注意：在大多情況下，使用字串模版或原始字串比字串連接更好。

---

### String Literals

String Literals ：字串文字

Kotlin has two types of string literals: escaped strings that may have escaped characters in them and raw strings that can contain newlines and arbitrary text. An escaped string is very much like a Java string:

Kotlin 有兩個字串文字類型：在字串中轉義 (跳脫) 字串可以有轉義 (跳脫) 字元並在原始字串可以包含換行和隨意文字。一個轉義 (跳脫) 字串非常像 Java 字串：
``` kotlin
val s = "Hello, world!\n"
```
Escaping is done in the conventional way, with a backslash. See [Characters](#characters) above for the list of supported escape sequences.

在常規方式做轉義 (跳脫) ，使用反斜線 `\` 。參閱支援轉義 (跳脫) 序列的列表 [Characters](#characters) 上述。

A raw string is delimited by a triple quote (`"""`), contains no escaping and can contain newlines and any other characters:

原始字串透過三引號 (`"""`) 分隔，沒有包含轉義 (跳脫) 字元且可以包含換行和任何其他字串：
``` kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```
You can remove leading whitespace with [`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) function:

你可以使用 [`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) 函數刪除前導空格：
``` kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```
By default `|` is used as margin prefix, but you can choose another character and pass it as a parameter, like `trimMargin(">")`.

透過預設 `|` 用為邊距前綴，但你可以選擇其他字元和傳遞它為參數，像 `trimMargin(">")` 。

---

### String Templates

String Templates ：字串模版

Strings may contain template expressions, i.e. pieces of code that are evaluated and whose results are concatenated into the string.A template expression starts with a dollar sign ($) and consists of either a simple name:

字串可以包含模版表示法，即執行代碼的片段並將結果連接到字串 `"println("i = $i")"` 。一個模版表示法開頭使用錢號 ($) 由一個簡單名稱組成：
``` kotlin
fun main(args: Array<String>) {
//sampleStart
val i = 10
println("i = $i") // prints "i = 10"
//sampleEnd
}
```
or an arbitrary expression in curly braces:

或在大括號中放一個隨意表示法 `${....}`：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
val s = "abc"
println("$s.length is ${s.length}") // prints "abc.length is 3"
//sampleEnd
}
```
Templates are supported both inside raw strings and inside escaped strings.If you need to represent a literal `$` character in a raw string (which doesn't support backslash escaping), you can use the following syntax:

模版支援在原始字串內和轉義 (跳脫) 字串內。如果你需要表示一個文字 `$` 字元在原始文字 (這不支援版斜線跳脫) ，你可以使用以下句法：

``` kotlin
val price = """
${'$'}9.99
"""
```