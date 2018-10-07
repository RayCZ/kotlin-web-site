---
type: doc
layout: reference
category: "Syntax"
title: "Object Expressions, Object Declarations and Companion Objects"
---

# Object Expressions and Declarations

Object Expressions and Declarations ：物件表達式和宣告

Sometimes we need to create an object of a slight modification of some class, without explicitly declaring a new subclass for it. Java handles this case with *anonymous inner classes*. Kotlin slightly generalizes this concept with *object expressions* and *object declarations*.

有時我們需要創建一個稍微修改某類別的物件，不需要對物件明確宣告一個新類別。 Java 使用匿名內部類別處理這種情況。 Kotlin 使用物件表達式和物件宣告稍微概括這個概含。

## Object expressions

Object expressions ：物件表達式

To create an object of an anonymous class that inherits from some type (or types), we write:

要創建從某些類型 (或多類型) 繼承的匿名類別物件 `object : MouseAdapter()`，我們寫：

``` kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
})
```

If a supertype has a constructor, appropriate constructor parameters must be passed to it. Many supertypes may be specified as a comma-separated list after the colon:

如果一個超 (父) 類型有建構元，必須將適當的建構元參數傳遞給它 `A(1)` 。在冒號之後很多超 (父) 類型可以指定為逗號-分隔列表 `object : A(1), B` 。

``` kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B { ... }

val ab: A = object : A(1), B {
    override val y = 15
}
```

If, by any chance, we need "just an object", with no nontrivial supertypes, we can simply say:

如果，任何機會，我們需要 "只是一個物件" ，沒有非凡的超類型，我們可以簡單的說 `object {....}`：

``` kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
```

Note that anonymous objects can be used as types only in local and private declarations. If you use an anonymous object as a return type of a public function or the type of a public property, the actual type of that function or property will be the declared supertype of the anonymous object, or `Any` if you didn't declare any supertype. Members added in the anonymous object will not be accessible.

注意：匿名物件只在區域或私有宣告用作類型 `object{...}`。如果你使用匿名物件為一個公開函數的回傳類型或一個公開屬性，函數或屬性的真實類型將被宣告匿名物件的超類型，或如果你未宣告任何超類型為 `Any` ，在匿名物件新增成員將不可存取。

``` kotlin
class C {
    // Private function, so the return type is the anonymous object type
    private fun foo() = object {
        val x: String = "x"
    }

    // Public function, so the return type is Any
    fun publicFoo() = object {
        val x: String = "x"
    }
    
    fun bar() {
        val x1 = foo().x        // Works
        val x2 = publicFoo().x  // ERROR: Unresolved reference 'x'
    }
}
```

Just like Java's anonymous inner classes, code in object expressions can access variables from the enclosing scope. (Unlike Java, this is not restricted to final variables.)

就像 Java 匿名內部類別，在物件表達式的代碼可以從閉包的範圍存取變數。 (不像 Java ，這是被限制為 `final` 變數。)

``` kotlin
fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0

    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
    
        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // ...
}
```

## Object declarations

Object declarations ：物件宣告

[Singleton](http://en.wikipedia.org/wiki/Singleton_pattern) may be useful in several cases, and Kotlin (after Scala) makes it easy to declare singletons:

單例模式在某些情況下有用的，而 Kotlin (在 Scala 之後) 使它容易宣告單例模式：

``` kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

This is called an *object declaration*, and it always has a name following the *object* keyword. Just like a variable declaration, an object declaration is not an expression, and cannot be used on the right hand side of an assignment statement.

這稱為物件宣告，並且物件宣告總是在 `object` 關鍵字之後有名稱。就像一個變數宣告，物件宣告不是一個表達式，也不可以用在賦值敘述的右手邊。

Object declaration's initialization is thread-safe.

物件宣告初始化是線程安全。

To refer to the object, we use its name directly:

參照到該物件，我們直接使用它的名稱：

``` kotlin
DataProviderManager.registerDataProvider(...)
```

Such objects can have supertypes:

這物件可以有超 (父) 類型：

``` kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
}
```

**NOTE**: object declarations can't be local (i.e. be nested directly inside a function), but they can be nested into other object declarations or non-inner classes.

注意：物件宣告不能在區域 (換句話說，在一個函數內直接嵌入) ，但它們可以內嵌到其他物件宣告或非內部類別。

### Companion Objects

Companion Objects ：夥伴物件

An object declaration inside a class can be marked with the *companion* keyword:

可以使用 `companion` 關鍵字標記在一個類別中的物件宣告：

``` kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

Members of the companion object can be called by using simply the class name as the qualifier:

透過簡單的類別名稱為修飾符調用夥伴物件的成員：

``` kotlin
val instance = MyClass.create()
```

The name of the companion object can be omitted, in which case the name `Companion` will be used:

夥伴物件的名稱可以被省略，在這種情況下將使用名稱 `Companion` ：

``` kotlin
class MyClass {
    companion object { }
}

val x = MyClass.Companion
```

Note that, even though the members of companion objects look like static members in other languages, at runtime those
are still instance members of real objects, and can, for example, implement interfaces:

請注意：即使夥伴物件的成員看起來像在其他語言的靜態成員，在運行時它們仍然還是真實物件的實例成員，並且可以，例如，實作介面：

``` kotlin
interface Factory<T> {
    fun create(): T
}

class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
```

However, on the JVM you can have members of companion objects generated as real static methods and fields, if you use the `@JvmStatic` annotation. See the [Java interoperability](java-to-kotlin-interop.md#static-fields) section for more details.

然而，在 JVM 上，如果你使用 `@JvmStatic` 註解，你可以讓夥伴物件的成員生成真實靜態方法和欄位，更多細節請參閱 [Java interoperability](java-to-kotlin-interop.md#static-fields) 章節。

### Semantic difference between object expressions and declarations

Semantic difference between object expressions and declarations ：物件表達式和宣告之間的語義差異

There is one important semantic difference between object expressions and object declarations:

物件表達式和物件宣告之間存在一個重要的語義差異：

* object expressions are executed (and initialized) **immediately**, where they are used;
  物件表達式立即執行 (和初始化) 的，使用它們時；
* object declarations are initialized **lazily**, when accessed for the first time;
  物件宣告在第一次存取時被懶惰的初始化；
* a companion object is initialized when the corresponding class is loaded (resolved), matching the semantics of a Java static initializer.
  當讀取 (解析) 對應的類別時，初始化夥伴物件，匹配 Java 靜態初始化的語義