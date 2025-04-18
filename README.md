# **F-Coding-Guidelines**

Company: Miroslav Husťák (sole owner)

## 1. Philosophy

Following a pure functional programming approach, avoiding object-oriented features unless absolutely necessary for interoperability with .NET libraries or specific frameworks.

## 2. General Principles

<span style="color:red;">**Functional-Only Style** </span>

No use of classes, inheritance, or interfaces unless required for interoperability with .NET or external frameworks (such as Fabulous or Elmish.WPF).

**No Mixing of Paradigms**

No blending of OOP and functional paradigms. Functional purity is preferred.

## 3. Error Handling and Reflection

**Defensive Coding**

All exceptions and nulls originating from .NET libraries are immediately transformed into `Result` or `Option` types at the boundary of the functional code.

**No Exception-Based Flow**

Errors are propagated using types (predominantly the `Result` type), not exceptions. Raising exceptions is avoided unless there is a compelling reason.

**Null Handling**

`Option.ofNull` rather than `Option.ofObj` or `Option.ofNullable` is used consistently for .NET types — both reference and value types (incl. String originating from a .NET library) — for simplicity and uniformity. String values may be handled with the help of string, Option.ofNullEmpty or Option.NullEmptySpace.  

**Reflection-Free Code**

Reflection is not used in application logic. Libraries that depend on reflection (such as some serialisers) are avoided unless no viable alternative exists.
Preferred Serialiser: Thoth.Json.Net
Favouring non-reflection-based libraries such as `Thoth.Json.Net` for (de)serialisation.

## 4. Code Structure & Patterns

**Monadic Abstractions**

Monadic custom computation expressions (using single-case discriminated unions for builders) are employed where appropriate. Monadic function composition (selected `FSharp.Core`, `FsToolkit`, and `FsToolkit.ErrorHandling` functions) is applied judiciously: `Option/Result.map/bind`, `Result.defaultValue/defaultWith/mapError`.

**Asynchronous Code**

Using F#'s `async { ... }` workflows; avoiding C#-style `async/await`. `Async.Parallel` is preferred for concurrency. `Task` may be used for performance-sensitive, CPU-bound operations.

**Functional Control Flow**

Using pattern matching and active patterns instead of `if...then...else`. Looping is achieved through tail-recursive functions or Haskell-like collection functions such as `map`, `iter`, or `fold`. No query expressions.

**Avoiding Imperative Constructs** 

•	No LINQ expressions<br/> 
•	No mutable state<br/>
•	No `for`/`while` loops<br/> 
•	Arrays immediately transformed into immutable collections at the boundary of the functional code unless retained for performance-sensitive use cases.<br/>

**MVU Organisation**

MVU (Model-View-Update) logic is kept in a single module per UI component. Splitting into multiple files/modules is avoided for the sake of readability.

**Single-Direction Dependency**

Avoiding the use of the `and` keyword and recursive namespaces to preserve the single-direction dependency concept.

## 5. Types and Testing

**Type-Driven Design**

Preference is given to type-driven development (using single-case discriminated unions) when reasonable and when memory overhead is acceptable.

**Testing Philosophy**

Pure functions are assumed to be correct by construction. Unit tests are optional for these; instead, integration tests and property-based testing are used.

## 6. Async-by-Default

**Preferring Async Versions**

If an async variant of an API exists, preference is given to it. Even if `Async.RunSynchronously` is initially used, the async model opens up future possibilities (such as cancellation or non-blocking constructs) with minimal refactoring.

## 7. Data Handling

**No ORMs and Micro ORMs (Object-Relational Mappers)**

Plain SQL is preferred for database access, using ADO.NET for MS SQL Server and ODP.NET for Oracle. Templates for common scenarios are hand-written (both async and sync).

**SQL Over SQL Type Providers**

SQL is written explicitly rather than through SQL type providers as it is unsure how SQL type providers handle large databases.

**Type Providers for Non-Database Scenarios**

Type providers for Excel and JSON are preferred over equivalent .NET libraries due to better type inference and significantly less boilerplate.

**Separation of Data and Operations on Data**

Keeping data separate from operations on data, in accordance with functional programming principles. This separation reinforces the decision to avoid mixing paradigms.

