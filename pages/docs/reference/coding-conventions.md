---
type: doc
layout: reference
category: Basics
title: Coding Conventions
---

# Coding Conventions (編碼慣例)

This page contains the current coding style for the Kotlin language.

這頁包含目前 Kotlin 語言的編碼風格

* [Source code organization](#source-code-organization)
* [Naming rules](#naming-rules)
* [Formatting](#formatting)
* [Documentation comments](#documentation-comments)
* [Avoiding redundant constructs](#avoiding-redundant-constructs)
* [Idiomatic use of language features](#idiomatic-use-of-language-features)
* [Coding conventions for libraries](#coding-conventions-for-libraries)

### Applying the style guide (應用風格指南)

To configure the IntelliJ formatter according to this style guide, please install Kotlin plugin version
1.2.20 or newer, go to Settings | Editor | Code Style | Kotlin, click on "Set from..." link in the upper
right corner, and select "Predefined style / Kotlin style guide" from the menu.

根據這份風格指南配置 IntelliJ 格式化，請安裝 Kotlin 外掛版本 1.2.20 或更新，到 Settings | Editor | Code Style | Kotlin, click on "Set from..." 右上角的連結，並且選單中選擇 "Predefined style / Kotlin style guide" 

To verify that your code is formatted according to the style guide, go to the inspection settings and enable
the "Kotlin | Style issues | File is not formatted according to project settings" inspection. Additional
inspections that verify other issues described in the style guide (such as naming conventions) are enabled by default.

根據這份風格指南檢驗你的代碼，去啟用 "Settings | Editor | inspections | Kotlin | Style issues | File is not formatted according to project settings" 檢查，預設情況下，在風格指南額外檢查描述校驗其他問題(例如命名慣例)

## Source code organization (來源代碼組織)

### Directory structure (目錄結構)

In mixed-language projects, Kotlin source files should reside in the same source root as the Java source files,
and follow the same directory structure (each file should be stored in the directory corresponding to each package statement).

在混合語言專案中，Kotlin 來源檔案與 Java 來源檔案應該位於相同起始(根)來源，並遵循相同目錄結構 (每個檔案應該儲存在對應每個 package 描述的目錄)

In pure Kotlin projects, the recommended directory structure is to follow the package structure with
the common root package omitted

在單一(純) Kotlin 專案中，推薦目錄結構是遵循 package 結構，使用上通用的起始(根) package 被省略

(e.g. if all the code in the project is in the "org.example.kotlin" package and its subpackages, files with the "org.example.kotlin" package should be placed directly under the source root, and files in "org.example.kotlin.foo.bar" should be in the "foo/bar" subdirectory of the source root).

(例如：如果在專案中所有的代碼在 "org.example.kotlin" package 和它的子 packages，檔案內宣告 package "org.example.kotlin" 應該直接放置在起始(根)來源下，並在 "org.example.kotlin.foo.bar"下的檔案應該在起始來源的子目錄 "foo/bar")

---

### Source file names (來源檔案命名)

If a Kotlin file contains a single class (potentially with related top-level declarations), its name should be the same as the name of the class, with the .kt extension appended.

如果 Kotlin 檔案內包含單個類別 (可能使用有關 top-level 宣告)，檔案的名稱應該與類別名稱相同，檔名附加 .kt 擴展

 If a file contains multiple classes, or only top-level declarations, choose a name describing what the file contains, and name the file accordingly. Use camel humps with an uppercase first letter (e.g. `ProcessDeclarations.kt`).

如果檔案包含多個類別，或只是 top-level 宣告，選擇檔案內包含的名稱描述，並相應地命名該檔案，使用駝峰式與開頭大寫(例如 `ProcessDeclarations.kt`)

The name of the file should describe what the code in the file does. Therefore, you should avoid using meaningless words such as "Util" in file names.

檔案名稱描述檔案內代碼做的事，因此，你應該避免使用無意義的單字在檔案名稱，例如 "Util"

---

### Source file organization (來源檔案組織)

Placing multiple declarations (classes, top-level functions or properties) in the same Kotlin source file is encouraged as long as these declarations are closely related to each other semantically and the file size remains reasonable (not exceeding a few hundred lines).

在相同 Kotlin 來源檔案放置多個宣告 (classes, top-level functions or properties ) 是被鼓勵支持的，只要這些宣告彼此緊密相關的語義並且檔案大小保持合理 (不超過幾百行)

In particular, when defining extension functions for a class which are relevant for all clients of this class, put them in the same file where the class itself is defined. When defining extension functions that make sense only for a specific client, put them next to the code of that client. Do not create files just to hold "all extensions of Foo".

特別地，為所有客戶端相關的類別定義擴展函數，放置它們在相同檔案內並定義類別本身，當只為特別客戶端定義有意義的擴展函數時，放它們在客戶端代碼的旁邊，不要建立檔案只為了保存 "Foo所有擴展"

---

### Class layout (類別怖局、安排)

Generally, the contents of a class is sorted in the following order:

通常，類別的內容按以下的順序做排序

- Property declarations and initializer blocks (屬性宣告和初始化區塊)
- Secondary constructors (第二建構元)
- Method declarations (方法宣告)
- Companion object (夥伴物件：類型於 Java 的靜態類)

Do not sort the method declarations alphabetically or by visibility, and do not separate regular methods from extension methods. Instead, put related stuff together, so that someone reading the class from top to bottom would be able to follow the logic of what's happening. Choose an order (either higher-level stuff first, or vice versa) and stick to it.

不會依字母或可見性排序方法的宣告，也不會分開從擴展方法中的常規方法，相反，放置相關的資料在一起，以便某人從上而下的閱讀類別，能循著正在發生事情的邏輯，選擇一個排序 (首要的高階資料，反之亦然) 和堅持下去

Put nested classes next to the code that uses those classes. If the classes are intended to be used externally and aren't referenced inside the class, put them in the end, after the companion object.

放內嵌類別到使用那些類別代碼的旁邊，如果類別有意的使用在外部類別並不會在類別內引用它，放它們在尾端，夥伴物件之後

---

### Interface implementation layout (介面實作怖局、安排)

When implementing an interface, keep the implementing members in the same order as members of the interface (if necessary, interspersed with additional private methods used for the implementation)

當在實作介面時，保持實作成員(屬性、方法)與介面成員(屬性、方法)相同的順序 (如果有必要，穿插著額外私有方法用於實作)

---

### Overload layout (多載怖局、安排)

**Overload：多載、超載、負載，在相同類別或檔案中，定義「名稱相同」，但「參數個數不同」或是「參數類型不同」的函數方法**

Always put overloads next to each other in a class.

在類別中放置多載彼此相鄰於代碼中的上下順序

## Naming rules (命名規則)

Kotlin follows the Java naming conventions. In particular:

Kotlin 遵循 Java 命名規例，特別地：

Names of packages are always lower case and do not use underscores (`org.example.myproject`). Using multi-word names is generally discouraged, but if you do need to use multiple words, you can either simply concatenate them together or use camel humps (`org.example.myProject`).

Package 命名總是小寫並且沒有使用底線 (`org.example.myproject`)，通常鼓勵使用多個單字命名，如果你需要使用多個單字，你不是用簡單串接就是使用駝峰式 (`org.example.myProject`)

Names of classes and objects start with an upper case letter and use camel humps:

類別與物件名稱開頭為大寫字母並使用駝峰式：

``` kotlin
open class DeclarationProcessor { ... }

object EmptyDeclarationProcessor : DeclarationProcessor() { ... }
```
---

### Function names (函數命名)

Names of functions, properties and local variables start with a lower case letter and use camel humps and no underscores:

函數命名，屬性或區域變數開頭為小寫字母並使用駝峰式、無底線

``` kotlin
fun processDeclarations() { ... }
var declarationCount = ...
```
Exception: factory functions used to create instances of classes can have the same name as the class being created:

例外：用於創建實例的「工廠模式」函數可以與正在創建類別相同的名稱

``` kotlin
abstract class Foo { ... }

class FooImpl : Foo { ... }

fun Foo(): Foo { return FooImpl(...) }
```
---

#### Names for test methods (測式方法的命名)

In tests (and only in tests), it's acceptable to use method names with spaces enclosed in backticks.
(Note that such method names are currently not supported by the Android runtime.) Underscores in method names are also allowed in test code.

在測試中(並只在測試中)，可以接受在反引號 \`\`中使用空格的方法名稱

(注意：這樣命名的方法名稱目前不支援Android運行時期)在測試代碼中允許方法名稱使用底線

``` kotlin
class MyTestCase {
     @Test fun `ensure everything works`() { ... }
     
     @Test fun ensureEverythingWorks_onAndroid() { ... }
}
```
---

### Property names (屬性命名)

Names of constants (properties marked with `const`, or top-level or object `val` properties with no custom `get` function that hold deeply immutable data) should use uppercase underscore-separated names:

常數命名 (屬性標記為 `const` ， 最高層級或物件有 val屬性沒有自定義持有深度不可變資料的 get 函數) 應該大寫字母加底線分隔命名：
``` kotlin
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"
```
Names of top-level or object properties which hold objects with behavior or mutable data should use regular camel-hump names:

最高層級或物件屬性命名持有物件使用行為或可變資料應該使用常規駝峰式命名：
``` kotlin
val mutableCollection: MutableSet<String> = HashSet()
```
Names of properties holding references to singleton objects can use the same naming style as `object` declarations:

屬性命名持有單例物件參照可以使用相同命名風格 `object` 宣告：

``` kotlin
val PersonComparator: Comparator<Person> = ...
```
For enum constants, it's OK to use either uppercase underscore-separated names
(`enum class Color { RED, GREEN }`) or regular camel-humps names starting with an uppercase letter, depending on the usage.

對於 enum 常數，使用大寫底線分隔命名或常規鴕峰式命名開頭大寫字母，取決於用法

---

#### Names for backing properties (背景屬性命名)

If a class has two properties which are conceptually the same but one is part of a public API and another is an implementation detail, use an underscore as the prefix for the name of the private property:

如果類別有兩個屬性在概念上相同，但一個為對外公開API的一部份，並且另一個為實作細節，使用底線為前綴(開頭)為私有屬性的命名

``` kotlin
class C {
    private val _elementList = mutableListOf<Element>()

    val elementList: List<Element>
         get() = _elementList
}
```
---

### Choosing good names (選擇好的命名)

The name of a class is usually a noun or a noun phrase explaining what the class _is_: `List`, `PersonReader`.

類別名稱通常為名詞或名詞短語解釋類別是：`List` , `PersonReader`

The name of a method is usually a verb or a verb phrase saying what the method _does_: `close`, `readPersons`.

方法名稱通常為動詞或動詞短語說明方法做的事：`close` , `readPersons`

The name should also suggest if the method is mutating the object or returning a new one. For instance `sort` is sorting a collection in place, while `sorted` is returning a sorted copy of the collection.

命名也表名方法是目前正在改變的物件或回傳新的物件，比如 `sort` 是排序執行中讓集合到位，而 `sorted` 回傳已排序集合的副本

The names should make it clear what the purpose of the entity is, so it's best to avoid using meaningless words (`Manager`, `Wrapper` etc.) in names.

命名應該清楚說明實體存在的目的是什麼，所以在命名時它最好避免無意義的單字 (`Manager`, `Wrapper` 等等)

When using an acronym as part of a declaration name, capitalize it if it consists of two letters (`IOStream`);
capitalize only the first letter if it is longer (`XmlFormatter`, `HttpInputStream`).

當使用縮寫為宣告命名的一部份時，如果它由兩個字母組成 (`IOStream`) 將縮寫為大寫字母 (`IO`)；如果它是較長的單字 (`XmlFormatter`) 將縮寫的首字為大寫字母 (`Xml`、`Formatter`)


## Formatting (編排格式)

In most cases, Kotlin follows the Java coding conventions.

大部份情況下，Kotlin 遵循 Java 編碼慣例

Use 4 spaces for indentation. Do not use tabs.

使用四個空格為縮排，不使用 tabs

For curly braces, put the opening brace in the end of the line where the construct begins, and the closing brace on a separate line aligned vertically with the opening construct.

對於大括號 `{}`, 在行尾放開頭大括號 `{` 表示建構開始，並在單獨一行關閉大括號 `}` 垂直對齊開頭建構

``` kotlin
if (elements != null) {
    for (element in elements) {
        // ...
    }
}
```
(Note: In Kotlin, semicolons are optional, and therefore line breaks are significant. The language design assumes Java-style braces, and you may encounter surprising behavior if you try to use a different formatting style.)

(注意：在Kotlin，分號 `;` 是可選的(可有可無)，因此換行是重要的，語言設計假設為 Java 風格的括號，如果你嘗試使用不同編排格式，你可能會遇到令人驚訝的問題)

---

### Horizontal whitespace (水平空格)

Put spaces around binary operators (`a + b`). Exception: don't put spaces around the "range to" operator (`0..i`).

在二元運算符的之間 (`a + b`) 放置空格 ，例外：不要在 "range to" 運算符 (`0..i`) 之間放空格

Do not put spaces around unary operators (`a++`)

不要在一個運算符 (`a++`) 之間放空格

Put spaces between control flow keywords (`if`, `when`, `for` and `while`) and the corresponding opening parenthesis.

在控制流程關鍵字 (`if`, `when`, `for` and `while`) 和相應的左括號 `(` 之間放空格，例如：if (....)、while (....)

Do not put a space before an opening parenthesis in a primary constructor declaration, method declaration or method call.

不要在主建構元宣告、方法宣告、方法調用的左括號之前放空格，例如：class A()、fun foo(x: Int )、call()

```kotlin
class A(val x: Int)

fun foo(x: Int) { ... }

fun bar() {
    foo(1)
}
```
Never put a space after `(`, `[`, or before `]`, `)`.

禁止在 `(` , `[` 之後，或是 `]` , `)` 之前放空格

Never put a space around `.` or `?.`: `foo.bar().filter { it > 2 }.joinToString()`, `foo?.bar()`

禁止在 `.` 或 `?.` 之間放空格：`foo.bar().filter { it > 2 }.joinToString()`, `foo?.bar()`

Put a space after `//`: `// This is a comment`

在 `//` 之後放空格：`// This is a comment`

Do not put spaces around angle brackets used to specify type parameters: `class Map<K, V> { ... }`

不要在尖括號 `<` `>` 用於指定類型參數之間放置空格：`class Map<K, V> { ... }`

Do not put spaces around `::`: `Foo::class`, `String::length`

不要在 `::` 之間放空格： `Foo::class`, `String::length`

Do not put a space before `?` used to mark a nullable type: `String?`

不要在 `?` 用於標記可空類型之間放空格： `String?`

As a general rule, avoid horizontal alignment of any kind. Renaming an identifier to a name with a different length should not affect the formatting of either the declaration or any of the usages.

作為一般規則，避免任何類型的水平對齊，識別符重新命名不同長度的名稱不應該影響宣告或任何用法的編排格式

---

### Colon (冒號 `:`)

Put a space before `:` in the following cases:

在 `:` 前放空格，在以下例子：

  * when it's used to separate a type and a supertype; (當它用於分隔一個類型與一個超(父)類型)
  * when delegating to a superclass constructor or a different constructor of the same class; (當調用超(父)類別建構元或一個相同類別不同建構元)
  * after the `object` keyword. (`object` 關鍵字之後)

Don't put a space before `:` when it separates a declaration and its type.

當冒號用於分隔一個宣告並為它的類型時，不要在 `:` 之前放空格

Always put a space after `:`.

一直(總是)在 `:` 之後放空格

``` kotlin
abstract class Foo<out T : Any> : IFoo {
    abstract fun foo(a: Int): T
}

class FooImpl : Foo() {
    constructor(x: String) : this(x) { ... }
    
    val x = object : IFoo { ... } 
} 
```
---

### Class header formatting (類別標頭編排格式)

Classes with a few primary constructor parameters can be written in a single line:

類別的主要建構元參數可以被寫在一行：

```kotlin
class Person(id: Int, name: String)
```
Classes with longer headers should be formatted so that each primary constructor parameter is in a separate line with indentation.Also, the closing parenthesis should be on a new line. If we use inheritance, then the superclass constructor call or list of implemented interfaces should be located on the same line as the parenthesis:

類別有較長的標頭應該被編排，以便每個主建構元參數使用縮排在單獨一行，此外，左括號 `)` 應該在新的一行，如果使用繼承，然後超(父)類別建構元調用或介面實作的列表(清單)應該位於括號同行：

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) { ... }
```
For multiple interfaces, the superclass constructor call should be located first and then each interface should be located in a different line:

對於多個介面，超(父)類別建構元調用應該位於第一個並且接續每個介面應該位於不同行：

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker { ... }
```
For classes with a long supertype list, put a line break after the colon and align all supertype names vertically:

對於類別有較長的超(父)類清單，在冒號後斷行並垂直對齊所有超(父)類型名稱：

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { ... }
}
```
To clearly separate the class header and body when the class header is long, either put a blank line
following the class header (as in the example above), or put the opening curly brace on a separate line:

當類別標頭太長時，清楚的分隔類別標頭與內文，在類別標頭後放置空白行 (如上所示)，或是放左大括號 `{` 在單行：
```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne 
    {
    fun foo() { ... }
}
```


Use regular indent (4 spaces) for constructor parameters.

使用常規縮排 (4空格) 作為建構元參數

> Rationale: This ensures that properties declared in the primary constructor have the same indentation as properties declared in the body of a class.

> 理由：這在確保在主建構元宣告屬性與類別本文宣告屬性有相同縮排

### Modifiers (修飾符)

If a declaration has multiple modifiers, always put them in the following order:

如果宣告多個修飾符，依以下順序(上到下)放置它們：

``` kotlin
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation
companion
inline
infix
operator
data
```
Place all annotations before modifiers:

在修飾符前放置所有註釋：

``` kotlin
@Named("Foo")
private val foo: Foo
```
Unless you're working on a library, omit redundant modifiers (e.g. `public`).

除非你從事於函式庫的應用，省略冗餘的修飾符 (例如 `public`)

### Annotation formatting (註釋編排格式)

Annotations are typically placed on separate lines, before the declaration to which they are attached, and with the same indentation:

典型的註釋放置在單行，在宣告之前附加它們，並使用相同縮排：

``` kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```
Annotations without arguments may be placed on the same line:

註釋沒有參數可以放置在同行：

``` kotlin
@JsonExclude @JvmField
var x: String
```
A single annotation without arguments may be placed on the same line as the corresponding declaration:

單個註釋沒有參數可與對應的宣告同行：
``` kotlin
@Test fun foo() { ... }
```
### File annotations (檔案註釋)

File annotations are placed after the file comment (if any), before the `package` statement, and are separated from `package` with a blank line (to emphasize the fact that they target the file and not the package).

檔案註釋放置在檔案註解 `/**...*/` 之後，在 `package` 敘述之前，使用空白行從 `package` 分隔

``` kotlin
/** License, copyright and whatever */
@file:JvmName("FooBar")

package foo.bar
```
### Function formatting (函數編排格式)

If the function signature doesn't fit on a single line, use the following syntax:

**Function signature：函數簽名，包括函數的名稱、參數順序、參數類型、泛型欄位等資訊總稱**

如果函數簽名不適合單行，使用以下句法：
``` kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType
): ReturnType {
    // body
}
```
Use regular indent (4 spaces) for function parameters.

使用常規縮排 (4空格) 作為函數參數

> Rationale: Consistency with constructor parameters

> 理由：與建構元的一致性

Prefer using an expression body for functions with the body consisting of a single expression.

偏好使用Lambda表達式當內文，函數的內文由單行表達式組合
``` kotlin
fun foo(): Int {     // bad
    return 1 
}

fun foo() = 1        // good
```
### Expression body formatting (表達式內文編排格式)

If the function has an expression body that doesn't fit in the same line as the declaration, put the `=` sign on the first line.Indent the expression body by 4 spaces.

如果函數有表達式內文不適合與宣告同行，放 `=` 符號在第一行，透過4空格縮排表達式內文

``` kotlin
fun f(x: String) =
    x.length
```
### Property formatting (屬性編排格式)

For very simple read-only properties, consider one-line formatting:

對於非常簡單唯讀屬性，考慮單行格式：
```kotlin
val isEmpty: Boolean get() = size == 0
```
For more complex properties, always put `get` and `set` keywords on separate lines:

對於更多複雜屬性，總是放 `get` 和 `put` 關鍵字在單獨一行
```kotlin
val foo: String
    get() { ... }
```
For properties with an initializer, if the initializer is long, add a line break after the equals sign
and indent the initializer by four spaces:

對於使用初始化屬性，如果初始化太長，等於符號 `=` 換行並且透過4空格縮排初始化
```kotlin
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```
### Formatting control flow statements (控制流程敘述編排格式)

If the condition of an `if` or `when` statement is multiline, always use curly braces around the body of the statement.Indent each subsequent line of the condition by 4 spaces relative to statement begin. Put the closing parentheses of the condition together with the opening curly brace on a separate line:

如果 `if` 或 `when` 敘述的條件是多行，使用括號 `()` 環繞敘述內文，透過4個空格相對放置敘述開始條件，縮排後續每行條件，放條件結尾的右括號 `)` 與開頭敘述左括號 `{` 在單獨一行：

``` kotlin
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```
> Rationale: Tidy alignment and clear separation of condition and statement body

> 理由：整潔對齊並清楚分隔條件和敘述內文

Put the `else`, `catch`, `finally` keywords, as well as the `while` keyword of a do/while loop, on the same line as the preceding curly brace:

放置 `else` , `catch` , `finally` 關鍵字，以及 do/while 循環的 `while` 關鍵字，與結尾大括號 `}` 同行

``` kotlin
if (condition) {
    // body
} else {
    // else part
}

try {
    // body
} catch (e: SomeException) {
    // handle
} finally {
    // cleanup
}

do  {
  // body
} while (condition)
```
In a `when` statement, if a branch is more than a single line, consider separating it from adjacent case blocks with a blank line:

在 `when` 敘述，如果分支敘述超過一行，考慮使用空白行從相鄰事件區塊分隔它：

``` kotlin
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ...
        }
    }
}
```
Put short branches on the same line as the condition, without braces.

與條件同行的短分支，沒有括號

``` kotlin
when (foo) {
    true -> bar() // good
    false -> { baz() } // bad
}
```

### Method call formatting (方法調用編排格式)

In long argument lists, put a line break after the opening parenthesis. Indent arguments by 4 spaces. 
Group multiple closely related arguments on the same line.

在太長參數列表，左括號 `(` 後換行，透過4個空格縮排參數，多個密切相關參數群組為一行

``` kotlin
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```
Put spaces around the `=` sign separating the argument name and value.

在 `=` 符號後空格分隔參數名稱與值

### Chained call wrapping (包裝鍊式調用)

When wrapping chained calls, put the `.` character or the `?.` operator on the next line, with a single indent:

當包裝鍊式調用時，放 `.` 字元或 `?.` 運算符在下一行的開頭，使用單行縮排(4空格)

``` kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```
The first call in the chain usually should have a line break before it, but it's OK to omit it if the code makes more sense that way.

在鏈式的第一個調用之前通常應該換行分隔物件，忽略它的代碼更有意義的方式是 ok 的

### Lambda formatting (Lambda編排格式)

In lambda expressions, spaces should be used around the curly braces, as well as around the arrow which separates the parameters from the body. If a call takes a single lambda, it should be passed outside of parentheses whenever possible.

在 Lambda 表達式，應該在大括號 `{}` 之間空格，以及箭頭從內文分隔參數之間空格，如果一個調用帶有單個 Lambda `it`，它盡可能在括號外傳遞

``` kotlin
list.filter { it > 10 }
```
If assigning a label for a lambda, do not put a space between the label and the opening curly brace:

如果為 Lambda 分配一個標籤，在標記 `@` 與左大括號 `{ ` 之間不要放置空格

``` kotlin
fun foo() {
    ints.forEach lit@{
        // ...
    }
}
```
When declaring parameter names in a multiline lambda, put the names on the first line, followed by the arrow and the newline:

當在多行 Lambda宣告參數命稱時，在第一行放置名稱，箭頭之後新的一行
``` kotlin
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ...
}
```
If the parameter list is too long to fit on a line, put the arrow on a separate line:

如果參數列表太長適合新的一行，在單獨一行放箭頭 `->`

``` kotlin
foo {
   context: Context,
   environment: Env
   ->
   context.configureEnv(environment)
}
```
## Documentation comments (文件註解)

For longer documentation comments, place the opening `/**` on a separate line and begin each subsequent line with an asterisk:

對於太長文件註解，在單獨一行放開頭 `/**` 並使用星號 `*` 在後續每行的開頭

``` kotlin
/**
 * This is a documentation comment
 * on multiple lines.
 */
```
Short comments can be placed on a single line:

短註解可以放置單行

``` kotlin
/** This is a short documentation comment. */
```
Generally, avoid using `@param` and `@return` tags. Instead, incorporate the description of parameters and return values directly into the documentation comment, and add links to parameters wherever they are mentioned. Use `@param` and `@return` only when a lengthy description is required which doesn't fit into the flow of the main text.

通常，避免使用 `@param` 和 `@return` 標籤，相反，將參數描述 `＠param ....` 與回傳值 `@return ...` 直接合併到文件註解，並且新增連結到參數無論他們有沒有注意到，只在需要冗長描述且不適合主要文字流程時才使用 `@param` 和 `return` 

``` kotlin
// Avoid doing this:

/**
 * Returns the absolute value of the given number.
 * @param number The number to return the absolute value for.
 * @return The absolute value.
 */
fun abs(number: Int) = ...

// Do this instead:

/**
 * Returns the absolute value of the given [number].
 */
fun abs(number: Int) = ...
```
## Avoiding redundant constructs (避免冗餘的建構元)

In general, if a certain syntactic construction in Kotlin is optional and highlighted by the IDE as redundant, you should omit it in your code. Do not leave unnecessary syntactic elements in code just "for clarity".

一般來說，如果在 Kotlin 某種語意結構由 IDE 視為冗餘是可選(可有可無)和突顯，你應該在你的代碼省略它，不要在代碼中留下不必要的語意元素，就是 '為了清晰' 

### Unit (單元)

**Unit：類似於 Java 的 null**

If a function returns Unit, the return type should be omitted:

如果函數回傳 Unit，回傳類型應該被省略

``` kotlin
fun foo() { // ": Unit" is omitted here

}
```
### Semicolons (分號)

Omit semicolons whenever possible.

盡可能省略

### String templates (字串模版)

Don't use curly braces when inserting a simple variable into a string template. Use curly braces only for longer expressions.

當插入簡單變數到字串模版時不需要大括號，只有對於較長的表達式才使用大括號

``` kotlin
println("$name has ${children.size} children")
```

## Idiomatic use of language features (慣用語言特徵)

### Immutability (不可變、不變性)

Prefer using immutable data to mutable. Always declare local variables and properties as `val` rather than `var` if they are not modified after initialization.

優先使用不可變的資料進行可變的，如果在初始化後不去修改區域變數和屬性，總是宣告為 `val` 而不是 `var` 

Always use immutable collection interfaces (`Collection`, `List`, `Set`, `Map`) to declare collections which are not mutated. When using factory functions to create collection instances, always use functions that return immutable collection types when possible:

總是使用不可變的集合介面(抽象介面) (`Collection`, `List`, `Set`, `Map`) 進行宣告集合不可改變，而不是利用集合的衍生(子)類別為可變的集合類型，當使用工廠函數進行建立集合實例，盡可能回傳不可變集合類型(抽象、介面)而不是具體類型：

``` kotlin
// Bad: use of mutable collection type for value which will not be mutated
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { ... }

// Good: immutable collection type used instead
fun validateValue(actualValue: String, allowedValues: Set<String>) { ... }

// Bad: arrayListOf() returns ArrayList<T>, which is a mutable collection type
val allowedValues = arrayListOf("a", "b", "c")

// Good: listOf() returns List<T>
val allowedValues = listOf("a", "b", "c")
```
簡單說，使用抽象不使用具體

---

### Default parameter values (預設參數值)

**Overload：多載、超載、負載，在相同類別或檔案中，定義「名稱相同」，但「參數個數不同」或是「參數類型不同」的函數方法**

Prefer declaring functions with default parameter values to declaring overloaded functions.

優先宣告函數使用預設值去宣告多載函數
``` kotlin
// Bad
fun foo() = foo("a")
fun foo(a: String) { ... }

// Good
fun foo(a: String = "a") { ... }
```
---

### Type aliases (類型別名)

If you have a functional type or a type with type parameters which is used multiple times in a codebase, prefer defining a type alias for it:

如果你有函數類型或使用類型參數的類型在代碼庫中多次使用，優先定義類型別名給它
```kotlin
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```
---

### Lambda parameters (Lambda 參數)

In lambdas which are short and not nested, it's recommended to use the `it` convention instead of declaring the parameter explicitly. In nested lambdas with parameters, parameters should be always declared explicitly.

在 Lambda 是短而不內嵌的(調用的callback)，建議使用 `it` 慣語而不是明顯的宣告參數，使用參數在內嵌 Lambda，參數應該明顯已被宣告

---

### Returns in a lambda (在 Lambda 回傳)

Avoid using multiple labeled returns in a lambda. Consider restructuring the lambda so that it will have a single exit point.If that's not possible or not clear enough, consider converting the lambda into an anonymous function.

在 Lambda避免使用多個 return 標記，考慮重構 Lambda 以便它有單個出口點，如果這不可能或不夠清楚，考慮轉換 Lambda 到匿名函數

Do not use a labeled return for the last statement in a lambda.

不要在 Lambda 最後一行敘述使用 return，Kotlin 自動會回傳最後一行

---

### Named arguments (命名參數)

Use the named argument syntax when a method takes multiple parameters of the same primitive type, or for parameters of `Boolean` type,unless the meaning of all parameters is absolutely clear from context.

當方法帶有多個相同原生類型的參數或 `Boolean` 類型的參數使用命名參數語法，除非所有參數的意義從環境中絕對清楚的
``` kotlin
drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)
```
---

### Using conditional statements (使用條件敘述)

Prefer using the expression form of `try`, `if` and `when`. Examples:

優先使用 `try` , `if` 和 `when` 表達式形式，例如：
``` kotlin
return if (x) foo() else bar()

return when(x) {
    0 -> "zero"
    else -> "nonzero"
}
```
The above is preferable to:

以上優於：

``` kotlin
if (x)
    return foo()
else
    return bar()
    
when(x) {
    0 -> return "zero"
    else -> return "nonzero"
}    
```
---

### `if` versus `when` ( `if` 與 `when` 相對差別)

Prefer using `if` for binary conditions instead of `when`. Instead of

二元條件優先使用 `if` 而不是 `when`，而不是以下列子：
``` kotlin
when (x) {
    null -> ...
    else -> ...
}
```
use `if (x == null) ... else ...`

Prefer using `when` if there are three or more options.

如果有三個或更多選項，優先使用 `when`

---

### Using nullable `Boolean` values in conditions (在條件上使用可空的 `Boolean` 值)

If you need to use a nullable `Boolean` in a conditional statement, use `if (value == true)` or `if (value == false)` checks.

如果在條件敘述上你需要使用可空的 `Boolean` ，使用`if (value == true)` 或 `if (value == false)` 檢查

---

### Using loops (使用循環)

Prefer using higher-order functions (`filter`, `map` etc.) to loops. Exception: `forEach` (prefer using a regular `for` loop instead, unless the receiver of `forEach` is nullable or `forEach` is used as part of a longer call chain).

優先使用高階函數 (`filter` , `map` 等等) 到循環，例外：`forEach` (優先使用常規 `for` 循環取代，除非 `forEach`的接收器可以為可空的或 `forEach` 用為較長鍊式調用的一部份)

When making a choice between a complex expression using multiple higher-order functions and a loop, understand the cost of the operations being performed in each case and keep performance considerations in mind. 

在使用多個高階函數的複雜表達式和環循之間做選擇，瞭解每個情況下正在執行操作的花費(成本)和留心持續考慮效能

---

### Loops on ranges (循環範圍)

Use the `until` function to loop over an open range:

使用 `until` 函數去遍歷(走訪)開放的範圍

```kotlin
for (i in 0..n - 1) { ... }  // bad
for (i in 0 until n) { ... }  // good
```
---

### Using strings (使用字串)

Prefer using string templates to string concatenation.

優先使用使用字串模版進行字串連接

Prefer to use multiline strings instead of embedding `\n` escape sequences into regular string literals.

優先使用多行字串而不是嵌入 `\n` 跳脫字元序列到常規字串文字中

To maintain indentation in multiline strings, use `trimIndent` when the resulting string does not require any internal indentation, or `trimMargin` when internal indentation is required:

為了在多行字串維護縮排，在結果字串不需要任何內部縮排使用 `trimIndent` ，或當內部縮排被需要 `trimMargin`

``` kotlin
assertEquals(
    """
    Foo
    Bar
    """.trimIndent(), 
    value
)

val a = """if(a > 1) {
          |    return a
          |}""".trimMargin()
```
---

### Functions vs Properties (函數 vs 屬性)

In some cases functions with no arguments might be interchangeable with read-only properties. Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.

在某些情況下，沒有參數的函數可以與唯讀屬性互換，儘管語義相似，有某些風格慣例優先屬性勝過函數

Prefer a property over a function when the underlying algorithm:

當在底層演算法優先使用屬性勝過函數：

* does not throw 不會丟出異常
* is cheap to calculate (or caсhed on the first run) 是便宜容易計算 (或 第一次運行保留快取)
* returns the same result over invocations if the object state hasn't changed (如果在物件的狀態沒有被改變，回傳相同的結果)

---

### Using extension functions (使用擴展函數)

Use extension functions liberally. Every time you have a function that works primarily on an object, consider making it an extension function accepting that object as a receiver. To minimize API pollution, restrict the visibility of extension functions as much as it makes sense. As necessary, use local extension functions, member extension functions, or top-level extension functions with private visibility.

使用豐富的擴展函數，每當你有一個在物件上主要工作的函數，考慮使它變成擴展函數接受物件為接收器，為了最小化污染 API ，限制擴展函數的可見性使它有意義的，根據需要，使用私有可見性的區域擴展函數、擴展函數成員、最高層級擴展函數

---

### Using infix functions (使用中綴函數)

Declare a function as infix only when it works on two objects which play a similar role. Good examples: `and`, `to`, `zip`. Bad example: `add`.

只有函數在兩個物件扮演類似的角色，才宣告函數為中綴，好的例子：`and` , `to` , `zip`。壞的例子： `add`

Don't declare a method as infix if it mutates the receiver object.

如果它是可變的接收器物件，則不要宣告方法為中綴

---

### Factory functions (工廠函數)

If you declare a factory function for a class, avoid giving it the same name as the class itself. Prefer using a distinct name making it clear why the behavior of the factory function is special. Only if there is really no special semantics, you can use the same name as the class.

如果你為類別宣告工廠函數，避免給它與類別本身相同的名子，優先使用獨特名子使它明確知道為何工廠函數的行為是特殊的，只有在沒有特殊語義的情況下，你可以使用與類別相同的名子

Example:

例子：

``` kotlin
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```
If you have an object with multiple overloaded constructors that don't call different superclass constructors and can't be reduced to a single constructor with default argument values, prefer to replace the overloaded constructors with factory functions.

如果你有一個物件有多個多載建構元，不能調用不同超(父)類別建構元並不可減少單個建構元的預設參數值，優先使用工廠函數代替多載建構元

---

### Platform types (平台類型)

A public function/method returning an expression of a platform type must declare its Kotlin type explicitly:

一個公開函數/方法回傳平台類型表達式必須明確宣告它是 Kotlin 類型：
``` kotlin
fun apiCall(): String = MyJavaApi.getProperty("name")
```
Any property (package-level or class-level) initialised with an expression of a platform type must declare its Kotlin type explicitly:

任何屬性 (package-level 或 class-level) 使用平台類型表達式初始化必須明確宣告它是 Kotlin類型：

``` kotlin
class Person {
    val name: String = MyJavaApi.getProperty("name")
}
```
A local value initialised with an expression of a platform type may or may not have a type declaration:

使用平台類型表達式的區域數值初始化可能有也可能沒有有類型宣告：
``` kotlin
fun main(args: Array<String>) {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```
---

### Using scope functions apply/with/run/also/let (使用範圍函數 apply/with/run/also/let)

**詳細來源代碼參考 [Standard.kt](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt)**

Kotlin provides a variety of functions to execute a block of code in the context of a given object. To choose the correct function, consider the following:

Kotlin 提供各種函數 (apply/with/run/also/let) 去執行給予物件內容的代碼區塊，去選擇正確函數，請考慮以下：

  * Are you calling methods on multiple objects in the block, or passing the instance of the context object as an argument? If you are, use one of the functions that allows you to access the context object as `it`,
    not `this` (`also` or `let`). Use `also` if the receiver is not used at all in the block.
  * 你在代碼區塊裡面多個物件調用方法，或傳遞內容物件的實例當作參數？如果是，允許你存取內容物件為 `it` 參數之一，不是 `this` (`also` 或 `let` 兩個函數都帶有參數 `it` 代表調用的物件)，如果在區塊中接收器根本沒有使用，請使用 `also` 

``` kotlin
// Context object is 'it'
class Baz {
    var currentBar: Bar?
    val observable: Observable

    val foo = createBar().also {
        currentBar = it                    // Accessing property of Baz
        observable.registerCallback(it)    // Passing context object as argument
    }
}

// Receiver not used in the block
val foo = createBar().also {
    LOG.info("Bar created")
}

// Context object is 'this'
class Baz {
    val foo: Bar = createBar().apply {
        color = RED    // Accessing only properties of Bar
        text = "Foo"
    }
}
```
  * What should the result of the call be? If the result needs to be the context object, use `apply` or `also`.
    If you need to return a value from the block, use `with`, `let` or `run`
  * 這個調用的回傳結果是什麼？
    如果回傳的結果需要變為內容物件，使用 `apply` 或 `also` (函數外 `{}.xxxx()`可以接續對這個物件的操作)。
    如果你需要從代碼區塊回傳值，使用 `with`、`let`、`run` (可以依代碼區塊的最後一行物件類型當回傳值)。

``` kotlin
// Return value is context object
class Baz {
    val foo: Bar = createBar().apply {
        color = RED    // Accessing only properties of Bar
        text = "Foo"
    }
}


// Return value is block result
class Baz {
    val foo: Bar = createNetworkConnection().let {
        loadBar()
    }
}
```
  * Is the context object nullable, or is it evaluated as a result of a call chain? If it is, use `apply`, `let` or `run`. Otherwise, use `with` or `also`.
  * 內容物件是否為可空的，或是作為調用鍊的結果進行評估？
    如果是，使用 `apply`、`let`、`run`。
    否則，使用 `with`、`also`。

``` kotlin
// Context object is nullable
person.email?.let { sendEmail(it) }

// Context object is non-null and accessible directly
with(person) {
    println("First name: $firstName, last name: $lastName")
}
```
---

### 額外說明：

`receiver` ： 為調用物件，例如：`"123".let{}` 代表 receiver = "123"

`let`

```kotlin
/**
 * Calls the specified function [block] with `this` value as its argument and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

* `{}` 內取得 receiver： `it` 也可自定義
* `{}` 內回傳：最後一行的物件 `-> R`
* `let{}.` 之後回傳：最後一行的物件 `: R`

---

`also`

```kotlin
/**
 * Calls the specified function [block] with `this` value as its argument and returns `this` value.
 */
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

* `{}` 內取得 receiver： `it` 也可自定義
* `{}` 內回傳： Unit `-> Unit`
* `also{}.` 之後類型： 調用物件 (receiver) `: T`

---

`apply`

```kotlin
/**
 * Calls the specified function [block] with `this` value as its receiver and returns `this` value.
 */
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

* `{}` 內取得 receiver： `this` (因為是擴展函數 `T.()`)
* `{}` 內回傳： Unit `-> Unit`
* `apply{}.` 之後類型： 調用物件 (receiver) `: T`

---

`run`

```kotlin
/**
 * Calls the specified function [block] with `this` value as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

* `{}` 內取得 receiver： `this` (因為是擴展函數 `T.()`)
* `{}` 內回傳：最後一行的物件 `-> R`
* `run{}.` 之後類型： 最後一行的物件 `: R`

---

`with`

```kotlin
/**
 * Calls the specified function [block] with the given [receiver] as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

**`with` 多了個參數決定回傳類型**

* `{}` 內取得 receiver： `this` (因為是擴展函數 `T.()`)
* `{}` 內回傳：最後一行的物件 `-> R`
* `run{}.` 之後類型： 最後一行的物件 `: R`

**學習方式：依取得調用物件，`let`(it) > `also`(it) > `apply`(this) > `run`(this) > `with`(this) **

---

## Coding conventions for libraries (函式庫的編碼慣例)

When writing libraries, it's recommended to follow an additional set of rules to ensure API stability:

當在寫函式庫代碼，推薦遵循以下額外規則組來確保 API 穩定性：

 * Always explicitly specify member visibility (to avoid accidentally exposing declarations as public API)
 * 總是明確指定成員的可見性 (為了避免，意外揭露宣告為公開 API)
 * Always explicitly specify function return types and property types (to avoid accidentally changing the return type when the implementation changes)
 * 總是明確指定函數回傳類型和屬性回傳類型 (為了避免當實作改變時，意外改變回傳類型)
 * Provide KDoc comments for all public members, with the exception of overrides that do not require any new documentation (to support generating documentation for the library)
 * 為所有公開成員提供 KDoc 註解，除了不需要任何文件覆寫之外 (支持產生函式庫文件)
