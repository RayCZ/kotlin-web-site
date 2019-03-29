---
type: doc
layout: reference
category: "Syntax"
title: "Type Checks and Casts: 'is' and 'as'"
---

# Type Checks and Casts: 'is' and 'as'

Type Checks and Casts: 'is' and 'as' ：類型檢查和強制轉型： `is` 和 `as`

## `is` and `!is` Operators

`is` 和 `!is` 運算符 

We can check whether an object conforms to a given type at runtime by using the `is` operator or its negated form `!is`:

我們可以透過使用 `is` 運算符或它的否定形式 `!is` 檢查物件是否在運行時符合特定的類型：

``` kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
}
else {
    print(obj.length)
}
```

## Smart Casts

Smart Casts ：智能型強制轉型

In many cases, one does not need to use explicit cast operators in Kotlin, because the compiler tracks the `is`-checks and [explicit casts](#unsafe-cast-operator) for immutable values and inserts (safe) casts automatically when needed:

在很多情況，在 Kotlin 不需要使用明顯的強制轉型運算符，因為編譯器會追蹤不可變的值做 `is`-檢查和[明顯的強制轉型](#unsafe-cast-operator)並在需要時自動插入 (安全) 強制轉型：

``` kotlin
fun demo(x: Any) {
    if (x is String) { //已經做過 is 判斷，系統自己做智能型強制轉型
        print(x.length) // x is automatically cast to String
    }
}
```

The compiler is smart enough to know a cast to be safe if a negative check leads to a return:

如果否定檢查導致回傳，編譯器是足夠聰明，知道強制轉型是安全的：

``` kotlin
    if (x !is String) return
    print(x.length) // x is automatically cast to String
```

or in the right-hand side of `&&` and `||`:

或在 `&&` 和 `||` 的右手邊：

``` kotlin
    // x is automatically cast to string on the right-hand side of `||`
    if (x !is String || x.length == 0) return

    // x is automatically cast to string on the right-hand side of `&&`
    if (x is String && x.length > 0) {
        print(x.length) // x is automatically cast to String
    }
```

Such _smart casts_ work for [*when*-expressions](control-flow.md#when-expression) and [*while*-loops](control-flow.md#while-loops) as well:

這樣的智能型強轉也適用於 [`when`-表達式](control-flow.md#when-expression) 和 [*while*-loops](control-flow.md#while-loops) ：

``` kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

Note that smart casts do not work when the compiler cannot guarantee that the variable cannot change between the check and the usage. More specifically, smart casts are applicable according to the following rules:

注意：當編譯器不保證變數在檢查和使用之間不可更改時，智能型強制轉型無法使用，更具體，智能型強制根據以下規則適用：

  * *val* local variables - always except for [local delegated properties](delegated-properties.md#local-delegated-properties-since-11);
    `val` 區域變數 - 總是除了[區域派外屬性](delegated-properties.md#local-delegated-properties-since-11)；
  * *val* properties - if the property is private or internal or the check is performed in the same module where the property is declared. Smart casts aren't applicable to open properties or properties that have custom getters;
    `val` 屬性 - 如果屬性是私有的或內部的或在相同模組宣告屬性執行檢查。智能型強制轉型不適用開放屬性或有自定義設置器的屬性；
  * *var* local variables - if the variable is not modified between the check and the usage, is not captured in a lambda that modifies it, and is not a local delegated property;
    `var` 區域變數 - 如果變數在檢查和使用之間不被更改，修改它在 Lambda 不會補獲，並且不是區域派外屬性；
  * *var* properties - never (because the variable can be modified at any time by other code).
    `var` 屬性 - 從不 (因為變數可以透過其他的程式碼在任何時間修改) 。

## "Unsafe" cast operator

"Unsafe" cast operator ： 「不安全的」強制轉行運算符

Usually, the cast operator throws an exception if the cast is not possible. Thus, we call it *unsafe*. The unsafe cast in Kotlin is done by the infix operator *as* (see [operator precedence](https://kotlinlang.org/docs/reference/grammar.html#precedence)):

通常，如果不能強制轉型，強制轉型丟出異常。因此，我們稱它「不安全」。在 Kotlin 的不安全強制轉型透過中綴運算符 `as` 完成 (參閱 [operator precedence](https://kotlinlang.org/docs/reference/grammar.html#precedence)) 。

``` kotlin
val x: String = y as String
```

Note that *null* cannot be cast to `String` as this type is not [nullable](null-safety.md), i.e. if `y` is null, the code above throws an exception. In order to match Java cast semantics we have to have nullable type at cast right hand side, like:

注意：「null」不可以強制轉型為 `String` ，因為此類型不能是[可空的](null-safety.md)，換句話說，如果 `y` 是 null ，上面的代碼丟出異常。為了匹配 Java 強制轉型語義，我們必須在強制轉型的右手邊有可空的類型，像：

``` kotlin
val x: String? = y as String?
```

## "Safe" (nullable) cast operator

"Safe" (nullable) cast operator ： 「安全的」 (可空的) 強制轉型運算符

To avoid an exception being thrown, one can use a *safe* cast operator *as?* that returns *null* on failure:

為了避免丟出異常，可以使用「安全的」強制轉型運算符 `as?` 在失敗時回傳「null」： 

``` kotlin
val x: String? = y as? String
```

Note that despite the fact that the right-hand side of *as?* is a non-null type `String` the result of the cast is nullable.

注意：雖然事實上 `as?` 的右手邊是非-可空的類型 `String` 強制轉型的結果是可空的。

## Type erasure and generic type checks

Type erasure and generic type checks ：類型消除和泛型檢查

Kotlin ensures type safety of operations involving [generics](generics.md) at compile time, while, at runtime, instances of generic types hold no information about their actual type arguments. For example, `List<Foo>` is erased to just `List<*>`. In general, there is no way to check whether an instance belongs to a generic type with certain type arguments at runtime. 

Kotlin 在編譯時期確保涉及[泛型](generics.md)操作的類型安全，然而，在運行時，泛型類型的實例沒有保存關於實際類型參數的資訊。例如： `List<Foo>` 被消除為 `List<*>` 。通常，在運行時沒有方法檢查實例是否使用某類型參數的泛型類型。

Given that, the compiler prohibits *is*-checks that cannot be performed at runtime due to type erasure, such as `ints is List<Int>` or `list is T` (type parameter). You can, however, check an instance against a [star-projected type](generics.md#star-projections):

以上所知，編譯器由於類型消除在運行時禁止並不可以執行 `is`-檢查，例如： `ints is List<Int>` 或 `list is T` (類型參數) 。然而，你可以針對[星號 - 投射類型](generics.md#star-projections)檢查實例：

```kotlin
if (something is List<*>) {
    something.forEach { println(it) } // The items are typed as `Any?`
}
```

Similarly, when you already have the type arguments of an instance checked statically (at compile time), you can make an *is*-check or a cast that involves the non-generic part of the type. Note that angle brackets are omitted in this case:

類似的，當你已經有靜態檢查類型參數的實例 (在編譯時) ，你可以進行 `is`-檢查或涉及類型的非-泛型部份的強制轉型。注意：在這情況省略尖括號 `<String>` 。

```kotlin
fun handleStrings(list: List<String>) {
    if (list is ArrayList) {
        // `list` is smart-cast to `ArrayList<String>`
    }
}
```

The same syntax with omitted type arguments can be used for casts that do not take type arguments into account: `list as ArrayList`. 

使用省略類型參數的相同語法可以用於不考慮類型參數的強制轉型： `list as ArrayList` 。

Inline functions with [reified type parameters](inline-functions.md#reified-type-parameters) have their actual type arguments inlined at each call site, which enables `arg is T` checks for the type parameters, but if `arg` is an instance of a generic type itself, *its* type arguments are still erased. Example:

使用[具體類型參數](inline-functions.md#reified-type-parameters)的行內置入函數，在每個調用場景置入它們實際類型參數，使得 `arg is T` 對類型參數做檢查，但如果 `arg` 泛型類型本身的實例，它的類型參數還是會被消失，例如： 


```kotlin
//sampleStart
inline fun <reified A, reified B> Pair<*, *>.asPairOf(): Pair<A, B>? {
    if (first !is A || second !is B) return null
    return first as A to second as B
}

val somePair: Pair<Any?, Any?> = "items" to listOf(1, 2, 3)

val stringToSomething = somePair.asPairOf<String, Any>()
val stringToInt = somePair.asPairOf<String, Int>()
val stringToList = somePair.asPairOf<String, List<*>>()
val stringToStringList = somePair.asPairOf<String, List<String>>() // Breaks type safety!
//sampleEnd

fun main() {
    println("stringToSomething = " + stringToSomething)
    println("stringToInt = " + stringToInt)
    println("stringToList = " + stringToList)
    println("stringToStringList = " + stringToStringList)
}

//ans:
//stringToSomething = (items, [1, 2, 3])
//stringToInt = null
//stringToList = (items, [1, 2, 3])
//stringToStringList = (items, [1, 2, 3])
```

## Unchecked casts

Unchecked casts ：未檢查的強制轉型

As said above, type erasure makes checking actual type arguments of a generic type instance impossible at runtime, and generic types in the code might be connected to each other not closely enough for the compiler to ensure type safety. 

如上所述，類型消除在運行時使得檢查泛型類型實例的實際類型參數是不可能的，並且在代碼中的泛型類型可能彼此連接不夠緊密以致於編譯器無法確保類型安全。

Even so, sometimes we have high-level program logic that implies type safety instead. For example:

即使如此，有時我們高階的程式邏輯而不是類型安全。例如：

``` kotlin
fun readDictionary(file: File): Map<String, *> = file.inputStream().use { 
    TODO("Read a mapping of strings to arbitrary elements.")
}

// We saved a map with `Int`s into that file
val intsFile = File("ints.dictionary")

// Warning: Unchecked cast: `Map<String, *>` to `Map<String, Int>`
val intsDictionary: Map<String, Int> = readDictionary(intsFile) as Map<String, Int>
```

The compiler produces a warning for the cast in the last line. The cast cannot be fully checked at runtime and provides no guarantee that the values in the map are `Int`.

編譯器在最後一行生成強制轉型的警告。在運行時無法完全檢查強制轉型並且無提供保證在 map 的值是 `Int` 。 

To avoid unchecked casts, you can redesign the program structure: in the example above, there could be interfaces `DictionaryReader<T>` and `DictionaryWriter<T>` with type-safe implementations for different types.  You can introduce reasonable abstractions to move unchecked casts from calling code to the implementation details. Proper use of [generic variance](generics.md#variance) can also help. 

為了避免未檢查的強制轉型，你可以重新設計程式結構：在上面的例子，可以使用類型-安全實作不同類型介面 `DictionaryReader<T>` 和 `DictionaryWriter<T>` 。你可以引入合理的抽象搬移未檢查的強制轉型從調用代碼到實作細節。正確使用[泛型變量](generics.md#variance)也有幫助。

For generic functions, using [reified type parameters](inline-functions.md#reified-type-parameters) makes the casts such as `arg as T` checked, unless `arg`'s type has *its own* type arguments that are erased.

對於泛型函數，使用[具體的類型參數](inline-functions.md#reified-type-parameters)處理強制轉型，例如： `arg as T` 檢查，除了 arg 的類型消除本身擁有的類型參數。

An unchecked cast warning can be suppressed by [annotating](annotations.md#annotations) the statement or the declaration where it occurs with `@Suppress("UNCHECKED_CAST")`:

透過使用 `@Suppress("UNCHECKED_CAST")` [註釋](annotations.md#annotations)敘述或宣告發生所在的地方，來抑制未檢查的強制轉型警告：

```kotlin
inline fun <reified T> List<*>.asListOfType(): List<T>? =
    if (all { it is T })
        @Suppress("UNCHECKED_CAST")
        this as List<T> else
        null
```

On the JVM, the [array types](basic-types.md#arrays) (`Array<Foo>`) retain the information about the erased type of their elements, and the type casts to an array type are partially checked: the nullability and actual type arguments of the elements type are still erased. For example, the cast `foo as Array<List<String>?>` will succeed if `foo` is an array holding any `List<*>`, nullable or not.

在 JVM 上，[陣列類型](basic-types.md#arrays)保留有關它的元素已消除類型的資訊，並且檢查部份陣列類型的強制轉型：可空的和仍然消除元素類型的實際類型參數。例如：如果 `foo` 是陣列持有任何 `List<*>` 元素，強制轉型 `foo as Array<List<String>?>` 將成功