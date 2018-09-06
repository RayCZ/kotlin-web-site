---
type: doc
layout: reference
category: "Syntax"
title: "Properties and Fields: Getters, Setters, const, lateinit"
---

# Properties and Fields

Properties and Fields  ：屬性與欄位

## Declaring Properties

Declaring Properties ：宣告屬性

Classes in Kotlin can have properties. These can be declared as mutable, using the *var* keyword or read-only using the *val* keyword.

在 Kotlin 中的類別可以有屬性。這些可以被宣告為可變的，使用 `var` 關鍵字或唯讀使用 `val` 關鍵字。

``` kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String? = ...
    var zip: String = ...
}
```

To use a property, we simply refer to it by name, as if it were a field in Java:

要使用屬性，我們只需要透過名稱引用它，就像它是 Java 中的欄位一樣：

``` kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // there's no 'new' keyword in Kotlin
    result.name = address.name // accessors are called
    result.street = address.street
    // ...
    return result
}
```

## Getters and Setters

Getters and Setters ：獲取屬性和設置屬性

Accessor ：存取器，為獲取屬性與設置屬性的集合

The full syntax for declaring a property is

宣告屬性的完整語法

``` kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

The initializer, getter and setter are optional. Property type is optional if it can be inferred from the initializer (or from the getter return type, as shown below).

初始化、獲取屬性、設置屬性是可選的。如果可以從初始化值 (或從獲取屬性回傳類型，如下所示) 推斷屬性類型，代表屬性類型是可選的。

Examples:

範例：

``` kotlin
var allByDefault: Int? // error: explicit initializer required, default getter and setter implied
var initialized = 1 // has type Int, default getter and setter
```

The full syntax of a read-only property declaration differs from a mutable one in two ways: it starts with `val` instead of `var` and does not allow a setter:

在兩種屬性宣告方式，唯讀屬性宣告的完整語法不同於可變屬性宣告，唯讀屬性開頭 `val` 代替 `var` 並且不允許設置屬性

``` kotlin
val simple: Int? // has type Int, default getter, must be initialized in constructor
val inferredType = 1 // has type Int and a default getter
```

We can write custom accessors, very much like ordinary functions, right inside a property declaration. Here's an example of a custom getter:

我們可以在屬性宣告之後，編寫自定義的存取器，非常像普通的方法。以下是自定義獲取屬性的範例：

``` kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

A custom setter looks like this:

一個自定義設置屬性看起來像：

``` kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```

By convention, the name of the setter parameter is `value`, but you can choose a different name if you prefer.

按照慣例，設置屬性參數的名稱是 `value`　，但如果你有偏好，你可以選擇不同名稱。

Since Kotlin 1.1, you can omit the property type if it can be inferred from the getter:

從 Kotlin 1.1 版本，如果你可以從獲取屬性推斷屬性類型，你可以省略屬性類型宣告：

``` kotlin
val isEmpty get() = this.size == 0  // has type Boolean
```

If you need to change the visibility of an accessor or to annotate it, but don't need to change the default implementation, you can define the accessor without defining its body:

如果你需要改變存取器的可見性或註釋它，但不需要改變預設的實作，你可以只定義存取器且存取器沒有內文

``` kotlin
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject

```

---

### Backing Fields

Backing Fields ：支援欄位，欄位代表的是屬性本身

請參閱 https://stackoverflow.com/questions/47678991/kotlin-explain-me-about-backing-fields

Fields cannot be declared directly in Kotlin classes. However, when a property needs a backing field, Kotlin provides it automatically. This backing field can be referenced in the accessors using the `field` identifier:

在 Kotlin 類別內不可以直接宣告欄位。然後，當有一個屬性需要一個支援欄位時， Kotlin自動提供支援欄位。這個支援欄位在存取器 (設置屬性/獲取屬性) 使用 `field` 識別並可以引用到支援欄位

``` kotlin
var counter = 0 // Note: the initializer assigns the backing field directly
    set(value) {
        if (value >= 0) field = value
    }
```

The `field` identifier can only be used in the accessors of the property.

`field` 識別只可以用在屬性的存取器。

A backing field will be generated for a property if it uses the default implementation of at least one of the accessors, or if a custom accessor references it through the `field` identifier.

如果屬性使用至少一個存取器並預設實作，或如果自定義存取器引用到屬性並透過 `field` 識別，屬性將產生支援欄位。

For example, in the following case there will be no backing field:

範例，在以下情況將沒有支援欄位：

``` kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

---

### Backing Properties

Backing Properties ：支援屬性

If you want to do something that does not fit into this "implicit backing field" scheme, you can always fall back to having a *backing property*:

如果你想要做一些不適合這種 " 隱式支援欄位 " 計劃，你總可以回到有支援屬性的方式：

``` kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

In all respects, this is just the same as in Java since access to private properties with default getters and setters is optimized so that no function call overhead is introduced.

在所有方面，這只是與 Java 相同方式，因為使用預設優化的獲取屬性與設置屬性存取私有屬性，因此不會引入函數調用的開銷。

## Compile-Time Constants

Compile-Time Constants ：編譯時期的常數

Properties the value of which is known at compile time can be marked as _compile time constants_ using the `const` modifier. Such properties need to fulfil the following requirements:

 編譯時期的常數使用 ` const` 修飾符標記為在編譯時期已知屬性的值。這樣屬性需要滿足以下需求：

  * Top-level or member of an `object`
    最高層級或物件成員
  * Initialized with a value of type `String` or a primitive type
    使用 `String` 類型或原生類型的值初始化
  * No custom getter
    沒有自定義獲取屬性

Such properties can be used in annotations:

這樣的屬性可以使用在註釋：

``` kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

## Late-Initialized Properties and Variables

Late-Initialized Properties and Variables ：延遲初始化的屬性與變數

Normally, properties declared as having a non-null type must be initialized in the constructor. However, fairly often this is not convenient. For example, properties can be initialized through dependency injection, or in the setup method of a unit test. In this case, you cannot supply a non-null initializer in the constructor, but you still want to avoid null checks when referencing the property inside the body of a class.

通常，屬性宣告為有非空的類型 `String?` 必須在建構元初始化。然而，這通常是相當的不方便。例如：屬性可以透過依賴注入初始化，或在單元測試的設置方法。在這樣的情況下，你不可以在建構元提供一個非空的初始化，但你仍想要避免可空的檢查在你需要在類別內中引用到屬性時。

To handle this case, you can mark the property with the `lateinit` modifier:

為了處理這種情況，你可以標記屬性使用 `lateinit` 修飾符：

``` kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }
    
    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```

The modifier can be used on `var` properties declared inside the body of a class (not in the primary constructor, and only when the property does not have a custom getter or setter) and, since Kotlin 1.2, for top-level properties and local variables. The type of the property or variable must be non-null, and it must not be a primitive type.

`lateinit` 可以用在類別的內文中用於 `var` 屬性 (不在主建構元，並只能當屬性沒有自定義的獲取屬性與設置屬性) ，從 Kotlin 1.2 版支援最高層級屬性與區域變數。屬性或變數類型必須是非空的，且屬性必須不是原生類型。

Accessing a `lateinit` property before it has been initialized throws a special exception that clearly identifies the property being accessed and the fact that it hasn't been initialized.

在屬性被初始化之前，存取一個 `lateinit` 屬性會丟出特殊的異常，異常會清楚的識別屬性正在被存取，以及屬性未被初始化的事實。

---

### Checking whether a lateinit var is initialized (since 1.2)

Checking whether a lateinit var is initialized (since 1.2) ：檢查 lateinit var 是否已經被初始化 (從 1.2 版)

To check whether a `lateinit var` has already been initialized, use `.isInitialized` on the [reference to that property](reflection.md#property-references):

檢查 `lateinit var` 是否已經被初始化，[在引用屬性時](reflection.md#property-references)使用 `.isInitialized`

```kotlin
if (foo::bar.isInitialized) {
    println(foo.bar)
}
```

This check is only available for the properties that are lexically accessible, i.e. declared in the same type or in one of the outer types, or at top level in the same file.

這個檢查只可用於屬性是詞彙存取，即是相同類型或外部類型之一的宣告，或在相同檔案的最高層級。

## Overriding Properties

Overriding Properties ：覆寫屬性

See [Overriding Properties](classes.md#overriding-properties)

參閱 [Overriding Properties](classes.md#overriding-properties)

## Delegated Properties

Delegated Properties ：委派屬性

The most common kind of properties simply reads from (and maybe writes to) a backing field. On the other hand, with custom getters and setters one can implement any behaviour of a property. Somewhere in between, there are certain common patterns of how a property may work. A few examples: lazy values, reading from a map by a given key, accessing a database, notifying listener on access, etc.

最常見的屬性只是從一個支援欄位讀取 (或許是寫入) 。另一方面使用自定義的獲取屬性和設置屬性之一可以實作屬性的任何行為。介於兩者之間，有一些某些常見的樣式關於屬性如何運作。一些範例：惰性值、透過 key 從map取值，存取資料庫、在存取時通知監聽者等等。

Such common behaviours can be implemented as libraries using [_delegated properties_](delegated-properties.md).

這些常見的行為可以被實作在函數庫使用 [_delegated properties_](delegated-properties.md)。