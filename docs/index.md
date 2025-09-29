### Special Escape Sequences

For any character that has a special escape sequence (`\t`, `\n`, `\r`, `\"`, `\'`, `\\`, and `\0`), that sequence is used rather than the equivalent Unicode (e.g., `\u{000a}`) escape sequence.

### Invisible Characters and Modifiers

Invisible characters, such as the zero width space and other control characters that do not affect the graphical representation of a string, are always written as Unicode escape sequences.

Control characters, combining characters, and variation selectors that *do* affect the graphical representation of a string are not escaped when they are attached to a character or characters that they modify. If such a Unicode scalar is present in isolation or is otherwise not modifying another character in the same string, it is written as a Unicode escape sequence.

The strings below are well-formed because the umlauts and variation selectors associate with neighboring characters in the string. The second example is in fact composed of *five* Unicode scalars, but they are unescaped because the specific combination is rendered as a single character.

```swift
let size = "Übergröße"
let shrug = "🤷🏿‍️"
```
In the example below, the umlaut and variation selector are in strings by themselves, so they are escaped.
```swift
let diaeresis = "\u{0308}"
let skinToneType6 = "\u{1F3FF}"
```
If the umlaut were included in the string literally, it would combine with the preceding quotation mark, impairing readability. Likewise, while most systems may render a standalone skin tone modifier as a block graphic, the example below is still forbidden because it is a modifier that is not modifying a character in the same string.
!!! danger "AVOID"
    ```swift
    let diaeresis = "̈"
    let skinToneType6 = "🏿"
    ```

### String Literals

Unicode escape sequences (`\u{????}`) and literal code points (for example, `Ü`) outside the 7-bit ASCII range are never mixed in the same string.

More specifically, string literals are either:
* composed of a combination of Unicode code points written literally and/or single character escape sequences (such as `\t`, but *not* `\u{????}`), or
* composed of 7-bit ASCII with any number of Unicode escape sequences and/or other escape sequences.

The following example is correct because `\n` is allowed to be present among other Unicode code points.
```swift
let size = "Übergröße\n"
```
The following example is allowed because it follows the rules above, but it is **not preferred** because the text is harder to read and understand compared to the string above.
```swift
let size = "\u{00DC}bergr\u{00F6}\u{00DF}e\n"
```
The example below is forbidden because it mixes code points outside the 7-bit ASCII range in both literal form and in escaped form.
!!! danger "AVOID"
    ```swift
    let size = "Übergr\u{00F6}\u{00DF}e\n"
    ```
**Aside:** Never make your code less readable simply out of fear that some programs might not handle non-ASCII characters properly. If that should happen, those programs are broken and must be fixed.

## Source File Structure

### File Comments

Comments describing the contents of a source file are optional. They are discouraged for files that contain only a single abstraction (such as a class declaration)—in those cases, the documentation comment on the abstraction itself is sufficient and a file comment is only present if it provides additional useful information. File comments are allowed for files that contain multiple abstractions in order to document that grouping as a whole.

### Import Statements

A source file imports exactly the top-level modules that it needs; nothing more and nothing less. If a source file uses definitions from both `UIKit` and `Foundation`, it imports both explicitly; it does not rely on the fact that some Apple frameworks transitively import others as an implementation detail.

Imports of whole modules are preferred to imports of individual declarations or submodules. There are a number of reasons to avoid importing individual members:
* There is no automated tooling to resolve/organize imports.
* Existing automated tooling (such as Xcode’s migrator) are less likely to work well on code that imports individual members because they are considered corner cases.
* The prevailing style in Swift (based on official examples and community code) is to import entire modules.

Imports of individual declarations are permitted when importing the whole module would otherwise pollute the global namespace with top-level definitions (such as C interfaces). Use your best judgment in these situations.

Imports of submodules are permitted if the submodule exports functionality that is not available when importing the top-level module. For example, `UIKit.UIGestureRecognizerSubclass` must be imported explicitly to expose the methods that allow client code to subclass `UIGestureRecognizer`—those are not visible by importing `UIKit` alone.

Import statements are not line-wrapped.

Import statements are the first non-comment tokens in a source file. They are grouped in the following fashion, with the imports in each group ordered lexicographically and with exactly one blank line between each group:
1. Module/submodule imports not under test
2. Individual declaration imports (`class`, `enum`, `func`, `struct`, `var`)

```swift
import CoreLocation
import MyThirdPartyModule
import SpriteKit
import UIKit

import func Darwin.C.isatty
```

### Type, Variable, and Function Declarations

To enhance cohesion and readability, a single source file may contain multiple top-level types when they are closely related and form a single conceptual unit. Grouping tightly coupled components together is encouraged, as it simplifies navigation and understanding when working on a specific piece of functionality. For example:

A protocol and the primary class that conforms to it can be defined in the same file.

A public type and its `fileprivate` helper types or error enums, which are used exclusively by the main type, are excellent candidates for being in the same file.

However, avoid placing large, complex, or unrelated types together. If a type is substantial enough to stand on its own, it should reside in its own dedicated file.

The order of types, variables, and functions in a source file, and the order of the members of those types, can have a great effect on readability. However, there is no single correct recipe for how to do it; different files and different types may order their contents in different ways.

What is important is that each file and type uses some logical order, which its maintainer could explain if asked. For example, new methods are not just habitually added to the end of the type, as that would yield “chronological by date added” ordering, which is not a logical ordering.

```swift
class DataManager {
    private var cache: [String: Data] = [:]
    let fileManager = FileManager.default

    init() {
        // ...
    }

    func loadData(for key: String) -> Data? {
        if let cachedData = cache[key] {
            return cachedData
        }
        return loadDataFromFile(named: key)
    }

    private func loadDataFromFile(named fileName: String) -> Data? {
        return nil
    }
}
```

### Overloaded Declarations

When a type has multiple initializers or subscripts, or a file/type has multiple functions with the same base name (though perhaps with different argument labels), *and* when these overloads appear in the same type or extension scope, they appear sequentially with no other code in between.

---

## General Formatting

### Column Limit

Swift code has a column limit of 200 characters. Except as noted below, any line that would exceed this limit must be line-wrapped as described in [Line-Wrapping](#line-wrapping).

**Exceptions:**
* Lines where obeying the column limit is not possible without breaking a meaningful unit of text that should not be broken (for example, a long URL in a comment).
* `import` statements.
* Code generated by another tool.

### Braces

In general, braces follow Kernighan and Ritchie (K&R) style for non-empty blocks with exceptions for Swift-specific constructs and rules:
* There **is no** line break before the opening brace (`{`), *unless* required by application of the rules in [Line-Wrapping](#line-wrapping).
* There **is a** line break after the opening brace (`{`), except in closures, where the signature of the closure is placed on the same line as the curly brace, if it fits, and a line break follows the `in` keyword.
* where it may be omitted as described in [One Statement Per Line](#one-statement-per-line).
* empty blocks may be written as `{}`.
* There **is a** line break before the closing brace (`}`), except where it may be omitted as described in [One Statement Per Line](#one-statement-per-line), or it completes an empty block.
* There **is a** line break after the closing brace (`}`), *if and only if* that brace terminates a statement or the body of a declaration. For example, an `else` block is written `} else {` with both braces on the same line.

### Semicolons

Semicolons (`;`) are **not used**, either to terminate or separate statements.
In other words, the only location where a semicolon may appear is inside a string literal or a comment.

!!! success "GOOD"
    ```swift
    func printSum(_ a: Int, _ b: Int) {
        let sum = a + b
        print(sum)
    }
    ```

!!! danger "AVOID"
    ```swift
    func printSum(_ a: Int, _ b: Int) {
        let sum = a + b;
        print(sum);
    }
    ```

### One Statement Per Line

There is **at most** one statement per line, and each statement is followed by a line break, except when the line ends with a block that also contains zero or one statements.
```swift
guard let value = value else { return 0 }

defer { file.close() }

switch someEnum {
    case .first: 5
    case .second: 10
    case .third: 20
}

let squares = numbers.map { $0 * $0 }

var someProperty: Int {
  get { otherObject.property }
  set { otherObject.property = newValue }
}

var someProperty: Int { otherObject.somethingElse() }

required init?(coder aDecoder: NSCoder) { fatalError("no coder") }
```
Wrapping the body of a single-statement block onto its own line is always allowed. Exercise best judgment when deciding whether to place a conditional statement and its body on the same line. For example, single line conditionals work well for early-return and basic cleanup tasks, but less so when the body contains a function call with significant logic. When in doubt, write it as a multi-line statement.

### Line-Wrapping
#### Function Declarations
```swift
`modifiers func name(formal arguments) { `//content` `}`
`modifiers func name(formal arguments) -> result { `//content` `}`
`modifiers func name<generic arguments>(formal arguments) throws -> result { `//content` `}`
`modifiers func name<generic arguments>(formal arguments) throws -> result` `where` `generic constraints { `//content` `}`
```

Applying the rules above from left to right gives us the following line-wrapping:
```swift
public func index<Elements: Collection, Element>(
    of element: Element,
    in collection: Elements) -> Elements.Index? where Elements.Element == Element, Element: Equatable {
    for current in elements {
        // ...
    }
}
```
Function declarations in protocols that are terminated with a closing parenthesis (`)`) may place the parenthesis on the same line as the final argument *or* on its own line.

```swift
public protocol ContrivedExampleDelegate {
    func contrivedExample(
        _ contrivedExample: ContrivedExample,
        willDoSomethingTo someValue: SomeValue)
}
```

```swift
public protocol ContrivedExampleDelegate {
    func contrivedExample(
        _ contrivedExample: ContrivedExample,
        willDoSomethingTo someValue: SomeValue
    )
}
```
If types are complex and/or deeply nested, individual elements in the arguments/constraints lists and/or the return type may also need to be wrapped. In these rare cases, the same line-wrapping rules apply to those parts as apply to the declaration itself.

```swift
public func performanceTrackingIndex<Elements: Collection, Element>(
    of element: Element,
    in collection: Elements) -> (
    Element.Index?,
    PerformanceTrackingIndexStatistics.Timings,
    PerformanceTrackingIndexStatistics.SpaceUsed) {
        // ...
}
```
However, **typealiases or some other means are often a better way to simplify complex declarations whenever possible.**

#### Type and Extension Declarations

The examples below apply equally to `class`, `struct`, `enum`, `extension`, and `protocol` (with the obvious exception that all but the first do not have superclasses in their inheritance list, but they are otherwise structurally similar).

```swift
class MyClass:
    MySuperclass,
    MyProtocol,
    SomeoneElsesProtocol,
    SomeFrameworkProtocol
{
    // ...
}

class MyContainer<Element>:
    MyContainerSuperclass,
    MyContainerProtocol,
    SomeoneElsesContainerProtocol,
    SomeFrameworkContainerProtocol
{
    // ...
}

class MyContainer<BaseCollection>:
    MyContainerSuperclass,
    MyContainerProtocol,
    SomeoneElsesContainerProtocol,
    SomeFrameworkContainerProtocol
where BaseCollection: Collection {
    // ...
}

class MyContainer<BaseCollection>:
    MyContainerSuperclass,
    MyContainerProtocol,
    SomeoneElsesContainerProtocol,
    SomeFrameworkContainerProtocol
where
    BaseCollection: Collection,
    BaseCollection.Element: Equatable,
    BaseCollection.Element: SomeOtherProtocolOnlyUsedToForceLineWrapping
{
    // ...
}
```
#### Function Calls

When a function call is line-wrapped, each argument is written on its own line, indented +4 from the original line.

As with function declarations, if the function call terminates its enclosing statement and ends with a closing parenthesis (`)`) (that is, it has no trailing closure), then the parenthesis may be placed *either* on the same line as the final argument *or* on its own line.

```swift
let index = index(
    of: veryLongElementVariableName,
    in: aCollectionOfElementsThatAlsoHappensToHaveALongName)
```

```swift
let index = index(
    of: veryLongElementVariableName,
    in: aCollectionOfElementsThatAlsoHappensToHaveALongName
)
```

If the function call ends with a trailing closure and the closure’s signature must be wrapped, then place it on its own line and wrap the argument list in parentheses to distinguish it from the body of the closure below it.

```swift
someAsynchronousAction.execute(withDelay: howManySeconds, context: actionContext) { context in
    doSomething(withContext: context)
}
```

#### Other Expressions
When line-wrapping other expressions that are not function calls (as described above), the second line (the one immediately following the first break) is indented exactly +4 from the original line.

When there are multiple continuation lines, indentation may be varied in increments of +4 as needed. In general, two continuation lines use the same indentation level if and only if they begin with syntactically parallel elements. However, if there are many continuation lines caused by long wrapped expressions, consider splitting them into multiple statements using temporary variables when possible.

!!! success "GOOD"
    ```swift
    let result = anExpression + thatIsMadeUpOf * aLargeNumber +
        ofTerms / andTherefore % mustBeWrapped + (
            andWeWill - keepMakingItLonger * soThatWeHave / aContrivedExample)
    ```

!!! danger "AVOID"
    ```swift
    let result = anExpression + thatIsMadeUpOf * aLargeNumber +
        ofTerms / andTherefore % mustBeWrapped + (
            andWeWill - keepMakingItLonger * soThatWeHave / aContrivedExample)
    ```
### Horizontal Whitespace

**Terminology note:** In this section, *horizontal whitespace* refers to *interior* space. These rules are never interpreted as requiring or forbidding additional space at the start of a line.

Beyond where required by the language or other style rules, and apart from literals and comments, a single Unicode space also appears in the following places **only**:
1. Separating any reserved word starting a conditional or switch statement (such as `if`, `guard`, `while`, or `switch`) from the expression that follows it if that expression starts with an open parenthesis (`(`).

!!! success "GOOD"
    ```swift
    if (x == 0 && y == 0) || z == 0 {
        // ...
    }
    ```

!!! danger "AVOID"
    ```swift
    if(x == 0 && y == 0) || z == 0 {
        // ...
    }
    ```

2. Before any closing curly brace (`}`) that follows code on the same line, before any open curly brace (`{`), and after any open curly brace (`{`) that is followed by code on the same line.

!!! success "GOOD"
    ```swift
    let nonNegativeCubes = numbers.map { $0 * $0 * $0 }.filter { $0 >= 0 }
    ```

!!! danger "AVOID"
    ```swift
    let nonNegativeCubes = numbers.map { $0 * $0 * $0 } .filter { $0 >= 0 }
    let nonNegativeCubes = numbers.map{$0 * $0 * $0}.filter{$0 >= 0}
    ```

3. **On both sides** of any binary or ternary operator, including the “operator-like” symbols described below, with exceptions noted at the end:

* The `=` sign used in assignment, initialization of variables/properties, and default arguments in functions.

!!! success "GOOD"
    ```swift
    var x = 5
    func sum(_ numbers: [Int], initialValue: Int = 0) {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    var x=5
    func sum(_ numbers: [Int], initialValue: Int=0) {
        // ...
    }
    ```
* The ampersand (`&`) in a protocol composition type.

!!! success "GOOD"
    ```swift
    func sayHappyBirthday(to person: NameProviding & AgeProviding) {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    func sayHappyBirthday(to person: NameProviding&AgeProviding) {
        // ...
    }
    ```
* The operator symbol in a function declaring/implementing that operator.

!!! success "GOOD"
    ```swift
    static func == (lhs: MyType, rhs: MyType) -> Bool {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    static func ==(lhs: MyType, rhs: MyType) -> Bool {
        // ...
    }
    ```
* The arrow (`->`) preceding the return type of a function.

!!! success "GOOD"
    ```swift
    func sum(_ numbers: [Int]) -> Int {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    func sum(_ numbers: [Int])->Int {
        // ...
    }
    ```
**Exception:** There is no space on either side of the dot (`.`) used to reference value and type members.

!!! success "GOOD"
    ```swift
    let width = view.bounds.width
    ```
!!! danger "AVOID"
    ```swift
    let width = view . bounds . width
    ```
**Exception:** There is no space on either side of the `..<` or `...` operators used in range expressions.

!!! success "GOOD"
    ```swift
    for number in 1...5 {
        // ...
    }
    let substring = string[index..<string.endIndex]
    ```

!!! danger "AVOID"
    ```swift
    for number in 1 ... 5 {
        // ...
    }
    let substring = string[index ..< string.endIndex]
    ```
4. After, but not before, the comma (`,`) in parameter lists and in tuple/array/dictionary literals.

!!! success "GOOD"
    ```swift
    let numbers = [1, 2, 3]
    ```

!!! danger "AVOID"
    ```swift
    let numbers = [1,2,3]
    let numbers = [1 ,2 ,3]
    let numbers = [1 , 2 , 3]
    ```
5. After, but not before, the colon (`:`) in
* Superclass/protocol conformance lists and generic constraints.

!!! success "GOOD"
    ```swift
    struct HashTable: Collection {
        // ...
    }
    struct AnyEquatable<Wrapped: Equatable>: Equatable {
        // ...
    }
    ```
    
!!! danger "AVOID"
    ```swift
    struct HashTable : Collection {
        // ...
    }
    struct AnyEquatable<Wrapped : Equatable> : Equatable {
        // ...
    }
    ```
* Function argument labels and tuple element labels.

!!! success "GOOD"
    ```swift
    let tuple: (x: Int, y: Int)
    func sum(_ numbers: [Int]) {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    let tuple: (x:Int, y:Int)
    let tuple: (x : Int, y : Int)
    func sum(_ numbers:[Int]) {
        // ...
    }
    func sum(_ numbers : [Int]) {
        // ...
    }
    ```
* Variable/property declarations with explicit types.

!!! success "GOOD"
    ```swift
    let number: Int = 5
    ```
!!! danger "AVOID"
    ```swift
    let number:Int = 5
    let number : Int = 5
    ```
* Shorthand dictionary type names.
!!! success "GOOD"
    ```swift
    var nameAgeMap: [String: Int] = []
    ```
!!! danger "AVOID"
    ```swift
    var nameAgeMap: [String:Int] = []
    var nameAgeMap: [String : Int] = []
    ```
* Dictionary literals.

!!! success "GOOD"
    ```swift
    let nameAgeMap = ["Ed": 40, "Timmy": 9]
    ```
!!! danger "AVOID"
    ```swift
    let nameAgeMap = ["Ed":40, "Timmy":9]
    let nameAgeMap = ["Ed" : 40, "Timmy" : 9]
    ```
6. At least two spaces before and exactly one space after the double slash (`//`) that begins an end-of-line comment.

!!! success "GOOD"
    ```swift
    let initialFactor = 2  // Warm up the modulator.
    ```
!!! danger "AVOID"
    ```swift
    let initialFactor = 2 //    Warm up the modulator.
    ```
7. Outside, but not inside, the brackets of an array or dictionary literals and the parentheses of a tuple literal.

!!! success "GOOD"
    ```swift
    let numbers = [1, 2, 3]
    ```
!!! danger "AVOID"
    ```swift
    let numbers = [ 1, 2, 3 ]
    ```
### Horizontal Alignment

**Terminology note:** *Horizontal alignment* is the practice of adding a variable number of additional spaces in your code with the goal of making certain tokens appear directly below certain other tokens on previous lines.

Horizontal alignment is forbidden except when writing obviously tabular data where omitting the alignment would be harmful to readability. In other cases (for example, lining up the types of stored property declarations in a `struct` or `class`), horizontal alignment is an invitation for maintenance problems if a new member is introduced that requires every other member to be realigned.

!!! success "GOOD"
    ```swift
    struct DataPoint {
        var value: Int
        var primaryColor: UIColor
    }
    ```

!!! danger "AVOID"
    ```swift
    struct DataPoint {
        var value:        Int
        var primaryColor: UIColor
    }
    ```
### Vertical Whitespace

A single blank line appears in the following locations:
1. Between consecutive members of a type: properties, initializers, methods, enum cases, and nested types, **except that**:
* A blank line is optional between two consecutive stored properties or two enum cases whose declarations fit entirely on a single line. Such blank lines can be used to create **logical groupings** of these declarations.
* A blank line is optional between two extremely closely related properties that do not otherwise meet the criterion above; for example, a private stored property and a related public computed property.
2. **Only as needed** between statements to organize code into logical subsections.
3. **Optionally** before the first member or after the last member of a type (neither is encouraged nor discouraged).
4. Anywhere explicitly required by other sections of this document.

**Multiple** blank lines are permitted, but never required (nor encouraged). If you do use multiple consecutive blank lines, do so consistently throughout your code base.

### Parentheses

Parentheses are **not** used around the top-most expression that follows an `if`, `guard`, `while`, or `switch` keyword.

!!! success "GOOD"
    ```swift
    if x == 0 {
        print("x is zero")
    }
    if (x == 0 || y == 1) && z == 2 {
        print("...")
    }
    ```
!!! danger "AVOID"
    ```swift
    if (x == 0) {
        print("x is zero")
    }
    if ((x == 0 || y == 1) && z == 2) {
        print("...")
    }
    ```
Optional grouping parentheses are omitted only when the author and the reviewer agree that there is no reasonable chance that the code will be misinterpreted without them, nor that they would have made the code easier to read. It is **not** reasonable to assume that every reader has the entire Swift operator precedence table memorized.

---

## Formatting Specific Constructs
### Non-Documentation Comments

Non-documentation comments always use the double-slash format (`//`), never the C-style block format (`/* ... */`).

### Properties

Local variables are declared close to the point at which they are first used (within reason) to minimize their scope.

With the exception of tuple destructuring, every `let` or `var` statement (whether a property or a local variable) declares exactly one variable.

!!! success "GOOD"
    ```swift
    var a = 5
    var b = 10
    let (quotient, remainder) = divide(100, 9)
    ```

!!! danger "AVOID"
    ```swift
    var a = 5, b = 10
    ```
### Switch Statements

- In multi-line cases, case statements are aligned with switch (no extra indentation).
- In single-line cases, case statements are indented one level (+4 spaces) from switch.
- The body of a case in multi-line format is indented +4 spaces from the case.
- Use single-line case only if the body is short (≈ 40 characters or less).
- The 40-character limit is approximate — developers may decide to switch to multi-line for better readability even with fewer characters.

!!! success "GOOD"
    ```swift
    switch order {
        case .ascending: print("Ascending")
        case .descending: print("Descending")
        case .same: print("Same")
    }
    
    switch order {
    case .ascending:
        print("This is a much longer statement that should be split")
        log("Another statement inside the same case")
    case .descending:
        doSomething()
        doSomethingElse()
    case .same:
        print("Same")
    }
    ```
!!! danger "AVOID"
    ```swift
    switch order {
        case .ascending:
            print("Wrong extra indentation")
        case .descending:
            print("Also wrong")
    }
    
    switch order {
    case .ascending:print("Ascending")
    case .descending:print("Descending")
    case .same:print("Same")
    }
    ```

### Enum Naming
The naming convention for enums depends on their role. The primary rule is to use singular names for enums that define a type, but plural names are acceptable when an enum is used as a namespace.
**The Main Rule: Singular for Types**
When an enum is used to define a type with distinct states or values, its name must be singular.

Think of it this way: the type is Direction, and a variable of that type can hold one of its values, such as .north. We don't write let myDirection: Directions.

!!! success "GOOD"
    ```swift
    // An enum as a type representing one state out of several.
    enum NetworkState {
        case connected
        case disconnected
        case connecting
    }
    
    // An enum as a type representing a specific choice.
    enum PaymentMethod {
        case creditCard(String)
        case cash
        case crypto(address: String)
    }
    ```
!!! danger "AVOID"
    ```swift
    // Avoid plural names for type names.
    enum NetworkStates { ... }
    
    enum PaymentMethods { ... }
    ```
The Exception: Plural for Namespaces

When a caseless enum is used as a namespace to group static constants, a plural name is appropriate and often preferred because it describes the group or collection of items stored within.

!!! success "GOOD"
    ```swift
    // "Strings" is a logical group for string constants.
    // "Images" is a collection of keys for images.
    enum AppConstants {
        enum Strings {
            static let welcomeTitle = "Welcome!"
            static let errorTitle = "Error"
        }
    
        enum Images {
            static let mainIcon = "app_icon"
            static let placeholder = "user_placeholder"
        }
    }
    ```
    
### Enum Cases

In general, there is only one `case` per line in an `enum`. The comma-delimited form may be used only when none of the cases have associated values or raw values, all cases fit on a single line, and the cases do not need further documentation because their meanings are obvious from their names.

!!! success "GOOD"
    ```swift
    public enum Token {
        case comma
        case semicolon
        case identifier
    }
    
    public enum Token {
        case comma, semicolon, identifier
    }
    ```
!!! danger "AVOID"
    ```swift
    public enum Token {
        case comma
        case semicolon
        case identifier(String)
    }
    
    public enum Token {
        case comma, semicolon, identifier(String)
    }
    ```

When an `enum` case does not have associated values, empty parentheses are never present.

!!! success "GOOD"
    ```swift
    public enum BinaryTree<Element> {
        indirect case node(element: Element, left: BinaryTree, right: BinaryTree)
        case empty
    }
    ```
!!! danger "AVOID"
    ```swift
    public enum BinaryTree<Element> {
        indirect case node(element: Element, left: BinaryTree, right: BinaryTree)
        case empty()
    }
    ```
The cases of an enum must follow a logical ordering that the author could explain if asked. If there is no obviously logical ordering, use a lexicographical ordering based on the cases’ names.

In the following example, the cases are arranged in numerical order based on the underlying HTTP status code.
```swift
public enum HTTPStatus: Int {
    case ok = 200
    case badRequest = 400
    case notAuthorized = 401
    case paymentRequired = 402
    case forbidden = 403
    case notFound = 404
    case internalServerError = 500
}
```
The following version of the same enum is less readable. Although the cases are ordered lexicographically, the meaningful groupings of related values has been lost.
```swift
public enum HTTPStatus: Int {
    case badRequest = 400
    case forbidden = 403
    case internalServerError = 500
    case notAuthorized = 401
    case notFound = 404
    case ok = 200
    case paymentRequired = 402
}
```
### Trailing Closures

Functions should not be overloaded such that two overloads differ **only** by the name of their trailing closure argument. Doing so prevents using trailing closure syntax—when the label is not present, a call to the function with a trailing closure is ambiguous.

Consider the following example, which prohibits using trailing closure syntax to call `greet`:
```swift
func greet(enthusiastically nameProvider: () -> String) {
    print("Hello, \(nameProvider())! It's a pleasure to see you!")
}
func greet(apathetically nameProvider: () -> String) {
    print("Oh, look. It's \(nameProvider()).")
}

greet { "John" }  // error: ambiguous use of 'greet'
```
This example is fixed by differentiating some part of the function name other than the closure argument—in this case, the base name:
```swift
func greetEnthusiastically(_ nameProvider: () -> String) {
    print("Hello, \(nameProvider())! It's a pleasure to see you!")
}
func greetApathetically(_ nameProvider: () -> String) {
    print("Oh, look. It's \(nameProvider()).")
}

greetEnthusiastically { "John" }
greetApathetically { "not John" }
```
If a function call has multiple closure arguments, then **none** are called using trailing closure syntax; **all** are labeled and nested inside the argument list’s parentheses.

!!! success "GOOD"
    ```swift
    UIView.animate(
        withDuration: 0.5,
        animations: {
            // ...
        },
        completion: { finished in
            // ...
        })
    ```
!!! danger "AVOID"
    ```swift
    UIView.animate(
        withDuration: 0.5,
        animations: {
            // ...
        }) { finished in
            // ...
        }
    ```

If a function call has multiple closure arguments, use multiple trailing closure syntax. The first trailing closure is written without its argument label; subsequent trailing closures must include their argument labels.

This modern syntax (available since Swift 5.3) is the preferred, idiomatic style as it improves readability and makes the call site resemble a native Swift control flow statement.

!!! success "GOOD"
    ```swift
    // Use modern multiple trailing closure syntax.
    // The code reads like a native control flow statement.
    UIView.animate(withDuration: 0.5) {
        // ...
    } completion: {  finished in
        // ...
    }
    ```

!!! danger "AVOID"
    ```swift
    // Avoid nesting multiple closures inside the parentheses.
    // This style is verbose and considered outdated.
    UIView.animate(
        withDuration: 0.5,
        animations: {
            // ...
        },
        completion: { finished in
            // ...
        }
    )
    ```

If a function has a single closure argument and it is the final argument, then it is **always** called using trailing closure syntax, except in the following cases to resolve ambiguity or parsing errors:
* As described above, labeled closure arguments must be used to disambiguate between two overloads with otherwise identical arguments lists.
* Labeled closure arguments must be used in control flow statements where the body of the trailing closure would be parsed as the body of the control flow statement.

!!! success "GOOD"
    ```swift
    Timer.scheduledTimer(timeInterval: 30, repeats: false) { timer in
        print("Timer done!")
    }
    if let firstActive = list.first(where: { $0.isActive }) {
        process(firstActive)
    }
    ```
!!! danger "AVOID"
    ```swift
    Timer.scheduledTimer(timeInterval: 30, repeats: false, block: { timer in
        print("Timer done!")
    })
    // This example fails to compile.
    if let firstActive = list.first { $0.isActive } {
        process(firstActive)
    }
    ```
When a function called with trailing closure syntax takes no other arguments, empty parentheses (`()`) after the function name are **never** present.
!!! success "GOOD"
    ```swift
    let squares = [1, 2, 3].map { $0 * $0 }
    ```

!!! danger "AVOID"
    ```swift
    let squares = [1, 2, 3].map({ $0 * $0 })
    let squares = [1, 2, 3].map() { $0 * $0 }
    ```

### Trailing Commas

Trailing commas in array and dictionary literals are forbidden. The last element in a multi-line literal must not have a comma following it.

!!! success "GOOD"
    ```swift
    let configurationKeys = [
        "bufferSize",
        "compression",
        "encoding"
    ]
    ```
!!! danger "AVOID"
    ```swift
    let configurationKeys = [
        "bufferSize",
        "compression",
        "encoding",
    ]
    ```
### Numeric Literals

It is recommended but not required that long numeric literals (decimal, hexadecimal, octal, and binary) use the underscore (`_`) separator to group digits for readability when the literal has numeric value or when there exists a domain-specific grouping.

Recommended groupings are three digits for decimal (thousands separators), four digits for hexadecimal, four or eight digits for binary literals, or value-specific field boundaries when they exist (such as three digits for octal file permissions).

Do not group digits if the literal is an opaque identifier that does not have a meaningful numeric value.

### Attributes
Parameterized attributes (such as `@availability(...)` or `@objc(...)`) are each written on their own line immediately before the declaration to which they apply, are lexicographically ordered, and are indented at the same level as the declaration.

!!! success "GOOD"
    ```swift
    @available(iOS 9.0, *)
    public func coolNewFeature() {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    @available(iOS 9.0, *) public func coolNewFeature() {
        // ...
    }
    ```

## Naming

### Naming Conventions Are Not Access Control

Restricted access control (`internal`, `fileprivate`, or `private`) is preferred for the purposes of hiding information from clients, rather than naming conventions.

Naming conventions (such as prefixing a leading underscore) are only used in rare situations when a declaration must be given higher visibility than is otherwise desired in order to work around language limitations—for example, a type that has a method that is only intended to be called by other parts of a library implementation that crosses module boundaries and must therefore be declared `public`.

### Identifiers

In general, identifiers contain only 7-bit ASCII characters. Unicode identifiers are allowed if they have a clear and legitimate meaning in the problem domain of the code base (for example, Greek letters that represent mathematical concepts) and are well understood by the team who owns the code.

!!! success "GOOD"
    ```swift
    let smile = "😊"
    let deltaX = newX - previousX
    let Δx = newX - previousX
    ```
!!! danger "AVOID"
    ```swift
    let 😊 = "😊"
    ```
### Initializers
For clarity, initializer arguments that correspond directly to a stored property have the same name as the property. Explicit `self.` is used during assignment to disambiguate them.

!!! success "GOOD"
    ```swift
    public struct Person {
        public let name: String
        public let phoneNumber: String

        public init(name: String, phoneNumber: String) {
            self.name = name
            self.phoneNumber = phoneNumber
        }
    }
    ```
!!! danger "AVOID"
    ```swift
    public struct Person {
        public let name: String
        public let phoneNumber: String

        public init(name otherName: String, phoneNumber otherPhoneNumber: String) {
            name = otherName
            phoneNumber = otherPhoneNumber
        }
    }
    ```
### Static and Class Properties

Static and class properties that return instances of the declaring type are **not** suffixed with the name of the type.

!!! success "GOOD"
    ```swift
    public class UIColor {
        public class var red: UIColor {
            // ...
        }
    }
    
    public class URLSession {
        public class var shared: URLSession {
            // ...
        }
    }
    ```
!!! danger "AVOID"
    ```swift
    public class UIColor {
        public class var redColor: UIColor {
            // ...
        }
    }
    
    public class URLSession {
        public class var sharedSession: URLSession {
            // ...
        }
    }
    ```
When a static or class property evaluates to a singleton instance of the declaring type, the names `shared` and `default` are commonly used. This style guide does not require specific names for these; the author should choose a name that makes sense for the type.

### Global Constants

Like other variables, global constants are `lowerCamelCase`. Hungarian notation, such as a leading `g` or `k`, is not used.

!!! success "GOOD"
    ```swift
    let secondsPerMinute = 60
    ```
!!! danger "AVOID"
    ```swift
    let SecondsPerMinute = 60
    let kSecondsPerMinute = 60
    let gSecondsPerMinute = 60
    let SECONDS_PER_MINUTE = 60
    ```

### Compiler Warnings
Code should compile without warnings when feasible. Any warnings that are able to be removed easily by the author must be removed.

A reasonable exception is deprecation warnings, where it may not be possible to immediately migrate to the replacement API, or where an API may be deprecated for external users but must still be supported inside a library during a deprecation period.

### Properties
The `get` block for a read-only computed property is omitted and its body is directly nested inside the property declaration.

!!! success "GOOD"
    ```swift
    var totalCost: Int {
        return items.sum { $0.cost }
    }
    ```
!!! danger "AVOID"
    ```swift
    var totalCost: Int {
        get {
            return items.sum { $0.cost }
        }
    }
    ```

### Types with Shorthand Names

Arrays, dictionaries, and optional types are written in their shorthand form whenever possible; that is, `[Element]`, `[Key: Value]`, and `Wrapped?`. The long forms `Array<Element>`, `Dictionary<Key, Value>`, and `Optional<Wrapped>` are only written when required by the compiler; for example, the Swift parser requires `Array<Element>.Index` and does not accept `[Element].Index`.

!!! success "GOOD"
    ```swift
    func enumeratedDictionary<Element>(
        from values: [Element],
        start: Array<Element>.Index? = nil) -> [Int: Element] {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    func enumeratedDictionary<Element>(
        from values: Array<Element>,
        start: Optional<Array<Element>.Index> = nil) -> Dictionary<Int, Element> {
        // ...
    }
    ```
`Void` is a `typealias` for the empty tuple `()`, so from an implementation point of view they are equivalent. In function type declarations (such as closures, or variables holding a function reference), the return type is always written as `Void`, never as `()`. In functions declared with the `func` keyword, the `Void` return type is omitted entirely.

Empty argument lists are always written as `()`, never as `Void`. (In fact, the function signature `Void -> Result` is an error in Swift because function arguments must be surrounded by parentheses, and `(Void)` has a different meaning: an argument list with a single empty-tuple argument.)

!!! success "GOOD"
    ```swift
    func doSomething() {
        // ...
    }
    let callback: () -> Void
    ```
!!! danger "AVOID"
    ```swift
    func doSomething() -> Void {
        // ...
    }
    
    func doSomething2() -> () {
        // ...
    }
    let callback: () -> ()
    ```
### Optional Types

Sentinel values are avoided when designing algorithms (for example, an “index” of −1 when an element was not found in a collection). Sentinel values can easily and accidentally propagate through other layers of logic because the type system cannot distinguish between them and valid outcomes.

`Optional` is used to convey a non-error result that is either a value or the absence of a value. For example, when searching a collection for a value, not finding the value is still a **valid and expected** outcome, not an error.

!!! success "GOOD"
    ```swift
    func index(of thing: Thing, in things: [Thing]) -> Int? {
        // ...
    }
    
    if let index = index(of: thing, in: lotsOfThings) {
        // Found it.
    } else {
        // Didn't find it.
    }
    ```
!!! danger "AVOID"
    ```swift
    func index(of thing: Thing, in things: [Thing]) -> Int {
        // ...
    }
    
    let index = index(of: thing, in: lotsOfThings)
    if index != -1 {
        // Found it.
    } else {
        // Didn't find it.
    }
    ```

Conditional statements that test that an `Optional` is non-nil *but do not access the wrapped value* are written as comparisons to `nil`. The following example is clear about the programmer’s intent:

!!! success "GOOD"
    ```swift
    if value != nil {
        print("value was not nil")
    }
    ```
This example, while taking advantage of Swift’s pattern matching and binding syntax, obfuscates the intent by appearing to unwrap the value and then immediately throw it away.
!!! danger "AVOID"
    ```swift
    if let _ = value {
        print("value was not nil")
    }
    ```

### Access Levels

Omitting an explicit access level is permitted on declarations. For top-level declarations, the default access level is `internal`. For nested declarations, the default access level is the lesser of `internal` and the access level of the enclosing declaration.

Specifying an explicit access level at the file level on an extension is forbidden. Each member of the extension has its access level specified if it is different than the default.

!!! success "GOOD"
    ```swift
    extension String {
        public var isUppercase: Bool {
            // ...
        }

        public var isLowercase: Bool {
            // ...
        }
    }
    ```
!!! danger "AVOID"
    ```swift
    public extension String {
        var isUppercase: Bool {
            // ...
        }

        var isLowercase: Bool {
            // ...
        }
    }
    ```

### Nesting and Namespacing

Swift allows `enums`, `structs`, and `classes to be nested, so nesting is preferred (instead of naming conventions) to express scoped and hierarchical relationships among types when possible. For example, flag `enums or error types that are associated with a specific type are nested in that type.

!!! success "GOOD"
    ```swift
    class Parser {
        enum Error: Swift.Error {
            case invalidToken(String)
            case unexpectedEOF
        }

        func parse(text: String) throws {
            // ...
        }
    }
    ```
!!! danger "AVOID"
    ```swift
    class Parser {
        func parse(text: String) throws {
            // ...
        }
    }
    enum ParseError: Error {
        case invalidToken(String)
        case unexpectedEOF
    }
    ```
Swift does not currently allow protocols to be nested in other types or vice versa, so this rule does not apply to situations such as the relationship between a controller class and its delegate protocol.

Declaring an `enum` without cases is the canonical way to define a “namespace” to group a set of related declarations, such as constants or helper functions. This `enum` automatically has no instances and does not require that extra boilerplate code be written to prevent instantiation.

!!! success "GOOD"
    ```swift
    enum Dimensions {
        static let tileMargin: CGFloat = 8
        static let tilePadding: CGFloat = 4
        static let tileContentSize: CGSize(width: 80, height: 64)
    }
    ```
!!! danger "AVOID"
    ```swift
    struct Dimensions {
        private init() {}

        static let tileMargin: CGFloat = 8
        static let tilePadding: CGFloat = 4
        static let tileContentSize: CGSize(width: 80, height: 64)
    }
    ```

### `guards` for Early Exits

A `guard` statement, compared to an `if` statement with an inverted condition, provides visual emphasis that the condition being tested is a special case that causes early exit from the enclosing scope.

Furthermore, `guard` statements improve readability by eliminating extra levels of nesting (the “pyramid of doom”); failure conditions are closely coupled to the conditions that trigger them and the main logic remains flush left within its scope.

This can be seen in the following examples; in the first, there is a clear progression that checks for invalid states and exits, then executes the main logic in the successful case. In the second example without `guard`, the main logic is buried at an arbitrary nesting level and the thrown errors are separated from their conditions by a great distance.

!!! success "GOOD"
    ```swift
    func discombobulate(_ values: [Int]) throws -> Int {
        guard let first = values.first else {
            throw DiscombobulationError.arrayWasEmpty
        }
        guard first >= 0 else {
            throw DiscombobulationError.negativeEnergy
        }

        var result = 0
        for value in values {
            result += invertedCombobulatoryFactory(of: value)
        }
        return result
    }
    ```
!!! danger "AVOID"
    ```swift
    func discombobulate(_ values: [Int]) throws -> Int {func discombobulate(_ values: [Int]) throws -> Int {
        if let first = values.first {
            if first >= 0 {
            var result = 0
            
            for value in values {
                result += invertedCombobulatoryFactor(of: value)
            }
            return result
            } else {
                throw DiscombobulationError.negativeEnergy
            }
        } else {
            throw DiscombobulationError.arrayWasEmpty
        }
    }
    ```
A guard-continue statement can also be useful in a loop to avoid increased indentation when the entire body of the loop should only be executed in some cases (but see also the for-where discussion below.)

### for-where Loops
When the entirety of a for loop’s body would be a single if block testing a condition of the element, the test is placed in the where clause of the for statement instead.

!!! success "GOOD"
    ```swift
    for item in collection where item.hasProperty {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    for item in collection {
        if item.hasProperty {
            // ...
        }
    }
    ```
### fallthrough in switch Statements
When multiple cases of a switch would execute the same statements, the case patterns are combined into ranges or comma-delimited lists. Multiple case statements that do nothing but fallthrough to a case below are not allowed.

!!! success "GOOD"
    ```swift
    switch value {
    case 1:
        print("one")
    case 2...4:
        print("two to four")
    case 5, 7:
        print("five or seven")
    default:
        break
    }
    ```
!!! danger "AVOID"
    ```swift
    switch value {
    case 1:
        print("one")
    case 2:
        fallthrough
     case 3:
        fallthrough
    case 4:
        print("two to four")
    case 5:
        fallthrough
    case 7:
        print("five or seven")
    default:
        break
    }
    ```
In other words, there is never a case whose body contains only the fallthrough statement. Cases containing additional statements which then fallthrough to the next case are permitted.

### Pattern Matching
The placement of let and var keywords in a pattern depends on the number of values being bound within that pattern.
- For patterns with a single binding, place the keyword directly before the variable (case .name(let value)).
- For patterns with two or more bindings, use the shorthand version where the keyword precedes the entire pattern (case let .name(value1, value2)).

This approach maintains clarity for simple, single-value cases while reducing verbosity and repetition in more complex cases with multiple values.

!!! success "GOOD"
    ```swift
    switch serverResponse {
    case .success(let data):
        print("Received (data.count) bytes.")
    case let .failure(code, message):
        print("Error \(code): \(message)")
    case .notModified:
        break
    }
    ```
!!! danger "AVOID"
    ```swift
    switch serverResponse {
    case let .success(data):
        print("Received (data.count) bytes.")
    case .failure(let code, let message):
        print("Error \(code): \(message)")
    case .notModified:
        break
    }
    ```
Labels of tuple arguments and enum associated values are omitted when binding a value to a variable with the same name as the label.

!!! success "GOOD"
    ```swift
    switch treeNode { 
        case .subtree(let left, let right): // ... 
        case .leaf(let element): // ... 
    }
    ```

!!! danger "AVOID"
    ```swift
    // Including the labels adds redundant noise.
    switch treeNode {
        case .subtree(left: let left, right: let right): // ...
        case .leaf(element: let element): // ...
    }
    ```

### Tuple Patterns
Assigning variables through a tuple pattern is only permitted if the left-hand side of the assignment is unlabeled.

!!! success "GOOD"
    ```swift
    let (a, b) = (y: 4, x: 5.0)
    ```
!!! danger "AVOID"
    ```swift
    // Labels on the left-hand side resemble type annotations and can be confusing.
    let (x: a, y: b) = (y: 4, x: 5.0)

    // This declares two variables: `Int` (which is a `Double`) and `Double` (which is an `Int`). `x` and `y` are not variables.
    let (x: Int, y: Double) = (y: 4, x: 5.0)
    ```
    
### Numeric and String Literals
When a literal is used to initialize a value of a type other than its default, specify the type explicitly.

!!! success "GOOD"
    ```swift
    // These are explicitly typed.
    let x2: Int32 = 50
    let x3 = 50 as Int32

    let y2: Character = "a"
    let y3 = "a" as Character

    // GOOD. The compiler emits errors for invalid coercions, which is safe.
    // error: integer literal '...' overflows when stored into 'Int64'
    let a = 0x8000_0000_0000_0000 as Int64
    // error: cannot convert value of type 'String' to type 'Character'
    let b = "ab" as Character
    ```

!!! danger "AVOID"
    ```swift
    // Using initializer syntax can lead to misleading compiler errors or runtime errors.

    // This fails to compile because the literal doesn't fit into a signed `Int` first.
    let a1 = UInt64(0x8000_0000_0000_0000)

    // This is significantly slower than coercion.
    let b = Character("a")

    // This traps at runtime.
    let c = Character("ab")
    ```

### Trapping vs. Overflowing Arithmetic
The standard (trapping-on-overflow) arithmetic operators (+, -, *) are used for most normal operations.

!!! success "GOOD"
    ```swift
    //Overflow will not cause the balance to go negative.
    let newBankBalance = oldBankBalance + recentHugeProfit
    ```

!!! danger "AVOID"
    ```swift
    //Overflow will cause the balance to wrap around, resulting in bad data.
    let newBankBalance = oldBankBalance &+ recentHugeProfit
    ```
Masking operations are permitted in domains that use modular arithmetic, such as cryptography and hash functions.

!!! success "GOOD"
    ```swift
    //For hash values, the distribution of the bit pattern matters.
    var hashValue: Int {
        return foo.hashValue &+ 31 * (bar.hashValue &+ 31 &* baz.hashValue)
    }
    ```

!!! danger "AVOID"
    ```swift
    //This will trap unpredictably depending on the hash values.
    var hashValue: Int {
        return foo.hashValue + 31 * (bar.hashValue + 31 * baz.hashValue)
    }
    ```
### Documentation Comments
General Format

Documentation comments are written using the triple slash (///). Javadoc-style block comments (/** ... */) are not permitted.

!!! success "GOOD"
    ```swift
    /// Returns the numeric value of the given digit.
    ///
    /// - Parameters:
    ///   - digit: The Unicode scalar whose numeric value should be returned.
    ///   - radix: The radix used to compute the numeric value.
    /// - Returns: The numeric value of the scalar.
    func numericValue(of digit: UnicodeScalar, radix: Int = 10) -> Int {
        // ...
    }
    ```

!!! danger "AVOID"
    ```swift
    /**
     * Returns the numeric value of the given digit.
     */
    func numericValue(of digit: UnicodeScalar, radix: Int = 10) -> Int {
      // ...
    }
    ```
### Single-Sentence Summary

Documentation comments begin with a brief single-sentence summary. Method summaries are verb phrases; property summaries are noun phrases.

!!! success "GOOD"
    ```swift
    /// The background color of the view.
    var backgroundColor: UIColor

    /// Returns the sum of the numbers in the given array.
    func sum(_ numbers: [Int]) -> Int {
        // ...
    }
    ```

!!! danger "AVOID"
    ```swift
    /// This property is the background color of the view.
    var backgroundColor: UIColor

    /// This method returns the sum of the numbers in the given array.
    func sum(_ numbers: [Int]) -> Int {
      // ...
    }
    ```
### Parameter, Returns, and Throws Tags

Use the singular - Parameter tag for a single argument. Use the plural - Parameters tag with a nested list for multiple arguments.

!!! success "GOOD"
    ```swift
    /// - Parameter command: The command to execute.
    func execute(command: String) -> String { ... }

    /// - Parameters:
    ///   - command: The command to execute.
    ///   - stdin: The string to use as standard input.
    func execute(command: String, stdin: String) -> String { ... }
    ```

!!! danger "AVOID"
    ```swift
    // Using the plural form for a single parameter.
    /// - Parameters:
    ///   - command: The command to execute.
    func execute(command: String) -> String { ... }

    // Using the singular form multiple times.
    /// - Parameter command: The command to execute.
    /// - Parameter stdin: The string to use as standard input.
    func execute(command: String, stdin: String) -> String { ... }
    ```
    Of course. Here is a new rule formatted in the style of the provided document, ready to be added to the Programming Practices section.

### Higher-Order Functions for Collections

Prefer using higher-order functions like map, filter, reduce, and flatMap for common collection operations over constructing manual for-in loops. This practice leads to more declarative, concise, and readable code by clearly stating the transformation's intent rather than the step-by-step mechanics of its implementation. It also avoids the boilerplate of creating and mutating intermediate collections.

Furthermore, for transformations that simply access a property of an element, you can use the even more concise KeyPath expression (\.propertyName). This further improves readability by directly referencing the property being mapped.

!!! success "GOOD"
    ```swift 
    // The intent—transforming and filtering—is immediately clear.
    let squaredEvens = numbers.map { $0 * $0 }.filter { $0 % 2 == 0 }
    
    // Using a KeyPath is the most direct way to extract properties.
    let userNames = users.map(\.name)
    ```

!!! danger "AVOID"
    ```swift  
    // This imperative approach requires more code and mental overhead to understand. 
    var squaredEvens: [Int] = []
    
    for number in numbers {
        let squared = number * number if squared % 2 == 0 { squaredEvens.append(squared) } 
    } 
    ```
    
### Transforming Optionals with map and flatMap

Prefer map and flatMap to perform transformations on optional values rather than using verbose if let or guard let statements. This approach produces more concise, declarative code by chaining the operation directly to the optional value itself.

Use map when your transformation function returns a non-optional value. Use flatMap when your transformation function returns an Optional value; this prevents creating a nested optional (e.g., Int??).

!!! success "GOOD"
    ```swift
    // 'map' is used because the transform returns a non-optional String.
    let optionalNumber: Int? = 42
    let optionalDescription = optionalNumber.map { "The number is ($0)" }
    
    // 'flatMap' is used because `Int(String)` returns an `Int?`.
    let optionalString: String? = "123"
    let optionalInt = optionalString.flatMap { Int($0) }
    ```
!!! danger "AVOID"
    ```swift
    // Unnecessarily verbose for a simple transformation.
    let optionalNumber: Int? = 42
    let optionalDescription: String?
    
    if let number = optionalNumber {
        optionalDescription = "The number is (number)"
    } else {
        optionalDescription = nil
    }
    
    // This guard statement adds boilerplate that `flatMap` handles automatically.
    guard let string = optionalString else {
        return nil
    }
    let optionalInt = Int(string)
    ```
    
### Acronyms and Initialisms

Names containing common acronyms and initialisms (such as URL, ID, API, or HTTP) should treat the acronym as a single word, keeping it fully capitalized. This applies to both UpperCamelCase type names and lowerCamelCase variable names. Avoid using mixed-case variants.

This practice improves clarity by treating well-known acronyms as indivisible units, which is how developers mentally read them.

!!! success "GOOD"
    ```swift
    var requestURL: URL
    let userID: Int
    let htmlContent: String
    
    struct APIClient {
        func downloadHTML() { ... }
    }
    ```
    
!!! danger "AVOID"
    ```swift
    // Avoid mixed-case or partially-lowercased acronyms.
    var requestUrl: URL
    let userId: Int
    let anHtmlDocument: String
    
    struct ApiClient {
        func downloadHtml() { ... }
    }
    ```

### Use Type Aliases for Clarity

Actively use type aliases (typealias) to give simpler, more descriptive names to complex types. This is especially useful for tuples, closures, and generic types with multiple constraints. Using a type alias makes code more readable, self-documenting, and easier to maintain.

A well-named type alias can express the purpose of a complex type, making function signatures and property declarations much clearer.

!!! success "GOOD"
    ```swift
    // The type alias clearly defines the structure and purpose.
    typealias HTTPResponse = (data: Data?, response: URLResponse?, error: Error?)
    
    func handle(response: HTTPResponse) {
        response.data.map { data in
            // ...
        }
    }
    
    typealias UserFetchCompletion = (Result<[User], APIError>) -> Void
    
    func fetchUsers(completion: @escaping UserFetchCompletion) {
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    // The complex, inline tuple makes the function signature hard to read.
    func handle(response: (data: Data?, response: URLResponse?, error: Error?)) {
        // ...
    }
    
    // The closure's purpose is not immediately clear from the signature.
    func fetchUsers(completion: @escaping (Result<[User], APIError>) -> Void) {
        // ...
    }
    ```
### Implicit Returns

You should omit the return keyword in functions, computed properties, and closures that are composed of a single expression. The return keyword should only be used when the compiler requires it, for instance, in bodies that contain multiple statements or other declarations.

This practice leverages a modern Swift feature that reduces boilerplate and improves conciseness, making the code's intent clearer. 📜

!!! success "GOOD"
    ```swift
    // For a single-line computed property
    var isEnabled: Bool {
        !items.isEmpty
    }
    
    // For a single-expression function
    func square(of number: Int) -> Int {
        number * number
    }
    
    // For a switch statement that is a single expression in a function
    func color(for state: ConnectionState) -> UIColor {
        switch state {
            case .connected: .green
            case .disconnected: .red
            case .connecting: .yellow
        }
    }
    ```
!!! danger "AVOID"
    ```swift
    // The return keyword is redundant here.
    var isEnabled: Bool {
        return !items.isEmpty
    }
    
    // The `return` keyword is unnecessary in a single-expression function.
    func square(of number: Int) -> Int {
        return number * number
    }
    
    // Each case is a single expression, so `return` is not needed.
    func color(for state: ConnectionState) -> UIColor {
        switch state {
            case .connected: return .green
            case .disconnected: return .red
            case .connecting: return .yellow
        }
    }
    ```
### Ternary Operators

When a ternary expression (? :) needs to be line-wrapped, a line break is inserted before both the ? and the :. The continuation lines containing these operators are indented exactly +4 from the original line.

This style ensures that all three parts of the expression—the condition, the true case, and the false case—are clearly separated and aligned, which greatly improves readability.

!!! success "GOOD"
    ```swift
    let accessLevel = user.isAdministrator
        ? "Administrator"
        : "Standard User"
    
    let color = isEnabled
        ? .primary
        : .secondary.withAlphaComponent(0.5)
    ```
!!! danger "AVOID"
    ```swift
    // Placing operators at the end of the line makes it harder to scan.
    let accessLevel = user.isAdministrator ?
    "Administrator" :
    "Standard User"
    
    // This inconsistent indentation is confusing.
    let color = isEnabled ? .primary
        : .secondary.withAlphaComponent(0.5)
    ```
    
### Abbreviations in Names
Type names (classes, structs, protocols, enums) must be fully spelled out. Avoid abbreviations to make the API as clear and self-documenting as possible. A type's name should fully describe its purpose without requiring a reader to guess what an abbreviation stands for.

However, property and local variable names may use common, contextually-clear abbreviations (such as btn, img, mgr, config). This avoids excessive verbosity within the implementation where the context is already established.

!!! success "GOOD"
    ```swift
    // The class name is fully spelled out for clarity.
    struct UserProfile {
        // Within the implementation, abbreviations for button color and manager are acceptable.
        private var btnColor: Color
        private let configManager: ConfigurationManager
    
        // ...
    }
    ```
!!! danger "AVOID"
    ```swift
    // The class name is unclear due to abbreviations. What is "VC"? "Usr"? "Prof"?
    struct UsrProfVC {
        // The property name is unnecessarily long and verbose.
        private var buttonBackgroundColor: Color
        private let configurationManagerInstance: ConfigurationManager
    
        // ...
    }
    ```

### Numeric Suffixes in Names
Avoid appending arbitrary numbers to the names of types such as classes, structs, protocols, and enums. A type's name should describe its purpose or domain, not its sequence or version. If you need variants of a type, prefer more descriptive names or use generics.

However, using numeric suffixes for properties or local variables is permissible when it is part of a well-defined, non-arbitrary sequence (e.g., coordinates, versioned assets, or mathematical components).

!!! success "GOOD"
    ```swift
    struct LineSegment { 
        var point1: CGPoint 
        var point2: CGPoint 
    }

    let themeColor1: Color
    let themeColor2: Color
    ```

!!! danger "AVOID"
    ```swift
    // AVOID: Type names should not be numbered. This creates confusion
    // about the relationship between UserV1 and UserV2.
    // Prefer a single User type with a version property, or use protocols.
    class UserV1 { ... }
    class UserV2 { ... }
    ```
