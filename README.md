# Wing Language Specification

- [0. Preface](#0-preface)
- [1. General](#1-general)
  - [1.1 Types](#11-types)
    - [1.1.1 Primitive Types](#111-primitive-types)
    - [1.1.2 Container Types](#112-container-types)
    - [1.1.3 Function Types](#113-function-types)
  - [1.2 Debugging Utilities](#12-debugging-utilities)
  - [1.3 Phase Modifiers](#13-phase-modifiers)
  - [1.4 Storage Modifiers](#14-storage-modifiers)
  - [1.5 Access Modifiers](#15-access-modifiers)
  - [1.6 Mutability](#16-mutability)
  - [1.7 Optionality](#17-optionality)
  - [1.8 Type Inference](#18-type-inference)
  - [1.9 Error Handling](#19-error-handling)
  - [1.10 Formatting](#110-formatting)
  - [1.11 Memory Management](#111-memory-management)
  - [1.12 Documentation Style](#112-documentation-style)
  - [1.13 Execution Model](#113-execution-model)
- [2. Expressions](#2-expressions)
  - [2.1 bring expression](#21-bring-expression)
  - [2.2 break expression](#22-break-expression)
  - [2.3 continue expression](#23-continue-expression)
  - [2.4 return expression](#24-return-expression)
  - [2.5 await Expression](#25-await-expression)
- [3. Statements](#3-statements)
  - [3.1 if statement](#31-if-statement)
  - [3.2 for statement](#32-for-statement)
  - [3.3 while statement](#33-while-statement)
- [4. Declarations](#4-declarations)
  - [4.1 Structs](#41-structs)
  - [4.2 Classes](#42-classes)
  - [4.3 Resources](#43-resources)
  - [4.4 Interfaces](#44-interfaces)
  - [4.5 Variables](#45-variables)
  - [4.6 Functions](#46-functions)
    - [4.6.1 Closures](#461-closures)
    - [4.6.2 Promises](#462-promises)
    - [4.6.3 Struct Expansion](#463-struct-expansion)
    - [4.6.4 Variadic Arguments](#464-variadic-arguments)
  - [4.7 Lists](#47-lists)
  - [4.8 Enumeration](#48-enumeration)
  - [4.9 Computed Properties](#49-computed-properties)
- [5. Module System](#5-module-system)
  - [5.1 Imports](#51-imports)
    - [5.1.1 Verbose Notation](#511-verbose-notation)
    - [5.1.2 Shorthand Notation](#512-shorthand-notation)
  - [5.2 Exports](#52-exports)
- [6. JSII Interoperability](#6-jsii-interoperability)
- [7. Miscellaneous](#7-miscellaneous)
  - [7.1 Strings](#71-strings)
    - [7.1.1 Normal strings "..."](#711-normal-strings-)
    - [7.1.2 Shell strings \`...\`](#712-shell-strings-)
  - [7.2 Comments](#72-comments)
  - [7.3 Operators](#73-operators)
    - [7.3.1 Relational Operators](#731-relational-operators)
    - [7.3.2 Logical Operators](#732-logical-operators)
    - [7.3.3 Bitwise Operators](#733-bitwise-operators)
    - [7.3.4 Mathematics Operators](#734-mathematics-operators)
    - [7.3.5 Operator Precedence](#735-operator-precedence)
    - [7.3.6 Short Circuiting](#736-short-circuiting)
    - [7.3.7 Equality](#737-equality)
  - [7.4 Kitchen Sink](#74-kitchen-sink)
  - [7.5 Credits](#75-credits)

## 0. Preface

The wing programming language (aka winglang[<sup>RFC</sup>][rfc]) is a general
purpose programming language designed for building applications for the cloud.

What makes wing special? Traditional programming languages are designed around
the premise of telling a single machine what to do. The output of the compiler
is a program that can be executed on that machine. But cloud applications are
distributed systems that consist of code running across multiple machines and
which intimately use various cloud resources and services to achieve their
business goals.

Wing’s goal is to allow developers to express all pieces of a cloud application
using the same programming language. This way, we can leverage the power of the
compiler to deeply understand the intent of the developer and implement them
through the mechanics of the cloud.

[`▲ top`][top]

---

## 1. General

### 1.1 Types

#### 1.1.1 Primitive Types

| Name   | Extra information                         |
| ------ | ----------------------------------------- |
| `nil`  | represents the absence of a value or type |
| `any`  | represents everything and anything        |
| `num`  | represents numbers (doubles)              |
| `str`  | UTF-16 encoded strings                    |
| `bool` | represents true or false                  |

User defined explicit "any" is supported iff declared by the user.  
Implicit "any" resolved by the compiler is a compile error.

> ```TS
> // Wing Code:
> let x = 1;                  // x is a num
> let v = 23.6;               // v is a num
> let y = "Hello";            // y is a str
> let z = true;               // z is a bool
> let w: any = 1;             // w is an any
> let q: num? = nil;          // q is an optional num
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const x: number = 1;
> const v: number = 23.6;
> const y: string = "Hello";
> const z: boolean = true;
> const w: any = 1;
> const q: number? = undefined;
> ```

[`▲ top`][top]

---

#### 1.1.2 Container Types

| Name          | Extra information                            |
| ------------- | -------------------------------------------- |
| `set<T>`      | set type (set of unique items)               |
| `map<T>`      | map type (key-value with string keys)        |
| `list<T>`     | variable size list (array) of a certain type |
| `mut_set<T>`  | mutable set type                             |
| `mut_map<T>`  | mutable map type                             |
| `mut_list<T>` | mutable list (array) type                    |
| `promise<T>`  | promise type (async code)                    |

> ```TS
> // Wing Code:
> let z = {1, 2, 3};          // immutable set (type is set<num>)
> let zm = mut_set<num>();    // mutable set
> let y = {"a": 1, "b": 2};   // immutable map
> let ym = mut_map<num>();    // mutable map
> let x = [1, 2, 3];          // immutable list (type is list<num>)
> let xm = mut_list<num>();   // mutable list (array)
> let w = SampleClass();      // class instance (mutability unknown)
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const z: Set<number> = Object.freeze(new Set([1, 2, 3]));
> const zm: Set<number> = new Set();
> const y: Map<string, number> = Object.freeze(new Map([["a", 1], ["b", 2]]));
> const ym: Map<string, number> = new Map();
> const x: number[] = Object.freeze([1, 2, 3]);
> const xm: number[] = [];
> const w: SampleClass = new SampleClass();
> ```

[`▲ top`][top]

---

#### 1.1.3 Function Types

Function type annotations are written as if they were closure declarations, with
the difference that body is replaced with return type annotation. Phase of the
function is determined with `=>` or `~>` operators. Latter being inflight. `->`
indicates a special type of function that is phase independent. Learn more about
them in the closure section.

```pre
(arg1: <type1>, arg2: <type2>, ...) [=|~|-]> <type>
```

> ```TS
> // Wing Code:
> // type annotation in wing: (num) => num
> let f1 = (x: num): num => { return x + 1; };
> // type annotation in wing: (num, str) ~> nil
> let f2 = (x: num, s: str) ~> { /* no-op */ };
> // type annotation in wing: (num, num) -> nil
> let f3 = (a: num, b: num) -> { print(a + b) };
> ```
> 
> ```TS
> // Equivalent TypeScript Code:
> const f1 = Object.freeze((x: number): number => { return x + 1; });
> const f2 = Object.freeze((x: number, s: string): undefined => { });
> const f3 = Object.freeze((a: number, b: number): undefined => { print(a+b) });
> ```


[`▲ top`][top]

---

### 1.2 Debugging Utilities

| Name     | Extra information                                        |
| -------- | -------------------------------------------------------- |
| `print`  | prints anything serializable.                            |
| `throw`  | creates and throws an instance of an exception           |
| `panic`  | exits with a serializable, dumps the trace + a core dump |
| `assert` | checks a condition and _panics_ if evaluated to false    |

Wing is a statically typed language, so attempting to redefine any of the above
functions, just like any other "symbol" will result in a compile-time error.  

Above functions can accept variadic arguments of any type except `throw` which
only accepts one argument and that is the message to be contained in the error.

"panic" is a fatal call by design. If intention is error handling, panic is the
last resort. Exceptions are non fatal and should be used instead for effectively
communicating errors to the user.

> ```TS
> // Wing Code:
> print(23, "Hello", true);
> throw("a recoverable error occurred");
> panic("a fatal error encountered", [1,2]);
> assert(x > 0, x < 10);
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> console.log(23, "Hello", true);
> // throws
> // calling panic in wing is fatal
> (() => {
>   console.error("Something went wrong", [1,2]);
>   // generate core dump
>   // show stack trace
>   process.exit(1);
> })();
> // multiple assertions
> (() => { assert.ok(x > 0); assert.ok(x < 10); })();
> ```

[`▲ top`][top]

---

### 1.3 Phase Modifiers

In Wing, we differentiate between code that executes during compilation and code
that executes after the application has been deployed by referring to them as
**preflight** and **inflight** code respectively.

The default (and implicit) execution context in Wing is preflight. This is
because in cloud applications, the entrypoint is definition of the app's cloud
infrastructure (and not the code that runs within a specific machine within this
cloud infrastructure).

Phase modifier `~` is allowed in the context of declaring interface and resource
members. Example code is shown in the [resources](#43-resources) section.

Design of language features in Wing loosely follow the design of WebAssembly and
its relation to WASI. Some features are designed to be "compute" and independent
of the underlying phase of the operation. Classes fall into this category. You
may use these features regardless of the execution phase.

Some features on other hand are "phase-aware" and are available during specific
phases and/or their behavior changes. Resources fall into this category.

Bridge between preflight and inflight is crossed with the help of immutable data
structures, "structs", and capture mechanic in Wing.

[`▲ top`][top]

---

### 1.4 Storage Modifiers

A storage modifier is a keyword that specifies the placement of a function or
variable in the program memory once compiled. Some declarations might have a
temporary storage (such as a local closure definition), while others might have
a permanent storage (such as a global variable).

Currently the only storage modifier is `static`. `static` indicates a definition
is only available once per program and for the entire duration of that program.
All statics must be defined inline and initialized right away.  
Statics are not allowed on structs or interfaces.

Statics are both supported in inflight as well as preflight mode of execution.

A declaration for a static member is a member declaration whose declaration
specifiers contain the keyword static. The keyword static must appear before
other specifiers. More details in the [classes](#42-classes) section.

The name of any static data member and static member function must be different
from the name of the containing class regardless of the casing.

Code samples for `static` are not shown here. They are shown in the relevant
sections below.

To avoid confusion, it is invalid to have a static and a non-static with the
same name. Overloading a static is allowed however.  
Accessing static is done via the type name and the `.` operator.

[`▲ top`][top]

---

### 1.5 Access Modifiers

Visibility inference is done with the following rules:

- if symbols' name starts with an underscore, visibility is `private`.
- if symbols' name does not start with an underscore and lacks any other access
  modifier, its visibility is `public`.
- `protected`: visibility is "protected" (public to self and derived classes).
  In this case the symbol _must_ not begin with an underscore (since "protected"
  is part of the public api of the module).
- `internal`: visibility is "internal" (public to current compilation unit). In
  this case, the symbol _must_ begin with an underscore (since "internal"s are
  not part of the public api of the module). This is also aligned with JSII
  internal.

Accessing fields, members, or structured data is done with `.`.

Visibility modifiers can be applied to members of classes and resources.  
Mixing `protected` and `internal` is not allowed.

[`▲ top`][top]

---

### 1.6 Mutability

In Wing we have two classes of "data": immutable and mutable.

Immutable data is important in Wing because it can cross machine boundaries
safely in a distributed system. In Wing, this implies that only immutable data
is capture-able from preflight safely into inflight.

All primitive types are immutable (`bool`, `str`, `num`, and `nil`).  
Resources are also immutable by design and can be captured into inflight.  
Immutable container types (`map`, `set` and `list`) as well as structs of
immutable types are also immutable.

Optionality modifier `?` applied to any type does not change the mutability of
the underlying type. Read more about optionality in its section below.

Re-assignment to variables that are defined with `let` is not allowed in Wing.
Re-assignment to class fields is allowed if field is marked with `rw`.  
Examples in the class section below.

`rw` is available in the body of class declarations.  
Assigning `rw` to immutables of the same type is allowed.

As a result of classes being able to represent mutable data, capturing them into
inflight is not supported. However, you may declare new classes inside inflight
code and have new objects instantiated in inflight.

[`▲ top`][top]

---

### 1.7 Optionality

Symbol `?` can mark a type as optional.  
Optionality means the value behind type can be either present or nil.

Rules of optionality apply to the entire new container type of `type?` and not
the value behind it (`type`).  

Using the `??` operator allows one to safely access the value behind an optional
variable by always providing a default, in case the variable is nil. This forces
a value to be present for the l-value (left hand side of the assignment operator
and guarantees what's returned is type-stripped from its `?` keyword).

> ```TS
> // Wing Code:
> let x? = 44;
> let y = x ?? 55;
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const x: number? = 44;
> const y: number = x ?? 55;
> ```

[`▲ top`][top]

---

### 1.8 Type Inference

Type can optionally be put between name and the equal sign, using a colon.  
Partial type inference is allowed while using the `?` keyword immediately after
the variable name.

When type annotation is missing, type will be inferred from r-value type.  
r-value refers to the right hand side of an assignment here.

All defined symbols are immutable (constant) by default.  
Type casting is generally not allowed unless otherwise specified.

Function arguments and their return type is always required. Function argument
type is inferred iff a default value is provided.

> ```TS
> // Wing Code:
> let i = 5;
> let m = i;
> let arr_opt? = list<num>();
> let arr: list<num> = [];
> let copy = arr;
> let i1? = nil;
> let i2: num? = i;
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const i: number = 5;
> const m: number = i;
> const arr_opt: number[]? = [];
> const arr: number[] = Object.freeze([]);
> const copy: number[] = Object.freeze([...arr]);
> const i1: any = undefined;
> const i2: number? = i;
> ```

[`▲ top`][top]

---

### 1.9 Error Handling

Exceptions and `try/catch/finally` is the error mechanism. Mechanics directly
translate to JavaScript. If exception is uncaught, it crashes your app with a
`panic` call. You can create a new exception with `throw`.

In the presence of `try`, `catch` is required but `finally` is optional.

`panic` is meant to be fatal error handling.  
`throw` is meant to be recoverable error handling.

An uncaught exception is considered user error but a panic call is not. Compiler
must guarantee exception safety by throwing a compile error if an exception is
expected from a call and it is not being caught.

> ```TS
> // Wing Code:
> try {
>   let x? = 1;
>   throw("hello exception");
> } catch e {
>   print(e);
> } finally {
>   print("done);
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> try {
>   let x: number? = 1;
>   throw new Error("hello exception");
> } catch (e) {
>   console.log(e);
> } finally {
>   console.log("done");
> }
> ```

[`▲ top`][top]

---

### 1.10 Formatting

Wing is opinionated about formatting and whitespace. The opinion is:

- indentations of lines are 2 spaces
- each statement must end with a semicolon
- interface names start with capital letter "I"
- enum members must be written in ALL_CAPS_SNAKE_CASE
- class, struct, interface, and resource names must be TitleCased
- every other declaration name must be snake_cased unless otherwise specified
- open curly brace must be at the same line of any declaration that requires it

If you are reading this section as a user and you are caught off guard by this
section, the philosophy behind this section is to end all unnecessary formatting
debates and instead make developers focus on writing the logic of their code.

We, as in Wing developers, find ourselves constantly applying this formatting to
languages that are not even opinionated about it. Therefore, we seize the moment
to make sure Wing code looks and feels uniform across the board.

Formatting is enforced by the compiler, however this is not enforced IFF it
occurs in a non entrypoint `.w` file AND the offending code is a direct import
and usage of a JSII module. This is to ensure isolated compatibility with third
party JSII modules. Users cannot explicitly turn formatting off. Compiler must
emit warnings for all implicit JSII formatting violations.

[`▲ top`][top]

---

### 1.11 Memory Management

There is no implicit memory de-allocation function, dynamic memory is managed by
Wing and is garbage collected (relying on JSII target GC for the meantime).

[`▲ top`][top]

---

### 1.12 Documentation Style

Wing currently leverages @JSDoc style comments in its format called the "runway"
comment format. "Runway" refers to what the left hand side of the comment block
represents. It vaguely resembles an airport runway!

> ```TS
> /*\
> |*|  Document your code with meaningful comments.
> |*|
> |*|  You can use Markdown for formatting.
> |*|  Compiler can generate documentation for you from jsdoc tags.
> \*/
> ```

[`▲ top`][top]

---

### 1.13 Execution Model

Execution model currently is delegated to the JSII target. This means if you are
targeting JSII with Node, Wing will use the event based loop that Node offers.

In Wing, writing and executing at root block scope level is forbidden except for
the entrypoint of the program. Root block scope is considered special and
compiler generates special instructions to properly assign all resources to
their respective scopes recursively down the constructs tree based on entry.

Entrypoint is always a wing source with an extension of `.w`. Within this entry
point, a root resource is made available for all subsequent resources that are
initialized and instantiated. Type of the root resource is determined by the
target being used by the compiler. The root resource might be of type `App` in
AWS CDK or `TerraformApp` in case of CDK for Terraform target.

> Following "shimming" is only done for the entrypoint file and nowhere else.
> Type of the "shim" changes from `cdk.Stack` to `TerraformStack` for cdk-tf.

> ```TS
> // Wing Entrypoint Code:
> let a = MyResource();
> let b = MyResource() be "my-resource";
> ```
> 
> ```TS
> // Equivalent TypeScript Code:
> (new class extends cdk.Stack {
>   constructor(scope: constructs.Construct, id: string) {
>     const a = new MyResource(this, "MyResource");
>     const b = new MyResource(this, "my-resource");
>   }
> })(new cdk.App(), "WingEntry");
> ```

[`▲ top`][top]

---

## 2. Expressions

### 2.1 bring expression

"bring" expression can be used to import and reuse code from other Wing files or
other JSII supported languages. The expression is detailed in its own section in
this document: [Module System](#5-module-system).

[`▲ top`][top]

---

### 2.2 break expression

**break** expression allows to end execution of a cycle. This includes for and
while loops currently.

> ```TS
> // Wing Code:
> for let i in 1..10 {
>   if i > 5 {
>     break;
>   }
>   print(i);
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> for (let i = 1; i < 10; i++) {
>   if (i > 5) {
>     break;
>   }
>   console.log(i);
> }
> ```

[`▲ top`][top]

---

### 2.3 continue expression

**continue** expression allows to skip to the next iteration of a cycle. This
includes for and while loops currently.

> ```TS
> // Wing Code:
> for let i in 1..10 {
>   if i > 5 {
>     continue;
>   }
>   print(i);
> }
> ```
>   
> ```TS
> // Equivalent TypeScript Code:
> for (let i = 1; i < 10; i++) {
>   if (i > 5) {
>     continue;
>   }
>   console.log(i);
> }
> ```

[`▲ top`][top]

---

### 2.4 return expression

**return** expression allows to return a value or exit from a called context.  
In case of a missing "return" statement in a method definition, the method will
return `nil` implicitly and inherently its return type becomes also `nil` if no
other type annotation with `?` is provided.

> ```TS
> // Wing Code:
> class MyClass {
>   myPublicMethod() {}
>   _myPrivateMethod(): nil {}
>   protected myProtectedMethod(): nil { return nil; }
>   internal _myInternalMethod(): str { return "hi!"; }
> }
> ```
> 
> ```TS
> // Equivalent TypeScript Code:
> class MyClass {
>   public myPublicMethod(): void {}
>   private myPrivateMethod(): undefined {}
>   protected myProtectedMethod(): undefined { return undefined; }
>   // specific compiled instruction is up to implementation of the compiler
>   public __wing__internal_myInternalMethod(): string { return "hi!"; }
> }
> ```

[`▲ top`][top]

---

### 2.5 await Expression

**await** expression allows to wait for a promise and grab its execution result.
"await" and "promise" are semantically similar to JavaScript's promises.  
"await" expression is only valid in `async` function declarations.  
awaiting non promises in Wing is a no-op just like in JavaScript.

> ```TS
> // Wing program:
> class MyClass {
>   async foo(): num {
>     let x = await some_promise();
>     return x;
>   }
>   boo(): promise<num> {
>     let x = some_promise();
>     return x;
>   }
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> class MyClass {
>   async foo(): number {
>     let x = await some_promise();
>     return x;
>   }
>   boo(): Promise<number> {
>     let x = some_promise();
>     return x;
>   }
> }
>  ```

[`▲ top`][top]

---

## 3. Statements

### 3.1 if statement

Flow control can be done with `if/elif/else` statements.  
The `if` statement is optionally followed by `elif` and `else`.  
"If" statement condition expression in Wing is not surrounded by parenthesis.

> ```TS
> // Wing program:
> let x = 1;
> let y = "sample";
> if x == 2 {
>   print("x is 2");
> } elif y != "sample" {
>   print("y is not sample");
> } else {
>   print("x is 1 and y is sample");
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const x: number = 1;
> const y: string = "sample";
> if (x === 2) {
>   console.log("x is 2");
> } else if (y !== "sample") {
>   console.log("y is not sample");
> } else {
>   console.log("x is 1 and y is sample");
> }
> ```

[`▲ top`][top]

---

### 3.2 for statement

`for..in` statement is used to iterate over a list or set.  
type annotation after an iteratee (left hand side of `in`) is optional.  
"For" statement condition expression in Wing is not surrounded by parenthesis.

> ```TS
> // Wing program:
> let arr = [1, 2, 3];
> let set = {1, 2, 3};
> for item in arr {
>   print(item);
> }
> for item: num in set {
>   print(item);
> }
> for item in 0..100 {
>   print(item);
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const arr: number[] = Object.freeze([1, 2, 3]);
> const set: Set<number> = Object.freeze(new Set([1, 2, 3]));
> for (const item of arr) {
>   console.log(item);
> }
> for (const item of set) {
>   console.log(item);
> }
> // calling 0..100 does not allocate, just returns an iterator
> function* iterator(lim) { let i = lim; while (i--) yield i; }
> const iter = iterator(100);
> for (const val of iter) {
>   console.log(val);
> }
> ```

[`▲ top`][top]

---

### 3.3 while statement

while statement is used to execute a block of code while a condition is true.  
"While" statement condition expression in Wing is not surrounded by parenthesis.

> ```TS
> // Wing program:
> while call_some_function() {
>   print("hello");
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> while (call_some_function()) {
>   console.log("hello");
> }
> ```

[`▲ top`][top]

---

## 4. Declarations

### 4.1 Structs

Structs are loosely modeled after typed JSON literals in JavaScript.  
Structs are defined with the `struct` keyword.  
Structs are "bags" of immutable data.

Structs can only have fields of primitive types, resources, and other structs.  
List, set, and map of above types is also allowed in struct field definition.  
Visibility, storage and phase modifiers are not allowed in struct fields.

Structs can inherit from multiple other structs.

> ```Rust
> // Wing program:
> struct MyDataModel1 {
>   field1: num;
>   field2: str;
> };
> struct MyDataModel2 {
>   field3: num;
>   field4: bool?;
> };
> struct MyDataModel3 extends MyDataModel1, MyDataModel2 {
>   field5: str;
> }
> let s1 = MyDataModel1 { field1: 1, field2: "sample" };
> let s2 = MyDataModel2 { field3: 1, field4: true };
> let s3 = MyDataModel2 { field3: 1, field4: nil };
> let s4 = MyDataModel3 {
>   field1: 12,
>   field2: "sample", 
>   field3: 11,
>   field4: false,
>   field5: "sample"
> };
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> interface MyDataModel1 {
>   public readonly field1: number;
>   public readonly field2: string;
> }
> interface MyDataModel2 {
>   public readonly field3: number;
>   public readonly field4?: boolean;
> }
> interface MyDataModel3 extends MyDataModel1, MyDataModel2 {
>   public readonly field5: string;
>   public readonly field6: number;
> }
> const s1: MyDataModel1 = { field1: 1, field2: "sample" };
> const s2: MyDataModel2 = { field3: 1, field4: true };
> const s3: MyDataModel2 = { field3: 1, field4: undefined };
> const s4: MyDataModel3 = {
>   field1: 12,
>   field2: "sample",
>   field3: 11,
>   field4: false,
>   field5: "sample",
>   field6: 11
> };
> ```

[`▲ top`][top]

---

### 4.2 Classes

Classes consist of fields and methods in any order.  
The class system is single-dispatch class based object orientated system.

A class member function that has the name **init** is considered to be a class
constructor (or initializer, or allocator).

```TS
class Name extends Base impl MyInterface1, MyInterface2 {
  init() {
    // constructor implementation
    // order is up to user
    this._field1 = 1;
    this._field2 = "sample";
  }

  // class fields (private by due to having leading underscore)
  _field1: num;
  _field2: str;

  // static method (access with Name.static_method(...))
  static static_method(arg: type, arg: type, ...) { /* impl */ }
  // private method
  _private_method(arg: type, arg: type, ...): type { /* impl */ }
  // visible to outside the instance
  public_method(arg:type, arg:type, ...) { /* impl */ }
  // visible to children only
  protected protected_method(type:arg, type:arg, ...) { /* impl */ }
  // public in current compilation unit only
  internal _internal_method3(type:arg, type:arg, ...): type { /* impl */ }
}
```

Default initialization does not exist in Wing. All member fields must be
initialized in the constructor. Absent initialization is a compile error.

Member function and field access in constructor with the "this" keyword before
all fields are initialized is invalid and should throw a compile error.

In other words, the `this` keyword is immutable to its field access operator `.`
before all the member fields are properly initialized. The behavior is similar
to JavaScript and TypeScript in their "strict" mode.

```TS
class Foo {
  x: num;
  init() { this.x = 1; }
}
class Bar {
  y: num;
  z: Foo;
  init() {
    // this.print() // is compile error here
    this.y = 1;
    // this.print() // is also compile error here
    this.z = Foo();
    this.print(); // OK to call here
  }
  public print() {
    print(this.y);
  }
}
let a = Bar();
a.print(); // prints 20.
```

Overloading methods is allowed. This means functions can be overloaded with many
signatures only varying in the number of arguments and their unique type order.
Overloading the constructor is also allowed.  
Inheritance is allowed with the `extends` keyword. `super` can be used to access
the base class, immediately up the inheritance chain (parent class).

Calling using the member access operator `.` before calling `super` in inherited
classes is forbidden. The behavior is similar to JavaScript and TypeScript in
their "strict" mode.

```TS
class Foo {
  x: num;
  init() { this.x = 0; }
  public method() { }
}
class Boo extends Foo {
  init() {
    // this.x = 10; // compile error
    super();
    this.x = 10; // OK
  }
  public override method() {
    // override implementation
  }
}
```

Classes can inherit and extend other classes using the `extends` keyword.  
Classes can implement interfaces iff the interfaces does not contain `~`. You
can use the keyword `final` to stop the inheritance chain.

```TS
class Foo {
  x: num;
  init() { this.x = 0; }
  public method() { }
}
final class Boo extends Foo {
  init() { super(); this.x = 10; }
  public override method() {
    // override implementation
  }
}
// compile error
// class FinalBoo extends Boo {}
```

By default all methods are virtual. But if you are about to override a method,
you need to explicitly provide the keyword **override**.  
Static, private, and internal methods cannot be and are not virtual.  

Statics are not inherited.  
As a result, statics can be overridden mid hierarchy chain. Access to statics is
through the class name that originally defined it `<class name>.Foo`.  

Child class must not introduce additional signatures (overloads) for overridden
(virtual) methods.

Multiple inheritance is invalid and forbidden.  
Multiple implementations of various interfaces is allowed.  
Multiple implementations of the same interface is invalid and forbidden.

In methods if return statement is missing, `return nil` is assumed.  
In methods if return type is missing, `: nil` is assumed.

[`▲ top`][top]

---

### 4.3 Resources

Resources provide first class composite pattern support in Wing. They are
modeled after and leverage the [constructs](https://github.com/aws/constructs)
programming model and as such are fully interoperable with CDK constructs.  
Resources can be defined like so:

```TS
// Wing Code:
resource Foo {
  init() { /* initialize preflight fields */ } // preflight constructor
  ~ init() {} // optional client initializer
  fin() {} // optional sync finalizer
  async fin() {} // async finalizer (can be either sync or async)

  // phase independent fields (advanced usage only)
  - foo(arg: num): num { return arg; }

  // inflight members
  ~ foo(arg: num): num { return arg; }
  ~ boo(): num { return 32; }

  // inflight fields
  ~ field1: num;
  ~ field2: str;
  ~ field3: bool;

  // preflight members
  foo(arg: num): num { return arg; }
  boo(): num { return 32; }
  
  // preflight fields
  field1: num;
  field2: str;
  field3: bool;

  // re-assignable class fields, read about them in the mutability section
  rw field4: num;
  rw field5: str;
}
```

Resources all have a scope and a unique ID. Compiler provides an implicit scope
and ID for each resource, both overrideable by user-defined ones in constructor.

The default for scope is `this`, which means the scope in which the resource was
defined. The implicit ID is the type name of the resource iff the resource type
is the only resource type being used in the current scope. In other words, if
there are multiple resources of the same type defined in the same scope, they
must all have an explicit id.

Resources instantiated at block scope root level of entrypoint are assigned the
root app construct as their default implicit scope.

Resource instantiation syntax uses the `let` keyword the same way variables are
declared in Wing. The difference is `be/in` keywords afterwards.

```pre
let <name>[: <type>] = <resource> [be <id>] [in <scope>];
```

```TS
// Wing Code:
let a = Foo(); // with default scope and id
let a = Foo() in scope; // with user-defined scope
let a = Foo() be "custom-id" in scope; // with user-defined scope and id
let a = Foo(...) be "custom-id" in scope; // with constructor arguments
```

"id" must be of type string. It can also be a string literal with substitution
support (normal strings as well as shell strings).  
"scope" must be a variable of resource type.

In addition to the `init` keyword for defining initializers, resources have a
unique `fin` definable method that offers async finalization of a resource in
preflight time.  
Order of execution of async finalization is not guaranteed.

Resources can be captured into inflight functions and once that happens, inside
the capture block only the inflight members are available.

Resources can extend other resources (but not structs) and implement interfaces.

[`▲ top`][top]

---

### 4.4 Interfaces

Interfaces represent a contract that a class or resource must fulfill.  
Interfaces are defined with the `interface` keyword.  
Both preflight and inflight signatures are allowed.  
`impl` keyword is used to implement an interface or multiple interfaces that are
separated with commas.

All methods of an interface are implicitly public and cannot be of any other
type of visibility (private, protected, etc.).

> ```TS
> // Wing program:
> interface IMyInterface1 {
>   field1: num;
>   method1(x: num): str;
> };
> interface IMyInterface2 {
>   ~ field2: str;
>   ~ method2(): str;
> };
> resource MyResource impl IMyInterface1, IMyInterface2 {
>   ~ field2: str;
>   ~ init(x: num) {
>     // inflight client initialization
>     this.field2 = "sample";
>     this.field1 = x;
>   }
>   method1(x: num): str {
>     return "sample: ${x}";
>   }
>   ~ method2(): str {
>     return this.field2;
>   }
> };
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> interface MyInterface1 {
>   public readonly field1: number;
>   public method1(x: number): string;
> }
> interface MyInterface2 {
>   public readonly __inflight__field2: string;
>   public __inflight__method2(): string;
> }
> // this is only shown as a hypothetical sample
> class MyResource extends constructs.Construct
>   implements MyInterface1, MyInterface2 {
>   public readonly field1: number;
>   public readonly __inflight__field2: string;
>   public __inflight__constructor() {
>     // inflight client initialization
>     this.field1 = x;
>     this.__inflight__field2 = "sample";
>   }
>   public __inflight__method2(): string {
>     return this.__inflight__field2;
>   }
>   public method1(x: number): string {
>     return `sample: ${x}`;
>   }
> }
> ```

[`▲ top`][top]

---

### 4.5 Variables

> Let let be let. (Elad B. 2022)

```pre
let <name>[: <type>] = <value>;
```

Assignment operator is `=`.  
Assignment declaration keyword is `let`.  
Type annotation is optional if a default value is given.  

> ```TS
> // Wing Code:
> let n = 10;
> let s: str = "hello";
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const n: number = 10;
> const s: string = "hello";
> ```

[`▲ top`][top]

---

### 4.6 Functions

#### 4.6.1 Closures

It is possible to create closures.  
It is not possible to create named closures.  
However, it is possible to create anonymous closures and assign to variables
(function literals). Inflight closures are also supported.

> ```TS
> // Wing Code:
> // preflight closure:
> let f1 = (a: num, b: num) => { print(a + b) };
> // inflight closure:
> let f2 = (a: num, b: num) ~> { print(a + b) };
> // phase independent closure:
> let f2 = (a: num, b: num) -> { print(a + b) };
> // OR:
> // preflight closure:
> let f4 = (a: num, b: num): nil => { print(a + b) };
> // inflight closure:
> let f5 = (a: num, b: num): nil ~> { print(a + b) };
> // phase independent closure:
> let f6 = (a: num, b: num): nil -> { print(a + b) };
> ```

`->` closure types are special in which the user can write phase independent or
"phase-shared" closures. These closures cannot consume any resources, neither
can they return any resources. They do pure "compute" operations.

[`▲ top`][top]

---

#### 4.6.2 Promises

Promises in Wing are defined with `promise<T>` syntax.  
Functions that use the keyword "await" in their body must return a promise.

> ```TS
> // Wing Code:
> let number = (): promise<num> => {
>   return 23;
> }
> let handler = (): promise<nil> => {
>   let t = await number();
>   print(t);
> }
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const number = Object.freeze((): number {
>   return 23;
> })
> const handler = Object.freeze((): undefined {
>   const t: number = await number();
>   console.log(t);
> })
> ```

[`▲ top`][top]

---

#### 4.6.3 Struct Expansion

If the last argument of a function call is a struct, then the struct in the call
is "expandable" with a special `:` syntax.  
In this calling signature, order of struct members do not matter.  
Partial struct expansion is not allowed.

```TS
struct MyStruct {
  field1: num;
  field2: num;
};
let f = (x: num, y: num, z: MyStruct) => {
  print(x + y + z.a + z.b);
}
// last arguments are expanded into their struct
f(1, 2, a: 3, b: 4);
```

[`▲ top`][top]

---

#### 4.6.4 Variadic Arguments

If the last argument of a function type is the `...args` keyword followed by a
`list` type, then the function accepts typed variadic arguments. Expansion of
variadic arguments is not supported currently and the container of variadic
arguments is accessible with the `args` key like a normal list instance.

```TS
let f = (x: num, ...args: list<num>) => {
  print(x + y + sizeof(args));
}
// last arguments are expanded into their struct
f(1, 2, 3, 4, 5, 6, 34..100);
```

[`▲ top`][top]

---

### 4.7 Lists

Lists are dynamically sized in Wing and are defined with the `[]` syntax.  
Individual list items are also accessed with the `[]` syntax. You can call
`sizeof` to get the size of the list.  
Lists are similar to dynamically sized arrays or vectors in other languages.

> ```TS
> // Wing Code:
> let list1 = [1, 2, 3];
> let list2 = ["a", "b", "c"];
> let list3 = list(list2);
> let l = sizeof(list1) + sizeof(list2) + sizeof(list3) + list1[0];
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const list1: number[] = Object.freeze([1, 2, 3]);
> const list2: string[] = Object.freeze(["a", "b", "c"]);
> const list3: string[] = ["a1", "b2", "c3"];
> const l = list1.length + list2.length + list3.length + list1[0];
> ```

[`▲ top`][top]

---

### 4.8 Enumeration

Enumeration type (enum) is a type that groups a list of named constant members.
Enumeration is defined by writing **enum**, followed by enumeration name and a
list of comma-separated constants in a {}. Last comma is optional in single line
definitions but required in multi line definitions.  
Naming convention for enums is to use "TitleCase" for name ALL_CAPS for members.

> ```TS
> // Wing Code:
> enum SomeEnum { ONE, TWO, THREE };
> enum MyFoo {
>   A,
>   B,
>   C,
> };
> let x: enum<MyFoo> = MyFoo.B;
> let y = x; // type is enum<MyFoo>
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> enum SomeEnum { ONE, TWO, THREE };
> enum MyFoo {
>   A,
>   B,
>   C,
> };
> const x: MyFoo = MyFoo.B;
> const y: MyFoo = x;
> ```

`nameof` operator is used to get the name of a constant member at compile time.
For example `nameof(MyEnum.MEMBER)` resolves to `"MEMBER"` at compile time.

This allows painless conditionals when enums are serialized and deserialized
over the wire without littering the source with strings everywhere. Compare:

```TS
// Wing Code:
enum SomeEnum { ONE, TWO, THREE };
let some_val: str = "ONE";
if some_val == nameof(SomeEnum.ONE) {
  // whatever1
} elif some_val == nameof(SomeEnum.TWO) {
  // whatever2
}
```

Which is functionally equivalent to:

```TS
// Wing Code:
enum SomeEnum { ONE, TWO, THREE };
let some_val: str = get_enum_serialized_from_network();
if some_val == "ONE" {
  // whatever1
} elif some_val == "TWO" {
  // whatever2
}
```

[`▲ top`][top]

---

### 4.9 Computed Properties

You may use the following syntax to define computed properties.  
Computed properties are syntactic sugar for getters and setters which themselves
are syntactic sugar for methods, therefore omitting either one is acceptable.  

"Read block" must always return a value of the same type as the property.

Keyword `new` can be used inside the write block to access the incoming r-value
of the assignment (write) operation. `new` is always the same type as the type
of the property itself.  

Both preflight and inflight computed properties are allowed.  
Keyword `rw` behind computed properties is not allowed.  
Async computed properties are not allowed.

```TS
// Wing Code:
struct Vec2 {
  x: num;
  y: num;
}

class Rect {
  rw size: Vec2;
  rw origin: Vec2;
  // computed property with a getter and setter block
  center: Vec2 {
    read {
      let center_x = origin.x + (size.width / 2);
      let center_y = origin.y + (size.height / 2);
      return Vec2(x: center_x, y: center_y);
    }
    write {
      origin.x = new.x - (size.width / 2);
      origin.y = new.y - (size.height / 2);
    }
  };
}
```

```TS
// Equivalent TypeScript Code:
interface Vec2 { x: number; y: number; }
class Rect {
  size: Vec2;
  origin: Vec2;
  // computed property with a getter and setter block
  get center(): Vec2 {
    let center_x = origin.x + (size.width / 2);
    let center_y = origin.y + (size.height / 2);
    return Vec2(x: center_x, y: center_y);
  }
  set center(_new: Vec2) {
    origin.x = _new.x - (size.width / 2);
    origin.y = _new.y - (size.height / 2);
  }
}
```

[`▲ top`][top]

---

## 5. Module System

The module system in Wing uses the "bring" expression to reuse code.  
**bring** expression allows code to "import" functions, classes and variables
from other files, to allow reusability.  
**bring** expression is only allowed at the top of the file before any other
code. Comments before the first bring expression are valid.

### 5.1 Imports

#### 5.1.1 Verbose Notation

"bring" expression starts with `from` keyword and followed by a file path,
either with or without an extension. If there is no extension, file path is
treated as a Wing import, otherwise it is treated as a JSII import.  
In case of Wing, quotes around the file path are optional.  
The expression is followed by a `bring` keyword and a list of names to import.  
Names can be renamed with `as` keyword.  
`bring *;` is invalid. It is not possible to import an entire namespace unbound.

> ```TS
> // Wing Code:
> from std bring * as std2;
> from std bring io;
> from std bring io as io2;
> from std bring io, fs, name as module;
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> import * from 'std';
> import * as std2 from 'std';
> import { io } from 'std';
> import { io as io2 } from 'std';
> import { io, fs, name as module } from 'std';
> ```

To promote polyglot programming, A string literal can also be placed after
**bring**. This allows importing and reusing code from JSII packages:

> ```TS
> // Wing Code:
> from "cdk-spa" bring * as spa;
> from "cdk-spa" bring SomeConstruct;
> from "cdk-spa" bring SomeConstruct as SomeConstruct2;
> from "cdk-spa" bring SomeConstruct, OtherType as module;
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> import * from 'cdk-spa';
> import * as spa from 'cdk-spa';
> import { SomeConstruct } from 'cdk-spa';
> import { SomeConstruct as SomeConstruct2 } from 'cdk-spa';
> import { SomeConstruct, OtherType as module } from 'cdk-spa';
> ```

[`▲ top`][top]

---

#### 5.1.2 Shorthand Notation

The verbose notation of `from <module> bring * as <name>` can be shortened to
`bring <module>` in case of importing Wing code and `bring <jsii> as <name>` for
JSII imports across your Wing source code.

```TS
bring std; // from std bring * as std;
bring cloud; // from cloud bring * as cloud;
bring "path/to/what.js" as what; // from "path/to/what.js" bring * as what;
```

[`▲ top`][top]

---

### 5.2 Exports

In preflight, anything with `public` at block scope level is importable.  
This includes functions, classes, structs, interfaces and resources.  
In inflight, the above excluding resources are importable.  
Variables are not exportable.

Resources are not usable in inflight functions. There is no synthesizer inside
and no deployment system in the inflight body to synthesize inflight resources.

[`▲ top`][top]

---

## 6. JSII Interoperability

You may import JSII modules in Wing and they are considered resources if their
JSII type manifest shows that the JSII module is a construct. Wing is a consumer
of JSII modules currently.

```ts
bring "aws-cdk-lib" as cdk;
let bucket = cdk.aws_s3.Bucket(
  public_access: true,
);
```

[`▲ top`][top]

---

## 7. Miscellaneous

### 7.1 Strings

Type of string is UTF-16 internally.  
All string declaration variants are multi-line.  
You can call `sizeof` to get the length of the string.

[`▲ top`][top]

---

#### 7.1.1 Normal strings "..."

The string inside the double quotes is processed, and all notations of form
`${<expression>}` are substituted from their respective scopes. The behavior is
similar to `` `text ${sub.prop}` `` notation in JavaScript.  
Processing unicode escape sequences happens in these strings.  
`"` and `$` can be escaped with backslash `\` inside string substitutions.

> ```TS
> // Wing Code:
> let name = "World";
> let s = "Hello, ${name}!";
> let l = sizeof(s);
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const name = "World";
> const s = `Hello, ${name}!`; // with substitution
> const l = s.length; // length of string
> ```

[`▲ top`][top]

---

#### 7.1.2 Shell strings \`...\`

If string is enclosed with backticks, the contents of that string will be
interpreted as a shell command and its output will be used as a string. `` `echo
"Hello"` `` is equal to `"Hello"`.  
The string inside the backtick is evaluated in shell at compile time and its
stdout is returned as a string. If command exits with non-zero, this throws with
an exception containing the stderr of the command and its return code.

The string is evaluated at compile time as a escape hatch for ops workflows.

Substitution is not allowed in shell strings.  
Shell strings are invalid in the bring expression.  
Not all targets support shell execution. Backticks throw in absence of a shell.
This is specifically geared towards WebAssembly builds of Wing.

Internally compiler calls the host environment's command processor (e.g.
`/bin/sh`, `cmd.exe`, `command.com`) with the enclosed command.

> ```TS
> // Wing Code:
> let name = `echo "World"`;
> let s = "Hello, ${name}!";
> ```
>
> ```TS
> // Equivalent TypeScript Code:
> const name = (() => {
>   let _stdout = "";
>   try {
>     const { stdout, stderr, status } = spawnSync('echo', ['hello'], {
>       shell: true
>     });
>     if (status !== 0) {
>       throw new Error(stderr.toString());
>     } else {
>       _stdout = stdout.toString();
>       return _stdout;
>     }
>   } catch(error){
>     console.error('Error', error.message);
>     throw error;
>   }
> })();
> let s = `Hello, ${name}!`;
> ```

[`▲ top`][top]

---

### 7.2 Comments

Single line comments start with a `//` and continue to the end of the line.  
Multi-line comments are supported with the `/* ... */` syntax.  
Commenting in Wing has a style that's described earlier in this document. 

> ```TS
> // comment
> /* comment */
> /*
>    multi line comment
> */
> ```

[`▲ top`][top]

---

### 7.3 Operators

Unary operators are not supported except outline below.  
Arithmetic assignment operators are not supported.  
Ternary or conditional operators are not supported.

#### 7.3.1 Relational Operators

| Operator | Description                                      | Example  |
| -------- | ------------------------------------------------ | -------- |
| `==`     | Checks for equality                              | `a == b` |
| `!=`     | Checks for inequality                            | `a != b` |
| `>`      | Checks if left is greater than right             | `a > b`  |
| `<`      | Checks if left less than right                   | `a < b`  |
| `>=`     | Checks if left is greater than or equal to right | `a >= b` |
| `<=`     | Checks if left is less than or equal to right    | `a <= b` |

[`▲ top`][top]

---

#### 7.3.2 Logical Operators

| Operator | Description          | Example    |
| -------- | -------------------- | ---------- |
| `&&`     | Logical AND operator | `a && b`   |
| `\|\|`   | Logical OR operator  | `a \|\| b` |
| `!`      | Logical NOT operator | `!a`       |

[`▲ top`][top]

---

#### 7.3.3 Bitwise Operators

| Operator | Description                 | Example  |
| -------- | --------------------------- | -------- |
| `&`      | Binary AND                  | `a & b`  |
| `\|`     | Binary OR                   | `a \| b` |
| `^`      | Binary XOR                  | `a ^ b`  |
| `~`      | Binary One's Complement     | `~a`     |
| `<<`     | Binary Left Shift Operator  | `a << 1` |
| `>>`     | Binary Right Shift Operator | `a >> 1` |

[`▲ top`][top]

---

#### 7.3.4 Mathematics Operators

| Operator | Description    | Example |
| -------- | -------------- | ------- |
| `*`      | Multiplication | `a * b` |
| `/`      | Division       | `a / b` |
| `\`      | Floor Division | `a \ b` |
| `%`      | Modulus        | `a % b` |
| `+`      | Addition       | `a + b` |
| `-`      | Subtraction    | `a - b` |
| `^`      | Exponent       | `a ^ b` |

[`▲ top`][top]

---

#### 7.3.5 Operator Precedence

| Operator             | Notes                                             |
| -------------------- | ------------------------------------------------- |
| ()                   | Parentheses                                       |
| +x, -x, ~x           | Unary plus, Unary minus, Bitwise NOT              |
| \*, /, \\, %         | Multiplication, Division, Floor division, Modulus |
| +, -                 | Addition, Subtraction                             |
| <<, >>               | Bitwise shift operators                           |
| &                    | Bitwise AND                                       |
| ^                    | Bitwise XOR                                       |
| \|                   | Bitwise OR                                        |
| ==, !=, >, >=, <, <= | Comparisons, Identity, operators                  |
| !                    | Logical NOT                                       |
| &&                   | Logical AND                                       |
| \|\|                 | Logical OR                                        |

Table above is in descending order of precedence.  
`=` operator in Wing does not return a value so you cannot do `let x = y = 1`.  
Operators of the same row in the table above have precedence from left to right
in the expression they appear in (e.g. `4 * 2 \ 3`). In other words, order is
determined by associativity.

[`▲ top`][top]

---

#### 7.3.6 Short Circuiting

For the built-in logical NOT operators, the result is `true` if the operand is
`false`. Otherwise, the result is `false`.

For the built-in logical AND operators, the result is `true` if both operands
are `true`. Otherwise, the result is `false`. This operator is short-circuiting
if the first operand is `false`, the second operand is not evaluated.

For the built-in logical OR operators, the result is `true` if either the first
or the second operand (or both) is `true`. This operator is short-circuiting if
the first operand is `true`, the second operand is not evaluated.

Note that bitwise logic operators do not perform short-circuiting.

In conditionals, if an optional type is used as the only r-value expression of
the condition statement, it's equivalent to checking it against `nil`. Note that
using a `bool?` type in this short-circuit is a compile error due to ambiguity.

```TS
let x? = 1;
if x {
  // ...
}
```

Which is equivalent to:

```TS
let x? = 1;
if x != nil {
  // ...
}
```

[`▲ top`][top]

---

#### 7.3.7 Equality

Of the operators supported, the following can be used with non-numeric operands:

- `==`: can be used to check for equality of types and values in operands.
- `!=`: can be used to check for inequality of types and values in operands.

When these operators are used on immutable data, values are checked for equality
as well as types. When these operators are used on mutable data, types and refs
are checked for equality. Concept of "refs" is loosely defined as "any object"
that's instantiated through a class currently and then gets reassigned to other
names until a mutable method is called on it, which then turns into a new "ref".

This behavior is the same as JavaScript with the addition of structural equality
for immutable data (on top of nominal typing).

[`▲ top`][top]

---

### 7.4 Kitchen Sink

This is an example with almost every feature of the Wing, showing you a whole
picture of what the syntax feels like.

```TS
bring cloud;
bring fs;

struct DenyListRule {
  package_name: str;
  version: str?;
  reason: str;
}

struct DenyListProps {
  rules: mut_list<DenyListRule>;
}

resource DenyList {
  _bucket: cloud.Bucket;
  _object_key: str;

  init(props: DenyListProps) {
    this._bucket = cloud.Bucket();
    this._object_key = "deny-list.json";

    let rules_dir = this._write_to_file(props.rules, this._object_key);
    this._bucket.upload("${rules_dir}/*/**", prune: true, retain_on_delete: true);
  }

  _write_to_file(list: mut_list<DenyListRule>, filename: str): str {
    let tmpdir = fs.mkdtemp();
    let filepath = "${tmpdir}/${filename}";
    let map = mut_map<DenyListRule>();
    for rule in list {
      append_rule(map, rule);
    }
    fs.write_json(filepath, map);
    return tmpdir;
  }

  ~ rules: mut_map<DenyListRule>?;

  ~ init() {
    // this._bucket is already initialized by the capture mechanic!
    this.rules = this._bucket.get(this._object_key) ?? mut_map<DenyListRule>();
  }

  ~ lookup(name: str, version: str): DenyListRule? {
    return this.rules[name] ?? this.rules["${name}/v${version}"];
  }

  ~ add_rule(rule: DenyListRule) {
    append_rule(this.rules, rule)
    this._bucket.set(this._object_key, this.rules);
  }
}

let maybe_suffix = () -> {
  if version != nil {
    return "/v${version}";
  } else {
    return "";
  }
}

let append_rule = (map: mut_map<DenyListRule>, rule: DenyListRule) -> {
  let suffix = maybe_suffix();
  let path = "${rule.package_name}${suffix}";
  map[path] = rule;
}

let deny_list = DenyList();
let filter_fn = (event: cloud.QueueEvent) ~> {
  let package_name = event.data["package_name"];
  let version = event.data["version"];
  let reason = event.data["reason"];
  if deny_list.lookup(package_name, version) {
    print("Package rejected: ${package_name}");
  } else {
    print("Package accepted: ${package_name}");
  }
}

queue = cloud.Queue();
filter = cloud.Function(filter_fn);
queue.add_consumer(filter);
```

[`▲ top`][top]

---

### 7.5 Credits

- <https://github.com/WheretIB/nullc>
- <https://github.com/chaos-lang/chaos>
- <https://github.com/BlazifyOrg/blazex>
- <https://github.com/YorickPeterse/inko>
- <https://github.com/thesephist/ink>
- <https://github.com/vlang/v>

[top]: #wing-language-reference
[rfc]: https://github.com/monadahq/rfcs/blob/main/0044-winglang-requirements.md
