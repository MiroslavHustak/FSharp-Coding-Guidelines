# **F# Coding Guidelines**

Company: Miroslav Husťák (sole owner)

## 1. Philosophy

Following a pure functional programming approach, avoiding object-oriented features and mutibility unless absolutely necessary for interoperability with .NET libraries or specific frameworks.

## 2. General Principles

**Functional-Only Style**

No use of OOP features such as classes, inheritance, or interfaces unless required for interoperability with .NET or external frameworks (such as Fabulous, TorchSharp, or Elmish.WPF).

**No Mixing of Paradigms**

No blending of OOP and functional paradigms. Functional purity is preferred.

## 3. Error Handling and Reflection

**Defensive Coding**

All exceptions and nulls originating from .NET libraries are immediately transformed into `Result` or `Option` types at the boundary of the functional code.

**No Exception-Based Flow**

Errors are propagated using types (predominantly the `Result` type), not exceptions. Raising exceptions is avoided unless there is a compelling reason.

**Handling of Nulls Creeping From .NET Libraries**

`Option.ofNull` rather than `Option.ofObj` or `Option.ofNullable` is used consistently for .NET types — both reference and value types (incl. strings originating from .NET libraries) — for simplicity and uniformity. Strings may be handled with the help of `string`, `Option.ofNullEmpty` or `Option.NullEmptySpace`.  

**Reflection-Free Code**

Reflection is not used in application logic. Libraries that depend on reflection (such as some serialisers) are avoided unless no viable alternative exists.

**Preferred (De)Serialiser:**

Favouring non-reflection-based libraries such as `Thoth.Json.Net` for (de)serialisation.

## 4. Code Structure & Patterns

**Monadic Abstractions**

Monadic custom computation expressions (with single-case discriminated unions used for creating builders) are employed where appropriate. If monadic function composition / ROP is used (functions from `FSharp.Core`, `FsToolkit`, and `FsToolkit.ErrorHandling`), it is recommended to limit its usage to the following functions: `Option.map/bind`, `Result.map/bind`, `Option.defaultValue`, `Result.defaultValue/defaultWith`, `Result.sequence`, and `Result.mapError`.

**Avoid Non-Monadic Bind in Computation Expressions**

When defining custom computation expressions, ensure the `Bind` method adheres to monadic principles. Non-monadic `Bind` implementations can lead to confusing or unpredictable behavior, break compatibility with standard F# idioms, and make code harder to reason about. 

**Asynchronous Code**

Using F#'s `async { ... }` workflows; avoiding C#-style `async/await`. `Async.Parallel` is preferred for concurrency. `Task` may be used for performance-sensitive, CPU-bound operations.

**Functional Control Flow**

Control flow is managed using pattern matching and active patterns instead of `if...then...else` constructs. Looping is typically implemented using either tail-recursive functions or continuation-passing style (CPS) recursive functions, ensuring tail-call optimization or CPS compliance, or through Haskell-like collection functions such as `map`, `iter`, or `fold`. Query expressions are not employed.

**Avoiding Imperative Constructs** 

- No LINQ expressions
- No query expressions
- No mutable state
- No `for` and `while` loops (I do mean it. The `for` loop is not as harmless as you might think.)
- `Array` to be immediately transformed into immutable collections at the boundary of the functional code unless retained for performance-sensitive use cases.

**MVU Organisation**

MVU (Model-View-Update) logic is kept in a single module per UI component. Splitting into multiple files/modules is avoided for the sake of readability.

**Collections**

Immutable collections (`List`, `Seq`, `Set`, `Map`) shall be employed unless there is a compelling reason to use `Array`. Exercise caution when using lazy-evaluated sequences (`Seq`).

**Single-Direction Dependency**

Avoiding the use of the `and` keyword and recursive namespaces to preserve the single-direction dependency concept.

## 5. Enhancing Type Safety

**Type-Driven Development**

Preference is given to type-driven development (using single-case discriminated unions) when reasonable and when memory overhead is acceptable.

**Ignore with Type Parameters**

Using `ignore` with type parameters (for example `ignore<FileInfo>`) catches partial application errors at compile time, preventing subtle bugs in cases where side effects (like logging or printing) are silently skipped.

## 6. Testing

**Testing Philosophy**

Pure functions are assumed to be correct by design especially when type-driven development is applied. Unit tests are optional for these; instead, integration tests (if at all necessary) and property-based testing (strongly recommended) are used.

## 7. Async-by-Default

**Preferring Asynchronous Versions**

If an asynchronous variant of an API exists, it is preferred. Even if Async.RunSynchronously is used initially, adopting the async model opens up future possibilities—such as cancellation or non-blocking constructs—with minimal refactoring. That said, caution should be exercised when using the asynchronous variant within parallel loops.

## 8. Data Handling

**No Fully-Fledged ORMs or Micro ORMs (Object-Relational Mappers)**

Preferring plain SQL for database interactions, avoiding ORMs and micro-ORMs.

**SQL Over SQL Type Providers**

SQL is written explicitly rather than through SQL type providers as it is unsure how SQL type providers handle large databases.

**Type Providers for Non-Database Scenarios**

Type providers for CSV, Excel and JSON are preferred over equivalent .NET libraries due to better type inference and significantly less boilerplate.

**Separation of Data and Operations on Data**

Keeping data separate from operations on data, in accordance with functional programming principles. This separation reinforces the decision to avoid mixing paradigms.

## 9. Learning Lessons from Others 

Learn lessons from others: https://www.youtube.com/watch?v=ua6zxKSiQ_g




