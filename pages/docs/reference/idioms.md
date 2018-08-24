---
type: doc
layout: reference
category: "Basics"
title: "Idioms"
---

# Idioms (慣用語法)

A collection of random and frequently used idioms in Kotlin. If you have a favorite idiom, contribute it by sending a pull request.

在Kotlin中隨機和經常使用的慣語集合，如果你有最喜歡的慣語，請寄送pull request貢獻它

---

### Creating DTOs (POJOs/POCOs)

``` kotlin
data class Customer(val name: String, val email: String)
```
provides a `Customer` class with the following functionality:

為 `Customer` 類別自動提供以下函數：

* getters (and setters in case of `var` ) for all properties
* `equals()`
* `hashCode()`
* `toString()`
* `copy()`
* `component1()`, `component2()`, ..., for all properties (see [Data classes](data-classes.md))


---

### Default values for function parameters (函數參數的預設值)

``` kotlin
fun foo(a: Int = 0, b: String = "") { ... }
```
---

### Filtering a list (過濾列表)

**Lambda表達式{}可以依自己需求要不要加參數**

``` kotlin
val positives = list.filter { x -> x > 0 }
```
Or alternatively, even shorter:

或者，甚至更短：

``` kotlin
val positives = list.filter { it > 0 }
```
---

### String Interpolation (字串插值)

**別名：String Templates**

``` kotlin
println("Name $name")
```
---

### Instance Checks (實例檢查)

**實例檢查用 `is`**

``` kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```
---

### Traversing a map/list of pairs (遍歷映射/結對的列表)

``` kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```
`k`, `v` can be called anything.

---

### Using ranges (使用範圍)

**注意：until不包括最後一個**

``` kotlin
for (i in 1..100) { ... }  // closed range: includes 100
for (i in 1 until 100) { ... } // half-open range: does not include 100
for (x in 2..10 step 2) { ... }
for (x in 10 downTo 1) { ... }
if (x in 1..10) { ... }
```
---

### Read-only list (唯讀列表)

``` kotlin
val list = listOf("a", "b", "c")
```
---

### Read-only map (唯讀映射)

``` kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```
---

### Accessing a map (存取映射)

``` kotlin
println(map["key"])
map["key"] = value
```
---

### Lazy property (惰性/執行時才產生的屬性 `by lazy`)

**lazy：一定要宣告 `val`，當程式有調用屬性時才建構**

``` kotlin
val p: String by lazy {
    // compute the string
}
```
---

### Extension Functions (擴展函數)

``` kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```
---

### Creating a singleton (建立單例模式 `object`)

**object：與類別用法相同，差別多做了單例模式**

``` kotlin
object Resource {
    val name = "Name"
}
```
---

### If not null shorthand (if not null的速記法 `?.`)

``` kotlin
val files = File("Test").listFiles()

println(files?.size)
```
---

### If not null and else shorthand (if not null 和 else 速記法 `?.` `?:`)

``` kotlin
val files = File("Test").listFiles()

println(files?.size ?: "empty")
```
---

### Executing a statement if null ( if null 的執行敘述 `?:`)

``` kotlin
val values = ...
val email = values["email"] ?: throw IllegalStateException("Email is missing!")
```
---

### Get first item of a possibly empty collection (取得集合第一個可能為空的項目)

``` kotlin
val emails = ... // might be empty
val mainEmail = emails.firstOrNull() ?: ""
```
---

### Execute if not null (if not null的執行 `value?.let{}`)

``` kotlin
val value = ...

value?.let {
    ... // execute this block if not null
}
```
---

### Map nullable value if not null (if not null 處理映射可空的值 `value?.let { transformValue(it) }`)

``` kotlin
val value = ...

val mapped = value?.let { transformValue(it) } ?: defaultValueIfValueIsNull
```
---

### Return on when statement (回傳`when`的敘述可由變數接收)

``` kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```
---

### 'try/catch' expression (try/cache 表達式可由變數接收)

``` kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // Working with result
}
```
---

### 'if' expression ('if' 表達式可由變數接收)

``` kotlin
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```
---

### Builder-style usage of methods that return `Unit` (建構者風格用法的方法回傳 `Unit`)

``` kotlin
fun arrayOfMinusOnes(size: Int): IntArray {
    return IntArray(size).apply { fill(-1) }
}
```
---


### Single-expression functions (單行表達式函數)

``` kotlin
fun theAnswer() = 42
```
This is equivalent to

這相當於

``` kotlin
fun theAnswer(): Int {
    return 42
}
```
This can be effectively combined with other idioms, leading to shorter code. E.g. with the `when`-expression:

這可以有效的與其他慣語結合，從而縮短代碼。例如：使用 `when`表達式
``` kotlin
fun transform(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}
```
---

### Calling multiple methods on an object instance ('with') (在物件實例調用多個方法與 `with`)

``` kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //draw a 100 pix square
    penDown()
    for(i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```
---

### Java 7's try with resources (嘗試使用 java 7的資源)

``` kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```
---

### Convenient form for a generic function that requires the generic type information (泛型函數的方便形式需要泛型相關的資訊)

``` kotlin
//  public final class Gson {
//     ...
//     public <T> T fromJson(JsonElement json, Class<T> classOfT) throws JsonSyntaxException {
//     ...

inline fun <reified T: Any> Gson.fromJson(json: JsonElement): T = this.fromJson(json, T::class.java)
```
---

### Consuming a nullable Boolean (使用可空的Boolean)

``` kotlin
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b` is false or null
}
```

