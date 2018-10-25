---
type: doc
layout: reference
category: Other
title: "Collections: List, Set, Map"
---

# Collections: List, Set, Map

Collections: List, Set, Map ：集合： List , Set , Map

Unlike many languages, Kotlin distinguishes between mutable and immutable collections (lists, sets, maps, etc). Precise control over exactly when collections can be edited is useful for eliminating bugs, and for designing good APIs.

和很多語言不同， Kotlin 區分可變和不可變之間的集合 (lists , sets , maps , 等等) 。精確完整的掌控何時編輯集合對於消除 bug 和設計好的 API 是用的。

It is important to understand upfront the difference between a read-only _view_ of a mutable collection, and an actually immutable collection. Both are easy to create, but the type system doesn't express the difference, so keeping track of that (if it's relevant) is up to you.

提前瞭解可變集合的唯讀視圖和實際不可變的集合之間區別是重要的。兩者都容易創建，但類型系統無法表示差異，所以持續追蹤它 (如果它是相關的) 取決於你。

The Kotlin `List<out T>` type is an interface that provides read-only operations like `size`, `get` and so on. Like in Java, it inherits from `Collection<T>` and that in turn inherits from `Iterable<T>`. Methods that change the list are added by the `MutableList<T>` interface. This pattern holds also for `Set<out T>/MutableSet<T>` and `Map<K, out V>/MutableMap<K, V>`.

Kotlin的 [`List<out T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/index.html) 類型是介面，介面提供唯讀的操作像是 `size` 、 `get` 等等。像在 Java 的 [`List<E>`](https://docs.oracle.com/javase/8/docs/api/java/util/List.html)，它從 `Collection<T>` 繼承並依次從 `Iterable<T>` 繼承。透過 [`MutableList<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/index.html#kotlin.collections.MutableList) 介面新增改變 list 的方法。這樣的模式也適用 [`Set<out T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-set/index.html#kotlin.collections.Set)/[`MutableSet<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-set/index.html#kotlin.collections.MutableSet) 和 [`Map<K, out V>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/index.html#kotlin.collections.Map)/[`MutableMap<K, V>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-map/index.html#kotlin.collections.MutableMap) 。

We can see basic usage of the list and set types below:

我可以在下面看到 list 和 set 的基本用法：

``` kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3) //可變的集合
val readOnlyView: List<Int> = numbers //不可變的集合
println(numbers)        // prints "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // prints "[1, 2, 3, 4]"
readOnlyView.clear()    // -> does not compile ， 因為不可變的集合不能改變

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```

Kotlin does not have dedicated syntax constructs for creating lists or sets. Use methods from the standard library, such as `listOf()`, `mutableListOf()`, `setOf()`, `mutableSetOf()`. Map creation in NOT performance-critical code can be accomplished with a simple [idiom](idioms.html#read-only-map): `mapOf(a to b, c to d)`.

Kotlin 不會有建立 list 和 set 專門的語法結構。從標準函式庫使用方法，例如： `listOf()` 、 `mutableListOf()` 、 `setOf()` 、 `mutableSetOf()` 。在非性能-關鍵代碼建立 Map 可以使用簡短的慣語完成： `mapOf(a to b, c to d)` 。

Note that the `readOnlyView` variable points to the same list and changes as the underlying list changes. If the only references that exist to a list are of the read-only variety, we can consider the collection fully immutable. A simple way to create such a collection is like this:

注意： `readOnlyView` 變數指到相同的 list 並且因為底層的改變而改變 (改變底層 `numbers` 影響 `readOnlyView`)。如果只引用到存在的 list 為唯讀的類型，我們可以考慮完全不可變的集合類型。一個簡單的方式去創建像這樣的集合：

``` kotlin
val items = listOf(1, 2, 3)
```

Currently, the `listOf` method is implemented using an array list, but in future more memory-efficient fully immutable collection types could be returned that exploit the fact that they know they can't change.

目前， `listOf`  方法使用 array list 實作，但在未來可以回傳更高效記憶體完全不可變的集合類型，利用他們已知不可改變的事實。

Note that the read-only types are [covariant](generics.md#variance). That means, you can take a `List<Rectangle>` and assign it to `List<Shape>` assuming `Rectangle` inherits from `Shape` (the collection types have the same inheritance relationship as the element types). This wouldn't be allowed with the mutable collection types because it would allow for failures at runtime: you might add a `Circle` into the `List<Shape>`, creating a `List<Rectangle>` with a `Circle` in it somewhere else in the program.

注意：唯讀類型是[協變的](generics.md#variance)，意味著，假設 `Rectangle` 繼承自 `Shape` ，你可以帶入 `List<Rectangle>` 並分配它給 `List<Shape>` (集合類型與元素類型有相同的繼承關係) 。使用可變集合類型不允許這樣做，因為允許它會在運行時期失敗：你或許新增 `Circle` 到 `List<Shape>` ，在程式的其他地方使用的 `Circle` 放到創建的 `List<Rectangle>` 。

Sometimes you want to return to the caller a snapshot of a collection at a particular point in time, one that's guaranteed to not change:

有時你想要在特定時間點向調用者回傳集合的快照，保證不會更改：

``` kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```

The `toList` extension method just duplicates the lists items, thus, the returned list is guaranteed to never change.

`toList` 擴展方法只是複製 list 的項目，因此，保證已回傳的 list 永遠不會改變。

There are various useful extension methods on lists and sets that are worth being familiar with:

在 lists 和 sets 有各種有用的擴展方法值得熟悉：

``` kotlin
val items = listOf(1, 2, 3, 4)
items.first() == 1
items.last() == 4
items.filter { it % 2 == 0 }   // returns [2, 4]

val rwList = mutableListOf(1, 2, 3)
rwList.requireNoNulls()        // returns [1, 2, 3]
if (rwList.none { it > 6 }) println("No items above 6")  // prints "No items above 6"
val item = rwList.firstOrNull()
```

... as well as all the utilities you would expect such as sort, zip, fold, reduce and so on.

以及你預期想要的工具方法像是： sort , zip , fold , reduce 等等。

Maps follow the same pattern. They can be easily instantiated and accessed like this:

Maps 遵循相同的模式。他們像這樣簡單的實例化和存取：

``` kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])  // prints "1"
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```
