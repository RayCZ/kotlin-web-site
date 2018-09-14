---
type: doc
layout: reference
category: "Syntax"
title: "Generics: in, out, where"
---

# Generics

Generics ：泛型

As in Java, classes in Kotlin may have type parameters:

與 Java 一樣，在 Kotlin 的類別可以有類型參數：

``` kotlin
class Box<T>(t: T) {
    var value = t
}
```

In general, to create an instance of such a class, we need to provide the type arguments:

通常，要創建這樣一個類別的一個實例，我們需要提供類型參數：

``` kotlin
val box: Box<Int> = Box<Int>(1)
```

But if the parameters may be inferred, e.g. from the constructor arguments or by some other means, one is allowed to omit the type arguments:

但如果參數可以被推斷，例如：從建構元參數或透過一些其他的手段，允許省略類型類型參數：

``` kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```

## Variance

Variance ：變量元素，可能是存在一個類別、`List`、`Map`... 等的一個子元素是沒有繼承的關係

wildcard ：為語言中的一種表示法 `<Type>` 代表類別、`List`、`Map`... 等的元素類型

One of the most tricky parts of Java's type system is wildcard types `<Type>` (see [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)). And Kotlin doesn't have any. Instead, it has two other things: declaration-site variance and type projections.

Java 類型系統中最棘手的部分之一是通配符類型 `<Type>` (看 [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)) 。而且 Kotlin 沒有，相反的，它有另外兩件事：宣告場景的變量元素和類型預測。

First, let's think about why Java needs those mysterious wildcards. The problem is explained in [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 31: *Use bounded wildcards to increase API flexibility*. First, generic types in Java are **invariant**, meaning that `List<String>` is **not** a subtype of `List<Object>`. Why so? If List was not **invariant**, it would have been no better than Java's arrays, since the following code would have compiled and caused an exception at runtime:

首先，讓我們思考有關 Java 為何需要那些神秘的通配符。這個問題在 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 31: *Use bounded wildcards to increase API flexibility* 解釋。首先在 Java 的泛型類型是不可變的，意思著 `List<String>` 不是一個  `List<Object>` 的子類型。為什麼這樣？如果 List 不是不可變，這將沒有比 Java 的陣列好，因為以下代碼被編譯並且在運行時造成一個異常。

``` java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! The cause of the upcoming problem sits here. Java prohibits this!
objs.add(1); // Here we put an Integer into a list of Strings
String s = strs.get(0); // !!! ClassCastException: Cannot cast Integer to String
```

So, Java prohibits such things in order to guarantee run-time safety. But this has some implications. For example, consider the `addAll()` method from `Collection` interface. What's the signature of this method? Intuitively, we'd put it this way:

**Function signature：函數簽名，包括函數的名稱、參數順序、參數類型、泛型欄位等資訊總稱**

所以， Java 禁止這樣的事情，為了保證運行時的安全.。但這有些含意。例如：考慮從 `Collection` 介面裡的 `addAll()` 方法。這個方法的簽名是什麼？直觀地，我這麼說吧：

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

But then, we would not be able to do the following simple thing (which is perfectly safe):

但接下來，我們將無法做到以下簡單的事情 (這是非常安全的) ：

``` java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from);
  // !!! Would not compile with the naive declaration of addAll:
  // Collection<String> is not a subtype of Collection<Object>
}
```

(In Java, we learned this lesson the hard way, see [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 28: *Prefer lists to arrays*)

(在 Java ，我們以艱難的方式學習這課程，看 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 28: *Prefer lists to arrays*)

That's why the actual signature of `addAll()` is the following:

這是為何實際的 `addAll()` 簽名是以下：

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```

The **wildcard type argument** `? extends E` indicates that this method accepts a collection of objects of `E` *or some subtype of* `E`, not just `E` itself. This means that we can safely **read** `E`'s from items (elements of this collection are instances of a subclass of E), but **cannot write** to it since we do not know what objects comply to that unknown subtype of `E`. In return for this limitation, we have the desired behaviour: `Collection<String>` *is* a subtype of `Collection<? extends Object>`. In "clever words", the wildcard with an **extends**\-bound (**upper** bound) makes the type **covariant**.

**以下的 collection 便於瞭解可以看成是 List 介面**

通配符類型參數 `? extends E` 表示此方法接受一個集合 `List` 的元素是 E 類型的物件或某個 E 類型的子類型，不只有 E 類型本身。這意味著我們可以從集合中 `<List>` 的項目安全的讀出 E 類型 (這個集合 `<List>` 的元素是 E 子類別的實例) ，但不可以寫入它，因為我們不知道放入的元素是遵守什麼樣未知的 E 子類型，這個限制的回報，我們有渴望的行為： `Collection<String>` 是一個 `Collection<? extends Object>` 的子類型。 "聰明的單字" ，通配符有 **擴展**-邊界 (上限) 使得類型是協變的。

**covariant ：不可變，代表在類別、`List`、`Map`... 等的元素類型是不可變的，例如：`List <View>`，這個集合就是只能有 View 的元素，不能是 View 的父類別或子類別，為不可變的元素**

**covariant ：協變，代表在類別、`List`、`Map`... 等的元素類型是可變的，經由表示 `extends` 給一個擴展的類別繼承範圍，例如： `<? extends View>` ，以 Android 來說像是 `Button` 、 `ImageView` 、 `TextView` 只要是 `View` 的子類別都可以放入集合中，從繼承關係中協變而來的元素，為了確保在集合「取出」的物件類型是安全的 ，在 Kotlin `<out T>`**

**contravariance ：逆變，代表在類別、`List`、`Map`... 等的元素類型是逆向變化的，經由表示 `super` 給定限制的類別繼承範圍，例如： `<? Super Button>` ，以 Android 來說只能是 `Button` 的父類別才能放入，所以有 `TextView` 、 `View` 、 `Object`才可以放入，算是一種限制集合元素的方式，為了確保在集合「放入」的物件類型安全，配合協變的取出是安全的，在 Kotlin `<in T>`**

The key to understanding why this trick works is rather simple: if you can only **take** items from a collection, then using a collection of `String`s and reading `Object`s from it is fine. Conversely, if you can only _put_ items into the collection, it's OK to take a collection of `Object`s and put `String`s into it: in Java we have `List<? super String>` a **supertype** of `List<Object>`.

理解為什麼這個技巧的關鍵是相當簡單：如果你只可以從集合取出**項目**，接著使用一個 `String` 的集合並且從集合中讀出 `Object` 是可以的。相反，如果你只可以放入**項目**到集合，帶入一個 `object` 的集合並且放 `String` 到集合也是可以的：在 Java 中，我們有 `List<? super String>` 是一個 `List<Object>` 的子類型。

The latter is called **contravariance**, and you can only call methods that take String as an argument on `List<? super String>` (e.g., you can call `add(String)` or `set(int, String)`), while if you call something that returns `T` in `List<T>`, you don't get a `String`, but an `Object`.

後者稱為逆變，並且你只可以在 `List<? super String>` 調用方法帶入 String 為一個參數 (例如：你可以調用 `add(String)` 或 `set(int, String)`)，而如果你在 `List<T>` 調用回傳 `T` 的東西，你不會獲到 `String` ，而是一個 `Object` 。

Joshua Bloch calls those objects you only **read** from **Producers**, and those you only **write** to **Consumers**. He recommends: "*For maximum flexibility, use wildcard types on input parameters that represent producers or consumers*", and proposes the following mnemonic:

Joshua Bloch 說：調那些物件你只可以從 **Producers (生產者)** 讀取，並且那些你可以寫入給 **Consumers (消費者)** 。他的建議。"*For maximum flexibility, use wildcard types on input parameters that represent producers or consumers*" ，並且提出以下助記符：

**PECS stands for Producer-Extends, Consumer-Super.**

*NOTE*: if you use a producer-object, say, `List<? extends Foo>`, you are not allowed to call `add()` or `set()` on this object, but this does not mean that this object is **immutable**: for example, nothing prevents you from calling `clear()` to remove all items from the list, since `clear()` does not take any parameters at all. The only thing guaranteed by wildcards (or other types of variance) is **type safety**. Immutability is a completely different story.

### Declaration-site variance

Suppose we have a generic interface `Source<T>` that does not have any methods that take `T` as a parameter, only methods that return `T`:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
// Java
interface Source<T> {
  T nextT();
}
```
</div>

Then, it would be perfectly safe to store a reference to an instance of `Source<String>` in a variable of type `Source<Object>` -- there are no consumer-methods to call. But Java does not know this, and still prohibits it:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! Not allowed in Java
  // ...
}
```
</div>

To fix this, we have to declare objects of type `Source<? extends Object>`, which is sort of meaningless, because we can call all the same methods on such a variable as before, so there's no value added by the more complex type. But the compiler does not know that.

In Kotlin, there is a way to explain this sort of thing to the compiler. This is called **declaration-site variance**: we can annotate the **type parameter** `T` of Source to make sure that it is only **returned** (produced) from members of `Source<T>`, and never consumed. 
To do this we provide the **out** modifier:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```
</div>

The general rule is: when a type parameter `T` of a class `C` is declared **out**, it may occur only in **out**\-position in the members of `C`, but in return `C<Base>` can safely be a supertype 
of `C<Derived>`.

In "clever words" they say that the class `C` is **covariant** in the parameter `T`, or that `T` is a **covariant** type parameter. 
You can think of `C` as being a **producer** of `T`'s, and NOT a **consumer** of `T`'s.

The **out** modifier is called a **variance annotation**, and  since it is provided at the type parameter declaration site, we talk about **declaration-site variance**. 
This is in contrast with Java's **use-site variance** where wildcards in the type usages make the types covariant.

In addition to **out**, Kotlin provides a complementary variance annotation: **in**. It makes a type parameter **contravariant**: it can only be consumed and never 
produced. A good example of a contravariant type is `Comparable`:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```
</div>

We believe that the words **in** and **out** are self-explaining (as they were successfully used in C# for quite some time already), 
thus the mnemonic mentioned above is not really needed, and one can rephrase it for a higher purpose:

**[The Existential](http://en.wikipedia.org/wiki/Existentialism) Transformation: Consumer in, Producer out\!** :-)

## Type projections

### Use-site variance: Type projections

It is very convenient to declare a type parameter T as *out* and avoid trouble with subtyping on the use site, but some classes **can't** actually be restricted to only return `T`'s! 
A good example of this is Array:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { ... }
    fun set(index: Int, value: T) { ... }
}
```
</div>

This class cannot be either co\- or contravariant in `T`. And this imposes certain inflexibilities. Consider the following function:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```
</div>

This function is supposed to copy items from one array to another. Let's try to apply it in practice:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 
copy(ints, any) // Error: expects (Array<Any>, Array<Any>)
```
</div>

Here we run into the same familiar problem: `Array<T>` is **invariant** in `T`, thus neither of `Array<Int>` and `Array<Any>` 
is a subtype of the other. Why? Again, because copy **might** be doing bad things, i.e. it might attempt to **write**, say, a String to `from`,
and if we actually passed an array of `Int` there, a `ClassCastException` would have been thrown sometime later.

Then, the only thing we want to ensure is that `copy()` does not do any bad things. We want to prohibit it from **writing** to `from`, and we can:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }
```
</div>

What has happened here is called **type projection**: we said that `from` is not simply an array, but a restricted (**projected**) one: we can only call those methods that return the type parameter 
`T`, in this case it means that we can only call `get()`. This is our approach to **use-site variance**, and corresponds to Java's `Array<? extends Object>`, 
but in a slightly simpler way.

You can project a type with **in** as well:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun fill(dest: Array<in String>, value: String) { ... }
```
</div>

`Array<in String>` corresponds to Java's `Array<? super String>`, i.e. you can pass an array of `CharSequence` or an array of `Object` to the `fill()` function.

### Star-projections

Sometimes you want to say that you know nothing about the type argument, but still want to use it in a safe way.
The safe way here is to define such a projection of the generic type, that every concrete instantiation of that generic type would be a subtype of that projection.

Kotlin provides so called **star-projection** syntax for this:

 - For `Foo<out T : TUpper>`, where `T` is a covariant type parameter with the upper bound `TUpper`, `Foo<*>` is equivalent to `Foo<out TUpper>`. It means that when the `T` is unknown you can safely *read* values of `TUpper` from `Foo<*>`.
 - For `Foo<in T>`, where `T` is a contravariant type parameter, `Foo<*>` is equivalent to `Foo<in Nothing>`. It means there is nothing you can *write* to `Foo<*>` in a safe way when `T` is unknown.
 - For `Foo<T : TUpper>`, where `T` is an invariant type parameter with the upper bound `TUpper`, `Foo<*>` is equivalent to `Foo<out TUpper>` for reading values and to `Foo<in Nothing>` for writing values.

If a generic type has several type parameters each of them can be projected independently.
For example, if the type is declared as `interface Function<in T, out U>` we can imagine the following star-projections:

 - `Function<*, String>` means `Function<in Nothing, String>`;
 - `Function<Int, *>` means `Function<Int, out Any?>`;
 - `Function<*, *>` means `Function<in Nothing, out Any?>`.

*Note*: star-projections are very much like Java's raw types, but safe.

## Generic functions

Not only classes can have type parameters. Functions can, too. Type parameters are placed **before** the name of the function:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString() : String {  // extension function
    // ...
}
```
</div>

To call a generic function, specify the type arguments at the call site **after** the name of the function:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
val l = singletonList<Int>(1)
```
</div>

Type arguments can be omitted if they can be inferred from the context, so the following example works as well:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
val l = singletonList(1)
```
</div>

## Generic constraints

The set of all possible types that can be substituted for a given type parameter may be restricted by **generic constraints**.

### Upper bounds

The most common type of constraint is an **upper bound** that corresponds to Java's *extends* keyword:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun <T : Comparable<T>> sort(list: List<T>) {  ... }
```
</div>

The type specified after a colon is the **upper bound**: only a subtype of `Comparable<T>` may be substituted for `T`. For example:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```
</div>

The default upper bound (if none specified) is `Any?`. Only one upper bound can be specified inside the angle brackets.
If the same type parameter needs more than one upper bound, we need a separate **where**\-clause:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}
```
</div>

## Type erasure

The type safety checks that Kotlin performs for generic declaration usages are only done at compile time.
At runtime, the instances of generic types do not hold any information about their actual type arguments.
The type information is said to be *erased*. For example, the instances of `Foo<Bar>` and `Foo<Baz?>` are erased to
just `Foo<*>`.

Therefore, there is no general way to check whether an instance of a generic type was created with certain type
arguments at runtime, and the compiler [prohibits such *is*{: .keyword }-checks](typecasts.html#type-erasure-and-generic-type-checks).

Type casts to generic types with concrete type arguments, e.g. `foo as List<String>`, cannot be checked at runtime.  
These [unchecked casts](typecasts.html#unchecked-casts) can be used when type safety is implied by the high-level 
program logic but cannot be inferred directly by the compiler. The compiler issues a warning on unchecked casts, and at 
runtime, only the non-generic part is checked (equivalent to `foo as List<*>`).

The type arguments of generic function calls are also only checked at compile time. Inside the function bodies, 
the type parameters cannot be used for type checks, and type casts to type parameters (`foo as T`) are unchecked. However,
[reified type parameters](inline-functions.html#reified-type-parameters) of inline functions are substituted by the actual 
type arguments in the inlined function body at the call sites and thus can be used for type checks and casts,
with the same restrictions for instances of generic types as described above.
