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

(例如：如果在專案中所有的代碼在 "org.example.kotlin" package 和它的子 packages，檔案內宣告 package  "org.example.kotlin" 應該直接放置在起始(根)來源下，並在 "org.example.kotlin.foo.bar"下的檔案應該在起始來源的子目錄 "foo/bar")

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

在相同 Kotlin 來源檔案放置多個宣告 (classes , top-level functions or properties ) 是被鼓勵支持的，只要這些宣告彼此緊密相關的語義並且檔案大小保持合理 (不超過幾百行)

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

When implementing an interface, keep the implementing members in the same order as members of the interface (if necessary,interspersed with additional private methods used for the implementation)

當在實作介面時，保持實作成員(屬性、方法)與介面成員(屬性、方法)相同的順序 (如果有必要，穿插著額外私有方法用於實作)

---

### Overload layout (多載怖局、安排)

**Overload：多載、超載、負載，在相同類別中，定義「名稱相同」，但「參數個數不同」或是「參數類型不同」的函數方法**

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

在控制關鍵字 (`if`, `when`, `for` and `while`) 和相應的左括號 `(` 之間放空格，例如：if (....)、while (....)

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

禁止在 `(` , `[` 之後，或是 `]` ,  `)` 之前放空格

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

### Class header formatting

Classes with a few primary constructor parameters can be written in a single line:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class Person(id: Int, name: String)
```
</div>

Classes with longer headers should be formatted so that each primary constructor parameter is in a separate line with indentation.
Also, the closing parenthesis should be on a new line. If we use inheritance, then the superclass constructor call or list of implemented interfaces
should be located on the same line as the parenthesis:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) { ... }
```
</div>

For multiple interfaces, the superclass constructor call should be located first and then each interface should be located in a different line:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker { ... }
```
</div>

For classes with a long supertype list, put a line break after the colon and align all supertype names vertically:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { ... }
}
```
</div>

To clearly separate the class header and body when the class header is long, either put a blank line
following the class header (as in the example above), or put the opening curly brace on a separate line:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { ... }
}
```
</div>

Use regular indent (4 spaces) for constructor parameters.

> Rationale: This ensures that properties declared in the primary constructor have the same indentation as properties
> declared in the body of a class.

### Modifiers

If a declaration has multiple modifiers, always put them in the following order:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
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
</div>

Place all annotations before modifiers:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@Named("Foo")
private val foo: Foo
```
</div>

Unless you're working on a library, omit redundant modifiers (e.g. `public`).

### Annotation formatting

Annotations are typically placed on separate lines, before the declaration to which they are attached, and with the same indentation:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```
</div>

Annotations without arguments may be placed on the same line:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@JsonExclude @JvmField
var x: String
```
</div>

A single annotation without arguments may be placed on the same line as the corresponding declaration:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@Test fun foo() { ... }
```
</div>

### File annotations

File annotations are placed after the file comment (if any), before the `package` statement, and are separated from `package` with a blank line (to emphasize the fact that they target the file and not the package).

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
/** License, copyright and whatever */
@file:JvmName("FooBar")

package foo.bar
```
</div>

### Function formatting

If the function signature doesn't fit on a single line, use the following syntax:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType
): ReturnType {
    // body
}
```
</div>

Use regular indent (4 spaces) for function parameters.

> Rationale: Consistency with constructor parameters

Prefer using an expression body for functions with the body consisting of a single expression.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun foo(): Int {     // bad
    return 1 
}

fun foo() = 1        // good
```
</div>

### Expression body formatting

If the function has an expression body that doesn't fit in the same line as the declaration, put the `=` sign on the first line.
Indent the expression body by 4 spaces.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
fun f(x: String) =
    x.length
```
</div>

### Property formatting

For very simple read-only properties, consider one-line formatting:

<div class="sample" markdown="1" theme="idea" data-highlight-only >
```kotlin
val isEmpty: Boolean get() = size == 0
```
</div>

For more complex properties, always put `get` and `set` keywords on separate lines:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
```kotlin
val foo: String
    get() { ... }
```
</div>

For properties with an initializer, if the initializer is long, add a line break after the equals sign
and indent the initializer by four spaces:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
```kotlin
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```
</div>

### Formatting control flow statements

If the condition of an `if` or `when` statement is multiline, always use curly braces around the body of the statement.
Indent each subsequent line of the condition by 4 spaces relative to statement begin. 
Put the closing parentheses of the condition together with the opening curly brace on a separate line:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```
</div>

> Rationale: Tidy alignment and clear separation of condition and statement body

Put the `else`, `catch`, `finally` keywords, as well as the `while` keyword of a do/while loop, on the same line as the 
preceding curly brace:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
if (condition) {
    // body
} else {
    // else part
}

try {
    // body
} finally {
    // cleanup
}
```
</div>

In a `when` statement, if a branch is more than a single line, consider separating it from adjacent case blocks with a blank line:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
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
</div>

Put short branches on the same line as the condition, without braces.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
when (foo) {
    true -> bar() // good
    false -> { baz() } // bad
}
```
</div>


### Method call formatting

In long argument lists, put a line break after the opening parenthesis. Indent arguments by 4 spaces. 
Group multiple closely related arguments on the same line.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```
</div>

Put spaces around the `=` sign separating the argument name and value.

### Chained call wrapping

When wrapping chained calls, put the `.` character or the `?.` operator on the next line, with a single indent:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```
</div>

The first call in the chain usually should have a line break before it, but it's OK to omit it if the code makes more sense that way.

### Lambda formatting

In lambda expressions, spaces should be used around the curly braces, as well as around the arrow which separates the parameters
from the body. If a call takes a single lambda, it should be passed outside of parentheses whenever possible.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
list.filter { it > 10 }
```
</div>

If assigning a label for a lambda, do not put a space between the label and the opening curly brace:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun foo() {
    ints.forEach lit@{
        // ...
    }
}
```
</div>

When declaring parameter names in a multiline lambda, put the names on the first line, followed by the arrow and the newline:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ...
}
```
</div>

If the parameter list is too long to fit on a line, put the arrow on a separate line:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
foo {
   context: Context,
   environment: Env
   ->
   context.configureEnv(environment)
}
```
</div>

## Documentation comments

For longer documentation comments, place the opening `/**` on a separate line and begin each subsequent line
with an asterisk:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
/**
 * This is a documentation comment
 * on multiple lines.
 */
```
</div>

Short comments can be placed on a single line:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
/** This is a short documentation comment. */
```
</div>

Generally, avoid using `@param` and `@return` tags. Instead, incorporate the description of parameters and return values
directly into the documentation comment, and add links to parameters wherever they are mentioned. Use `@param` and
`@return` only when a lengthy description is required which doesn't fit into the flow of the main text.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
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
</div>

## Avoiding redundant constructs

In general, if a certain syntactic construction in Kotlin is optional and highlighted by the IDE
as redundant, you should omit it in your code. Do not leave unnecessary syntactic elements in code
just "for clarity".

### Unit

If a function returns Unit, the return type should be omitted:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun foo() { // ": Unit" is omitted here

}
```
</div>

### Semicolons

Omit semicolons whenever possible.

### String templates

Don't use curly braces when inserting a simple variable into a string template. Use curly braces only for longer expressions.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
println("$name has ${children.size} children")
```
</div>


## Idiomatic use of language features

### Immutability

Prefer using immutable data to mutable. Always declare local variables and properties as `val` rather than `var` if
they are not modified after initialization.

Always use immutable collection interfaces (`Collection`, `List`, `Set`, `Map`) to declare collections which are not
mutated. When using factory functions to create collection instances, always use functions that return immutable
collection types when possible:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
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
</div>

### Default parameter values

Prefer declaring functions with default parameter values to declaring overloaded functions.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
// Bad
fun foo() = foo("a")
fun foo(a: String) { ... }

// Good
fun foo(a: String = "a") { ... }
```
</div>

### Type aliases

If you have a functional type or a type with type parameters which is used multiple times in a codebase, prefer defining
a type alias for it:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```
</div>

### Lambda parameters

In lambdas which are short and not nested, it's recommended to use the `it` convention instead of declaring the parameter
explicitly. In nested lambdas with parameters, parameters should be always declared explicitly.


### Returns in a lambda

Avoid using multiple labeled returns in a lambda. Consider restructuring the lambda so that it will have a single exit point.
If that's not possible or not clear enough, consider converting the lambda into an anonymous function.

Do not use a labeled return for the last statement in a lambda.

### Named arguments

Use the named argument syntax when a method takes multiple parameters of the same primitive type, or for parameters of `Boolean` type,
unless the meaning of all parameters is absolutely clear from context.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)
```
</div>

### Using conditional statements

Prefer using the expression form of `try`, `if` and `when`. Examples:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
return if (x) foo() else bar()

return when(x) {
    0 -> "zero"
    else -> "nonzero"
}
```
</div>

The above is preferable to:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
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
</div>

### `if` versus `when`

Prefer using `if` for binary conditions instead of `when`. Instead of

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
when (x) {
    null -> ...
    else -> ...
}
```
</div>

use `if (x == null) ... else ...`

Prefer using `when` if there are three or more options.

### Using nullable `Boolean` values in conditions

If you need to use a nullable `Boolean` in a conditional statement, use `if (value == true)` or `if (value == false)` checks.

### Using loops

Prefer using higher-order functions (`filter`, `map` etc.) to loops. Exception: `forEach` (prefer using a regular `for` loop instead,
unless the receiver of `forEach` is nullable or `forEach` is used as part of a longer call chain).

When making a choice between a complex expression using multiple higher-order functions and a loop, understand the cost
of the operations being performed in each case and keep performance considerations in mind. 

### Loops on ranges

Use the `until` function to loop over an open range:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
for (i in 0..n - 1) { ... }  // bad
for (i in 0 until n) { ... }  // good
```
</div>

### Using strings

Prefer using string templates to string concatenation.

Prefer to use multiline strings instead of embedding `\n` escape sequences into regular string literals.

To maintain indentation in multiline strings, use `trimIndent` when the resulting string does not require any internal
indentation, or `trimMargin` when internal indentation is required:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
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
</div>

### Functions vs Properties

In some cases functions with no arguments might be interchangeable with read-only properties. 
Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.

Prefer a property over a function when the underlying algorithm:

* does not throw
* is cheap to calculate (or caсhed on the first run)
* returns the same result over invocations if the object state hasn't changed

### Using extension functions

Use extension functions liberally. Every time you have a function that works primarily on an object, consider making it
an extension function accepting that object as a receiver. To minimize API pollution, restrict the visibility of
extension functions as much as it makes sense. As necessary, use local extension functions, member extension functions,
or top-level extension functions with private visibility.

### Using infix functions

Declare a function as infix only when it works on two objects which play a similar role. Good examples: `and`, `to`, `zip`.
Bad example: `add`.

Don't declare a method as infix if it mutates the receiver object.

### Factory functions

If you declare a factory function for a class, avoid giving it the same name as the class itself. Prefer using a distinct name
making it clear why the behavior of the factory function is special. Only if there is really no special semantics,
you can use the same name as the class.

Example:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```
</div>

If you have an object with multiple overloaded constructors that don't call different superclass constructors and
can't be reduced to a single constructor with default argument values, prefer to replace the overloaded constructors with
factory functions.

### Platform types

A public function/method returning an expression of a platform type must declare its Kotlin type explicitly:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun apiCall(): String = MyJavaApi.getProperty("name")
```
</div>

Any property (package-level or class-level) initialised with an expression of a platform type must declare its Kotlin type explicitly:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class Person {
    val name: String = MyJavaApi.getProperty("name")
}
```
</div>

A local value initialised with an expression of a platform type may or may not have a type declaration:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun main(args: Array<String>) {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```
</div>

### Using scope functions apply/with/run/also/let

Kotlin provides a variety of functions to execute a block of code in the context of a given object. To choose the correct
function, consider the following:

  * Are you calling methods on multiple objects in the block, or passing the instance of the context object as an 
    argument? If you are, use one of the functions that allows you to access the context object as `it`,
    not `this` (`also` or `let`). Use `also` if the receiver is not used at all in the block.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
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
</div>
​    
  * What should the result of the call be? If the result needs to be the context object, use `apply` or `also`.
    If you need to return a value from the block, use `with`, `let` or `run`

<div class="sample" markdown="1" theme="idea" data-highlight-only>
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
</div>
  * Is the context object nullable, or is it evaluated as a result of a call chain? If it is, use `apply`, `let` or `run`.
    Otherwise, use `with` or `also`.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
// Context object is nullable
person.email?.let { sendEmail(it) }

// Context object is non-null and accessible directly
with(person) {
    println("First name: $firstName, last name: $lastName")
}
```
</div>


## Coding conventions for libraries

When writing libraries, it's recommended to follow an additional set of rules to ensure API stability:

 * Always explicitly specify member visibility (to avoid accidentally exposing declarations as public API)
 * Always explicitly specify function return types and property types (to avoid accidentally changing the return type
   when the implementation changes)
 * Provide KDoc comments for all public members, with the exception of overrides that do not require any new documentation
   (to support generating documentation for the library)
