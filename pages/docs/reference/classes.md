---
type: doc
layout: reference
category: "Syntax"
title: "Classes and Inheritance"
related:
    - functions.md
    - nested-classes.md
    - interfaces.md
---

# Classes and Inheritance

Classes and Inheritance ：類別和繼承

## Classes

Classes ：類別

Classes in Kotlin are declared using the keyword *class*:

在 Kotlin 的類別使用關鍵字 `class` 宣告：

``` kotlin
class Invoice { ... }
```
The class declaration consists of the class name, the class header (specifying its type parameters, the primary constructor etc.) and the class body, surrounded by curly braces. Both the header and the body are optional; if the class has no body, curly braces can be omitted.

類別宣告由類別名稱、類別標頭 (指定它的類型參數、主建構元等等) 、類別內文所組成，內文由大括號圍繞，標頭和內文是可選的 (可填或不填) ；如果類別沒有內文，大括號可以被省略

``` kotlin
class Empty
```
---

### Constructors

Constructors ：建構元

A class in Kotlin can have a **primary constructor** and one or more **secondary constructors**. The primary
constructor is part of the class header: it goes after the class name (and optional type parameters).

在 Kotlin 的類別可以有主建構元和一個或更多個次要建構元，主建構元是類別標頭的一部分；在類別名之後 (和可選的類型參數)

``` kotlin
class Person constructor(firstName: String) { ... }
```
If the primary constructor does not have any annotations or visibility modifiers, the *constructor* keyword can be omitted:

如果主建構元不會有任何註釋 `@Nonnull` 或可見性 `public` 的修飾符，`constructor` 關建字可以被省略

``` kotlin
class Person(firstName: String) { ... }
```
The primary constructor cannot contain any code. Initialization code can be placed in **initializer blocks**, which are prefixed with the *init* keyword.

主建構元不可以包含任何執行代碼，初始化代碼可以被放到初始化區塊，使用 `init` 關鍵字前綴修飾

During an instance initialization, the initializer blocks are executed in the same order as they appear in the class body, interleaved with the property initializers:

在實例初始化期間，初始化區塊的執行與下面類別內文出現的順序相同，與屬性初始化交錯執行

``` kotlin
//sampleStart
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints ${name}")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}
//sampleEnd

fun main(args: Array<String>) {
    InitOrderDemo("hello")
}

//ans:
//First property: hello
//First initializer block that prints hello
//Second property: 5
//Second initializer block that prints 5
```
Note that parameters of the primary constructor can be used in the initializer blocks. They can also be used in property initializers declared in the class body:

請注意：主建構元的參數可以被用在初始化區塊中，它們也可以用在類別內文中宣告屬性初始化

``` kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```
In fact, for declaring properties and initializing them from the primary constructor, Kotlin has a concise syntax:

實際上，對於來自主建構元宣告屬性和初始化它們， Kotlin 有一個簡潔的句法
``` kotlin
class Person(val firstName: String, val lastName: String, var age: Int) { ... }
```
Much the same way as regular properties, the properties declared in the primary constructor can be
mutable (*var*) or read-only (*val*).

與常規屬性大致相同的方式，在建構元宣告屬性可以為可變的 (`var`) 或 唯讀的 (`val`)

If the constructor has annotations or visibility modifiers, the *constructor* keyword is required, and the modifiers go before it:

如果建構元有註釋或可見性修飾符， 必須要有 `constructor` 關鍵字， 並且註釋或修飾符在 `constructor` 前面

``` kotlin
class Customer public @Inject constructor(name: String) { ... }
```
For more details, see [Visibility Modifiers](visibility-modifiers.md#constructors).

更多細節，看參閱 [Visibility Modifiers](visibility-modifiers.md#constructors)

---

#### Secondary Constructors

Secondary Constructors ：次要建構元

The class can also declare **secondary constructors**, which are prefixed with *constructor*:

類別也可以宣告次要建構元，使用 `constructor` 為前綴
``` kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```
If the class has a primary constructor, each secondary constructor needs to delegate to the primary constructor, either directly or indirectly through another secondary constructor(s). Delegation to another constructor of the same class is done using the *this* keyword:

如果類別已有主建構元，每個次要建構元需要調用主建構元，用直接或間接方式通過另一個次要建構元調用主建構元 `: this(name)`，使用 `this` 完成相同類別的另一個建構元調用
``` kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```
Note that code in initializer blocks effectively becomes part of the primary constructor. Delegation to the primary constructor happens as the first statement of a secondary constructor, so the code in all initializer blocks is executed before the secondary constructor body. Even if the class has no primary constructor, the delegation still happens implicitly, and the initializer blocks are still executed:

注意：在初始化區塊的該代碼有效的成為主建構元的一部分，調用主建構元發生在次要建構元的一個敘述，所以在執行次要建構元內文之前先執行所有在初始化區塊的代碼，即使類別沒有主建構元，調用還是會隱性發生，並且初始化區塊仍然被執行

``` kotlin
//sampleStart
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
//sampleEnd

fun main(args: Array<String>) {
    Constructors(1)
}

//ans:
//Init block
//Constructor
```
If a non-abstract class does not declare any constructors (primary or secondary), it will have a generated primary constructor with no arguments. The visibility of the constructor will be public. If you do not want your class to have a public constructor, you need to declare an empty primary constructor with non-default visibility:

如果一個非抽象類別沒有宣告任何建構元 (主要或次要)，它將自動生成一個沒有參數的主建構元，建構元的可見性將為公開的，如果你不想要你的類別有公開建構元，你需要宣告一個空的、非預設可見性建構元
``` kotlin
class DontCreateMe private constructor () { ... }
```
> **NOTE**: On the JVM, if all of the parameters of the primary constructor have default values, the compiler will generate an additional parameterless constructor which will use the default values. This makes it easier to use Kotlin with libraries such as Jackson or JPA that create class instances through parameterless constructors.
>
> **注意**：在 JVM 上，如果主建構元的所有參數有預設值，編譯器將生成額外無參數的建構元，無參數建構元將使用預設值，這使它更容易去使用 Kotlin的函式庫，例如： Jackson 、 JPA ，函式庫通過無參數建構元建立類別實例
>
> ```kotlin
> class Customer(val customerName: String = "")
> ```

---

### Creating instances of classes

Creating instances of classes ：建立類別的實例

To create an instance of a class, we call the constructor as if it were a regular function:

為了建立一個類別的實例，我們可以調用建構元，如果建構元是一個常規函數：
``` kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```
Note that Kotlin does not have a *new* keyword.

注意： Kotlin 不會有 `new` 關鍵字

Creating instances of nested, inner and anonymous inner classes is described in [Nested classes](nested-classes.md).

創建內嵌、內部、匿名內部類別的實例描述在 [Nested classes](nested-classes.md)

---

### Class Members

Class Members ：類別成員

Classes can contain:

類別可以包含：

* [Constructors and initializer blocks](classes.md#constructors)
  建構元和初始化區塊
* [Functions](functions.md)
  函數
* [Properties](properties.md)
  屬性
* [Nested and Inner Classes](nested-classes.md)
  內嵌和內部類別
* [Object Declarations](object-declarations.md)
  物件宣告


## Inheritance

Inheritance ：繼承

All classes in Kotlin have a common superclass `Any`, that is the default superclass for a class with no supertypes declared:

在 Kotlin 的所有類別有共同的超 (父) 類別 `Any` ，對於一個類別沒有宣告超 (父) 類型， `Any` 類別是預設超 (父) 類別
``` kotlin
class Example // Implicitly inherits from Any
```
> Note: `Any` is not `java.lang.Object`; in particular, it does not have any members other than `equals()`, `hashCode()` and `toString()`. Please consult the [Java interoperability](java-interop.html#object-methods) section for more details.
>
> 注意： `Any` 不是 `java.lang.Object` ；特別是，它沒有任何成員除了 `equals()` 、 `hashCode()` 、 `toString()` ，更多細節請參閱 [Java interoperability](java-interop.html#object-methods) 章節

To declare an explicit supertype, we place the type after a colon in the class header:

去宣告明顯的超 (父) 類型，在類別標頭的冒號之後放置類型
``` kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```
> The *open* annotation on a class is the opposite of Java's *final*: it allows others
> to inherit from this class. By default, all classes in Kotlin are final, which corresponds to [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 19: *Design and document for inheritance or else prohibit it*.
>
> 在類別 `open` 註釋與 Java 的 final 相反：它允許從這個類別讓其他類別去繼承，預值下，在 Kotlin 所有類別是 `final` ，對應於 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html) , Item 19: *Design and document for inheritance or else prohibit it*

If the derived class has a primary constructor, the base class can (and must) be initialized right there,
using the parameters of the primary constructor.

如果衍生 (子) 類別已有主建構元，基礎類別可以 (且必須) 在主建構元裡初始化，並且使用主建構元的參數

If the class has no primary constructor, then each secondary constructor has to initialize the base type
using the *super* keyword, or to delegate to another constructor which does that.Note that in this case different secondary constructors can call different constructors of the base type:

如果類別沒有主建構元，接著每個次要建構元必須初始化基礎 (父) 類型使用 `super` 關鍵字，或去調用其他建構元做該件事，注意：在這個情況，不同建構元可以各自調用不同的基礎 (父) 類型的建構元：

**建構元各自調用父類別建構元 `super(ctx)` 或 ``super(ctx, attrs)`**

``` kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```
---

### Overriding Methods

Overriding Methods ：覆寫方法

As we mentioned before, we stick to making things explicit in Kotlin. And unlike Java, Kotlin requires explicit
annotations for overridable members (we call them *open*) and for overrides:

如我們之前所述，我們堅持在 Kotlin 中明確表達做的事情，並且不像 Java ， Kotlin需要明確註釋為可覆寫的成員 (我們調用它們需要有 `open`) 並衍生 (子) 類別註釋 `override`

``` kotlin
open class Base {
    open fun v() { ... }
    fun nv() { ... }
}
class Derived() : Base() {
    override fun v() { ... }
}
```
The *override* annotation is required for `Derived.v()`. If it were missing, the compiler would complain. If there is no *open* annotation on a function, like `Base.nv()`, declaring a method with the same signature in a subclass is illegal, either with *override* or without it. In a final class (e.g. a class with no *open* annotation), open members are prohibited.

**Function signature：函數簽名，包括函數的名稱、參數順序、參數類型、泛型欄位等資訊總稱**

`Derived.v()` 需要註釋 `override` - `override fun v() { ... }`，如果 `override` 被遺忘，編譯器可能會抱怨，如果在函數沒有 `open` 註釋，像 `Base.nv()` 在子類別宣告一個方法使用相同簽名是非法的，無論是使用 `override` 或不使用它，在一個 `final` 類別 (例如：沒有 `open` 註釋的類別) ，公開的成員被禁用

A member marked *override* is itself open, i.e. it may be overridden in subclasses. If you want to prohibit re-overriding, use *final*:

一個成員被標記 `override` 代表它本身是 `open` ，即它在子類別可以被覆寫，如果你想要禁用 `re-override` ，使用 `final` ：
``` kotlin
open class AnotherDerived() : Base() {
    final override fun v() { ... }
}
```
---

### Overriding Properties 

Overriding Properties ：覆寫屬性

Overriding properties works in a similar way to overriding methods; properties declared on a superclass that are then redeclared on a derived class must be prefaced with *override*, and they must have a compatible type. Each declared property can be overridden by a property with an initializer or by a property with a getter method.

覆寫屬性的工作與覆寫方法類似；在超 (父) 類別宣告屬性接著在衍生 (子) 類別重新宣告屬性，必須以 `override` 為宣告的開始，並且它們必須有兼容類型，每個被宣告的屬性可以由衍生 (子) 類別初始化屬性或屬性的 getter 方法覆寫

``` kotlin
open class Foo {
    open val x: Int get() { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```
You can also override a `val` property with a `var` property, but not vice versa. This is allowed because a `val` property essentially declares a getter method, and overriding it as a `var` additionally declares a setter method in the derived class.

你也可以使用 `var` 屬性覆寫一個 `val` 屬性，但不可以反過來，這是允許的，因為一個 `val` 屬性本質上已宣告一個 getter 方法，並且在衍生 (子) 類別覆寫屬性為 `var` ，額外多宣告一個 setter 方法

Note that you can use the *override* keyword as part of the property declaration in a primary constructor.

注意：在主建構元你可以使用 `override` 關鍵字為屬性宣告的一部分
``` kotlin 
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```
---

### Derived class initialization order

Derived class initialization order ：衍生類別初始化順序

During construction of a new instance of a derived class, the base class initialization is done as the first step (preceded only by evaluation of the arguments for the base class constructor) and thus happens before the initialization logic of the derived class is run. 

在衍生 (子) 類別創建實例的建構期間，基礎 (父) 類別初始化為第一步完成 (只在基礎 (父) 類別建構元參數執行之前) ，因此在衍生 (子) 類別的初始化邏輯運行之前發生

``` kotlin
//sampleStart
open class Base(val name: String) {

    init { println("Initializing Base") }

    open val size: Int = 
        name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {

    init { println("Initializing Derived") }

    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }
}
//sampleEnd

fun main(args: Array<String>) {
    println("Constructing Derived(\"hello\", \"world\")")
    val d = Derived("hello", "world")
}

//ans:
//Constructing Derived("hello", "world")
//Argument for Base: Hello
//Initializing Base
//Initializing size in Base: 5
//Initializing Derived
//Initializing size in Derived: 10
```
**執行順序：父類別建構元參數 > 父類別初始化區塊 > 父類別屬性 > 子類別初始化區塊 > 子類別屬性**

It means that, by the time of the base class constructor execution, the properties declared or overridden in the derived class are not yet initialized. If any of those properties are used in the base class initialization logic (either directly or indirectly, through another overridden *open* member implementation), it may lead to incorrect behavior or a runtime failure. When designing a base class, you should therefore avoid using *open* members in the constructors, property initializers, and *init* blocks.

這意味著，在基礎 (父) 類別建構元執行時，在衍生 (子) 類別宣告或覆寫屬性還沒被初始化，如果在基礎 (父) 類別初始化邏輯中使用任何這些屬性 (直接或是間接，透過另一個覆寫 `open` 修飾的成員實作) ， 它可能導致不正確的行為或運行時期失敗，因此當我們設計一個基礎 (父) 類別時，你應該避免在建構元、屬性初始化、`init` 區塊使用修飾為 `open` 的成員

---

### Calling the superclass implementation

Calling the superclass implementation ：調用超 (父) 類別實作

Code in a derived class can call its superclass functions and property accessors implementations using the *super* keyword:

在衍生 (子) 類別可以使用 `super` 調用它的超 (父) 類別函數或屬性存取器 (setter/getter) 實作：


``` kotlin
open class Foo {
    open fun f() { println("Foo.f()") }
    open val x: Int get() = 1
}
```

``` kotlin
class Bar : Foo() {
    override fun f() { 
        super.f()
        println("Bar.f()") 
    }
    
    override val x: Int get() = super.x + 1
}
```

Inside an inner class, accessing the superclass of the outer class is done with the *super* keyword qualified with the outer class name: `super@Outer`:

在內部類別內，存取外部類別的超 (父) 類別，使用外部類別名稱修飾 `super` 關鍵字來完成 `super@Outer` ：

``` kotlin
class Bar : Foo() {
    override fun f() { /* ... */ }
    override val x: Int get() = 0
    
    inner class Baz {
        fun g() {
            super@Bar.f() // Calls Foo's implementation of f()
            println(super@Bar.x) // Uses Foo's implementation of x's getter
        }
    }
}
```

---

### Overriding Rules

Overriding Rules ：覆寫規則

In Kotlin, implementation inheritance is regulated by the following rule: if a class inherits many implementations of the same member from its immediate superclasses,it must override this member and provide its own implementation (perhaps, using one of the inherited ones).To denote the supertype from which the inherited implementation is taken, we use *super* qualified by the supertype name in angle brackets, e.g. `super<Base>`:

member ：成員，包括屬性與函數

在 Kotlin 中，實作繼承受以下規則約束：如果一個類別從它的直屬超 (父) 類別，繼承相同函數的多個實作，它必須覆寫這個函數並且提供它擁有的實作 (可能使用其中一個繼承的實作)，為了表示超 (父) 類型，帶入繼承實作的超 (父) 類型，我們在尖括號使用超 (父) 類型名稱修飾 `super` ，既是： `super<Base>`

``` kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // interface members are 'open' by default
    fun b() { print("b") }
}

class C() : A(), B {
    // The compiler requires f() to be overridden:
    override fun f() {
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
```

It's fine to inherit from both `A` and `B`, and we have no problems with `a()` and `b()` since `C` inherits only one implementation of each of these functions. But for `f()` we have two implementations inherited by `C`, and thus we have to override `f()` in `C` and provide our own implementation that eliminates the ambiguity.

繼承 `A` 和 `B` 都沒關係，並且使用函數 `a()` 和 `b()` 都沒有問題，因為 `C` 只繼承這些函數的一個實作，但對於 `f()` 我們由 `C` 繼承兩個實作，因為我們在 `C` 必需覆寫 `f()` 並且提供我們擁有的實作消除分歧

## Abstract Classes

Abstract Classes ：抽象類別

A class and some of its members may be declared *abstract*. An abstract member does not have an implementation in its class. Note that we do not need to annotate an abstract class or function with open – it goes without saying.

一個類別和它的一些成員可以被宣告為  `abstract` ，一個抽象成員在它的類別本身不會有有實作，注意：我們不需要使用 `open` 來註釋抽象類別或函數 - 不可而喻 (抽象代表已經是要去實作了)

We can override a non-abstract open member with an abstract one

我們可以使用 `abstract` 再次覆寫一個非抽象 `open` 的成員

``` kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## Companion Objects

Companion Objects ：夥伴物件，類似於 Java 的 static

In Kotlin, unlike Java or C#, classes do not have static methods. In most cases, it's recommended to simply use package-level functions instead.

在 Kotlin 中，不像 Java 或 C# ，類別不需要靜態方法，在大多情況下，推薦簡單地使用 package-level 函數取代靜態

If you need to write a function that can be called without having a class instance but needs access to the internals of a class (for example, a factory method), you can write it as a member of an [object declaration](object-declarations.md)
inside that class.

如果你需要寫一個可以被調用沒有類別實例的函數，但需要存取類別的內部 (例如：工廠方法) ，你可以在類別內編寫為一個 [object 宣告](object-declarations.md)的成員

Even more specifically, if you declare a [companion object](object-declarations.md#companion-objects) inside your class, you'll be able to call its members with the same syntax as calling static methods in Java/C#, using only the class name as a qualifier.

更具體來說，如果在你的類別內宣告一個 [夥伴物件](object-declarations.md#companion-objects) ，你將能調用類別內的成員，與在 Java/C# 中調用靜態方法相同的句法