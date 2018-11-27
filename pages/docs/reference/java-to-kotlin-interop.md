---
type: doc
layout: reference
category: "Java Interop"
title: "Calling Kotlin from Java"
---

# Calling Kotlin from Java

Calling Kotlin from Java ：從 Java 調用 Kotlin

Kotlin code can be called from Java easily.

可以輕鬆從 Java 可以調用 Kotlin 代碼。 

## Properties

Properties ：屬性

A Kotlin property is compiled to the following Java elements:

Kotlin 的屬性被編譯到以下 Java 元素：

 * A getter method, with the name calculated by prepending the `get` prefix;
   一個獲取屬性的方法，使用名稱透過前面加上 `get` 前綴執行；
 * A setter method, with the name calculated by prepending the `set` prefix (only for `var` properties);
   一個設置屬性的方法，使用名稱透過前面加上 `set` 前綴執行 (只用於 `var` 屬性)；
 * A private field, with the same name as the property name (only for properties with backing fields).
   一個私有欄位，使用與屬性相同的名稱 (只用於使用支援欄位的屬性) 。

For example, `var firstName: String` gets compiled to the following Java declarations:

例如， `var firstName: String` 被編譯為以下 Java 宣告：

``` java
private String firstName;

public String getFirstName() {
    return firstName;
}

public void setFirstName(String firstName) {
    this.firstName = firstName;
}
```

If the name of the property starts with `is`, a different name mapping rule is used: the name of the getter will be the same as the property name, and the name of the setter will be obtained by replacing `is` with `set`. For example, for a property `isOpen`, the getter will be called `isOpen()` and the setter will be called `setOpen()`. This rule applies for properties of any type, not just `Boolean`.

如果屬性的名稱使用 `is` 開頭，使用不同名稱的映射規則：獲取屬性的名稱將與屬性名稱相同，並且設置屬性的名稱將透過 `set` 代替 `is` 獲得，例如，對於屬性 `isOpen` ，獲取屬性將調用 `isOpen()` 並且設置屬性將調用 `setOpen()` 。這個規則適用於任何類型的屬性，不只 `Boolean` 。 

## Package-Level Functions

Package-Level Functions ： Package-Level 函數

All the functions and properties declared in a file `example.kt` inside a package `org.foo.bar`, including extension functions, are compiled into static methods of a Java class named `org.foo.bar.ExampleKt`.

在 package `org.foo.bar` 內的 `example.kt` 檔案宣告所有函數和屬性，包括擴展函數，被編譯到被命名為 `org.foo.bar.ExampleKt`  Java 類別的靜態方法。

**Package-Level 讓 Kotlin 檔案 (.kt) 生成一個類別 XxxxKt，而檔案內宣告的類別將生成另一個類別檔，以下會有兩個類別檔： ExampleKt.class 、 Foo.class**

``` kotlin
// example.kt
package demo

class Foo

fun bar() { ... }

```

``` java
// Java
new demo.Foo();
demo.ExampleKt.bar(); // Java 類別 ExampleKt 的 bar 靜態方法
```

The name of the generated Java class can be changed using the `@JvmName` annotation:

可以使用 `@JvmName` 註釋改變生成 Java 類別的名稱：

``` kotlin
@file:JvmName("DemoUtils") // 本來是依 .kt 檔來生成類別 XxxxKt 改成 DemoUtils

package demo

class Foo

fun bar() { ... }

```

``` java
// Java
new demo.Foo();
demo.DemoUtils.bar(); //使用類別 DemoUtils 的 bar 靜態方法
```

Having multiple files which have the same generated Java class name (the same package and the same name or the same @JvmName annotation) is normally an error. However, the compiler has the ability to generate a single Java facade class which has the specified name and contains all the declarations from all the files which have that name. To enable the generation of such a facade, use the @JvmMultifileClass annotation in all of the files.

有多個檔案生成相同的 Java 類別名稱 (相同 package 及相同名稱或相同 `@JvmName` 註釋) 通常是錯誤的。然而，編譯器能夠生成有指定名稱的單個 Java 外觀類別，並包含從有該名稱所有檔案的所有宣告。啟用這樣的外觀生成方式，在所有的檔案使用 `@JvmMultifileClass` 註釋。

``` kotlin
// oldutils.kt
@file:JvmName("Utils") // 生成的 Java 類別名
@file:JvmMultifileClass // 啟用多檔案相同名稱

package demo

fun foo() { ... }
```

``` kotlin
// newutils.kt
@file:JvmName("Utils") // 生成的 Java 類別名
@file:JvmMultifileClass // 啟用多檔案相同名稱

package demo

fun bar() { ... }
```

``` java
// Java
demo.Utils.foo();
demo.Utils.bar(); // 同個 Utils 類別包含靜態的 foo() 、 bar() 方法
```

## Instance Fields

Instance Fields ：實例欄位

If you need to expose a Kotlin property as a field in Java, you need to annotate it with the `@JvmField` annotation. The field will have the same visibility as the underlying property. You can annotate a property with `@JvmField` if it has a backing field, is not private, does not have `open`, `override` or `const` modifiers, and is not a delegated property.

如果你需要揭露 Kotlin 的屬性為 Java 的欄位，你需要使用 `@JvmField` 註釋註記它。欄位將有與底層屬性相同的可見性。如果屬性有支援欄位，不是私有的，不會有 `open` 、 `override` 、 `const` 修飾符並且不是委外屬性，你可以使用 `@JvmField` 註記屬性。

``` kotlin
class C(id: String) {
    @JvmField val ID = id // 註記屬性 @JvmField
}
```

``` java
// Java
class JavaClient {
    public String getID(C c) {
        return c.ID; // 調用屬性
    }
}
```

[Late-Initialized](properties.md#late-initialized-properties-and-variables) properties are also exposed as fields. The visibility of the field will be the same as the visibility of `lateinit` property setter.

[延遲初始化](properties.md#late-initialized-properties-and-variables)屬性也被揭露為欄位。欄位的可見性將與 `lateinit` 屬性的設置屬性方法可見性相同。

## Static Fields

Static Fields ：靜態欄位

Kotlin properties declared in a named object or a companion object will have static backing fields
either in that named object or in the class containing the companion object.

在已命名物件或夥伴物件中宣告 Kotlin 的屬性，將在已命名的物件或在包含夥伴物件的類別中有靜態支援欄位。

Usually these fields are private but they can be exposed in one of the following ways:

通常這些欄位是私有的，但可以在以下方式之一揭露他們：

 - `@JvmField` annotation;
 - `lateinit` modifier;
 - `const` modifier.

Annotating such a property with `@JvmField` makes it a static field with the same visibility as the property itself.

使用 `@JvmField` 註記這樣的屬性，讓它變為靜態欄位，與屬性本身相同的可見性。

``` kotlin
class Key(val value: Int) {
    companion object {
        @JvmField
        val COMPARATOR: Comparator<Key> = compareBy<Key> { it.value } // 為靜態的欄位
    }
}
```

``` java
// Java
Key.COMPARATOR.compare(key1, key2); // 為靜態的欄位所以可以直接調用屬性
// public static final field in Key class
```

A [late-initialized](properties.html#late-initialized-properties-and-variables) property in an object or a companion object has a static backing field with the same visibility as the property setter.

在物件或夥伴物件的[延遲初始化](properties.md#late-initialized-properties-and-variables)屬性有靜態支援欄位與屬性的設置屬性方法相同可見性。

``` kotlin
object Singleton {
    lateinit var provider: Provider // 延遲初始化的屬性，為靜態支援欄位
}
```

``` java
// Java
Singleton.provider = new Provider(); // 為靜態的欄位所以可以直接調用屬性
// public static non-final field in Singleton class
```

Properties annotated with `const` (in classes as well as at the top level) are turned into static fields in Java:

使用 `const` 註記屬性 (在類別中以及最高層級) 被轉換到 Java 的靜態欄位：

``` kotlin
// file example.kt

object Obj {
    const val CONST = 1 // 物件內使用 const 註記
}

class C {
    companion object {
        const val VERSION = 9 // 夥伴物件內 const 註記
    }
}

const val MAX = 239 // 最高層級 const 註記
```

In Java:

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
int v = C.VERSION;
```

## Static Methods

Static Methods ：靜態方法

As mentioned above, Kotlin represents package-level functions as static methods. Kotlin can also generate static methods for functions defined in named objects or companion objects if you annotate those functions as `@JvmStatic`. If you use this annotation, the compiler will generate both a static method in the enclosing class of the object and an instance method in the object itself. For example:

如上所述， Kolin 表示 package-level 函數為靜態方法。 如果你可以註記那些函數為 `@JvmStatic` ， Kotlin 也可以在已命名的物件或夥伴物件中定義函數生成靜態方法。如果你使用這樣的註釋，編譯器將在物件的封閉類別中生成靜態方法，並在物件本身生成實例。例如：

``` kotlin
class C {
    companion object {
        @JvmStatic fun foo() {}
        fun bar() {}
    }
}
```

Now, `foo()` is static in Java, while `bar()` is not:

現在， `foo()` 在 Java 中是靜態，而 `bar()` 不是：

``` java
C.foo(); // works fine , 在物件的封閉類別中生成靜態方法
C.bar(); // error: not a static method , 不是靜態無法靜類別直接調用，必須透過實例調用
C.Companion.foo(); // instance method remains , 在物件本身生成實例，來調用
C.Companion.bar(); // the only way it works , 唯一的調用方式
```

Same for named objects:

已命名的物件也是相同方式：

``` kotlin
object Obj {
    @JvmStatic fun foo() {}
    fun bar() {}
}
```

In Java:

在 Java ：

``` java
Obj.foo(); // works fine , 物件類型直接調用，而非實例
Obj.bar(); // error , 不是靜態無法物件直接調用，必須透過實例調用
Obj.INSTANCE.bar(); // works, a call through the singleton instance , 在物件本身生成實例，來調用
Obj.INSTANCE.foo(); // works too , 透過實例調用
```

`@JvmStatic` annotation can also be applied on a property of an object or a companion object making its getter and setter methods be static members in that object or the class containing the companion object.

`@JvmStatic` 註釋也可以在物件或夥伴物件的屬性應用，使它的獲取屬性和設置屬性方法在物件或類別包含的夥伴物件成為靜態成員。

## Visibility

Visibility ：可見性

The Kotlin visibilities are mapped to Java in the following way:

Kotlin 可性見在以下方法映射到 Java ：

* `private` members are compiled to `private` members;
  `private` 成員被編譯到 `private` 成員；
* `private` top-level declarations are compiled to package-local declarations;
  `private` 最高層級宣告被編譯為 package-local 宣告；
* `protected` remains `protected` (note that Java allows accessing protected members from other classes in the same package and Kotlin doesn't, so Java classes will have broader access to the code);
  `protected` 保留 `protected` (注意： Java 允許從相同 package 其他的類別存取 `protected` 成員，而 Kotlin 不會，所以 Java 類別將更廣泛的存取代碼) ；
* `internal` declarations become `public` in Java. Members of `internal` classes go through name mangling, to make it harder to accidentally use them from Java and to allow overloading for members with the same signature that don't see each other according to Kotlin rules;
  `internal` 宣告在 Java 變成 `public` 。 `internal` 類別的成員經歷名稱粉碎，使它更困難的從 Java 意外使用到它們，並且根據 Kotlin 規則允許使用相同簽名的成員多載，這些成員不會彼此看到。
* `public` remains `public`.
  `public` 保留 `public` 。

## KClass

KClass ：KClass 在 Kotlin 的類別

Sometimes you need to call a Kotlin method with a parameter of type `KClass`. There is no automatic conversion from `Class` to `KClass`, so you have to do it manually by invoking the equivalent of the `Class<T>.kotlin` extension property:

有時你可能需要使用 `KClass` 類型的參數來調用 Kotlin 的方法。沒有任何從 `Class` 到 `KClass` 的自動轉換，所以你必須透過調用 `Class<T>.kotlin` 擴展屬性的對等物手動完成。

```kotlin
kotlin.jvm.JvmClassMappingKt.getKotlinClass(MainView.class)
```

## Handling signature clashes with @JvmName

Handling signature clashes with @JvmName ：使用 `@JvmName` 處理簽名的衝突

Sometimes we have a named function in Kotlin, for which we need a different JVM name the byte code. The most prominent example happens due to *type erasure*:

有時我們在 Kotlin 中有已命名的函數，我們為何需要不同 JVM 名稱的 byte code 。最突出的例子由於**類型消除**的發生：

``` kotlin
fun List<String>.filterValid(): List<String> // 泛型經過編譯後不會保留它的類型資訊
fun List<Int>.filterValid(): List<Int>
```

These two functions can not be defined side-by-side, because their JVM signatures are the same: `filterValid(Ljava/util/List;)Ljava/util/List;`. If we really want them to have the same name in Kotlin, we can annotate one (or both) of them with `@JvmName` and specify a different name as an argument:

這裡有兩個函數不可以一起定義，因為它們的 JVM 簽名是相同： `filterValid(Ljava/util/List;)Ljava/util/List;` 。如果我們真的想要它們在 Kotlin 有相同名稱，我們可以使用 `@JvmName` 註記它們的一個 (或兩個) 並指定不同的名稱作為參數：

``` kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValidInt") // 使用註記表示不同名稱
fun List<Int>.filterValid(): List<Int>
```

From Kotlin they will be accessible by the same name `filterValid`, but from Java it will be `filterValid` and `filterValidInt`.

從 Kotlin 透過相同名稱 `filterValid` 存取它們，但從 Java 它將是 `filterValid` 和 `filterValidInt` 。

The same trick applies when we need to have a property `x` alongside with a function `getX()`:

當你們需要有屬性 `x` 與函數 `getX()` 一起使用時，相同的技巧也適用：

``` kotlin
val x: Int
    @JvmName("getX_prop") // 定義新的 getX_prop 名稱來取得值
    get() = 15

fun getX() = 10 //在 Java 會有名稱衝突，所以上面的改為新名稱
```

To change the names of generated accessor methods for properties without explicitly implemented getters and setters, you can use `@get:JvmName` and `@set:JvmName`:

沒有明確已實作獲取屬性和設置屬性的情況下，改變生成屬性存取器方法 (getter/setter) 的名稱，你可以使用 `@get:JvmName` 和 `@set:JvmName` ：

``` kotlin
@get:JvmName("x")
@set:JvmName("changeX")
var x: Int = 23
```

## Overloads Generation

Normally, if you write a Kotlin function with default parameter values, it will be visible in Java only as a full
signature, with all parameters present. If you wish to expose multiple overloads to Java callers, you can use the
`@JvmOverloads` annotation.

The annotation also works for constructors, static methods etc. It can't be used on abstract methods, including methods
defined in interfaces.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class Foo @JvmOverloads constructor(x: Int, y: Double = 0.0) {
    @JvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") { ... }
}
```
</div>

For every parameter with a default value, this will generate one additional overload, which has this parameter and
all parameters to the right of it in the parameter list removed. In this example, the following will be
generated:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
// Constructors:
Foo(int x, double y)
Foo(int x)

// Methods
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```
</div>

Note that, as described in [Secondary Constructors](classes.html#secondary-constructors), if a class has default
values for all constructor parameters, a public no-argument constructor will be generated for it. This works even
if the `@JvmOverloads` annotation is not specified.


## Checked Exceptions

As we mentioned above, Kotlin does not have checked exceptions.
So, normally, the Java signatures of Kotlin functions do not declare exceptions thrown.
Thus if we have a function in Kotlin like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
// example.kt
package demo

fun foo() {
    throw IOException()
}
```
</div>

And we want to call it from Java and catch the exception:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
// Java
try {
  demo.Example.foo();
}
catch (IOException e) { // error: foo() does not declare IOException in the throws list
  // ...
}
```
</div>

we get an error message from the Java compiler, because `foo()` does not declare `IOException`.
To work around this problem, use the `@Throws` annotation in Kotlin:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@Throws(IOException::class)
fun foo() {
    throw IOException()
}
```
</div>

## Null-safety

When calling Kotlin functions from Java, nobody prevents us from passing *null*{: .keyword } as a non-null parameter.
That's why Kotlin generates runtime checks for all public functions that expect non-nulls.
This way we get a `NullPointerException` in the Java code immediately.

## Variant generics

When Kotlin classes make use of [declaration-site variance](generics.html#declaration-site-variance), there are two 
options of how their usages are seen from the Java code. Let's say we have the following class and two functions that use it:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```
</div>

A naive way of translating these functions into Java would be this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` java
Box<Derived> boxDerived(Derived value) { ... }
Base unboxBase(Box<Base> box) { ... }
```
</div>

The problem is that in Kotlin we can say `unboxBase(boxDerived("s"))`, but in Java that would be impossible, because in Java 
  the class `Box` is *invariant* in its parameter `T`, and thus `Box<Derived>` is not a subtype of `Box<Base>`. 
  To make it work in Java we'd have to define `unboxBase` as follows:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
Base unboxBase(Box<? extends Base> box) { ... }  
```
</div>

Here we make use of Java's *wildcards types* (`? extends Base`) to emulate declaration-site variance through use-site 
variance, because it is all Java has.

To make Kotlin APIs work in Java we generate `Box<Super>` as `Box<? extends Super>` for covariantly defined `Box` 
(or `Foo<? super Bar>` for contravariantly defined `Foo`) when it appears *as a parameter*. When it's a return value,
we don't generate wildcards, because otherwise Java clients will have to deal with them (and it's against the common 
Java coding style). Therefore, the functions from our example are actually translated as follows:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
// return type - no wildcards
Box<Derived> boxDerived(Derived value) { ... }

// parameter - wildcards 
Base unboxBase(Box<? extends Base> box) { ... }
```
</div>

NOTE: when the argument type is final, there's usually no point in generating the wildcard, so `Box<String>` is always
  `Box<String>`, no matter what position it takes.

If we need wildcards where they are not generated by default, we can use the `@JvmWildcard` annotation:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)
// is translated to 
// Box<? extends Derived> boxDerived(Derived value) { ... }
```
</div>

On the other hand, if we don't need wildcards where they are generated, we can use `@JvmSuppressWildcards`:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
// is translated to 
// Base unboxBase(Box<Base> box) { ... }
```
</div>

NOTE: `@JvmSuppressWildcards` can be used not only on individual type arguments, but on entire declarations, such as 
functions or classes, causing all wildcards inside them to be suppressed.

### Translation of type Nothing

The type [`Nothing`](exceptions.html#the-nothing-type) is special, because it has no natural counterpart in Java. Indeed, every Java reference type, including
`java.lang.Void`, accepts `null` as a value, and `Nothing` doesn't accept even that. So, this type cannot be accurately
represented in the Java world. This is why Kotlin generates a raw type where an argument of type `Nothing` is used:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun emptyList(): List<Nothing> = listOf()
// is translated to
// List emptyList() { ... }
```
</div>
