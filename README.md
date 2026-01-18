# Notes about my Golang crash course

This repository hosts the notes and snippets I wrote while familiarizing with Golang, coming from a Java background.

## Index

- [Abstract](#abstract)
- [Project Structure](#project-structure)
- [Features](#features)
- [Coding patterns](#coding-patterns)
- [Concurrency](#concurrency)
- [Style quirks](#style-quirks)

## Abstract

Compared to Java, Go feels more "essential". Its coding patterns are more direct; simpler, in a way. It deliberately lacks the complexity that allows a Java developer to represent a rich domain model through hierarchically-organized classes, exchanging it for coding patterns that aim to make every snippet make sense, often even if read in isolation. Its coding style feels a bit like that of a scripting language - which we may argue that Go is, at heart.

While some of the most acclaimed features in Java development come from external libraries, and the presence of a myriad of alternative third-party options and tools is considered among the strengths of the Java ecosystem, Go is tightly self-complete and self-consistent: installing the Go runtime also brings the full set of tools and libraries that developers need in order to ship features, including: testing tools, performance testing tools, formatting/styling tools, HTTP libraries, filesystem libraries and many more, most notably the top-notch libraries supporting concurrency. A select few external libraries are commonly used, but the amount of external dependencies involved is orders of magnitude less than what I'm used to in Java.

## Project structure

### Modules and packages

A Go application is typically represented by a **module** and organized over **packages**.

**Packages** are Go's basic unit of code organization; every Go file belongs to a package. Practically speaking, a package is a directory containing `.go` files with the same `package` declaration.

A **module** is a collection of packages, potentially complemented by dependencies. It's defined by a `go.mod` file.

**Example structure:**

```
myproject/
  ├── go.mod                  (contains module name, go verison and optionally dependencies)
  ├── main.go                 (package main)
  └── utils/
      ├── date_utils.go       (package utils)
      └── string_utils.go     (package utils)
```

Test files are marked by the `_test` suffix in their name, and are typically placed alongside the files they refer to.

**Example structure with tests:**

```
myproject/
  ├── go.mod
  ├── main.go
  ├── main_test.go
  └── utils/
      ├── date_utils.go
      ├── date_utils_test.go
      ├── string_utils.go
      └── string_utils_test.go
```

### Other conventions

In addition to the simplified project structure presented in the past section, Go also has a handful of additional conventions.

`internal/` directory: code in `internal/` can only be imported by code in the parent tree; it enforces true privacy beyond package-level visibility.

```text
myapp/
  ├── go.mod
  ├── main.go                    Can import internal/
  ├── internal/
  │   └── auth/
  │       └── token.go
  └── handlers/
      └── user.go                Can import internal/

otherproject/
  └── main.go                    Cannot import myapp/internal/
```

`cmd/` directory: for multiple binaries

```text
myproject/
  ├── cmd/
  │   ├── server/
  │   │   └── main.go           (builds "server" binary)
  │   └── worker/
  │       └── main.go           (builds "worker" binary)
```

`vendor/` directory: stores copies of dependencies (created by `go mod vendor`); Go tools recognize it automatically.

`testdata`: for text fixtures; ignored by go build, used for test data files.

### Capitalization and visibility

Seemingly just another style convention, capitalization in the names of objects actually drives visibility within the module: lowercase elements are package-private, uppercase elements are public.

```go
package mathutils

// Public function - accessible from other packages
func Add(a, b int) int {
	return a + b
}

// private function - only accessible within mathutils package
func multiply(a, b int) int {
	return a * b
}

// Public struct with mixed visibility fields
type Calculator struct {
	Name    string // Public field
	version int    // private field
}
```

## Features

As I mentioned in the [abstract](#abstract), the Go runtime comes with a full set of standard tools that allow a complete developmente experience right off the bat, with little need of external dependencies.

### Package management

First of all, the `go` CLI completely covers all needs in terms of package management.

In fact, the usual `build`, `install`, `test`, `run` commands, plus the commands for listing and fetching external modules (for which I'm used to rely on a third-party pakcage manager such as Maven or Gradle) are standard features of the `go` CLI:

- `go build` - Compiles packages and dependencies into executable binaries
- `go clean` - Removes build artifacts and cached files
- `go doc` - Shows documentation for packages, functions, and symbols
- `go get` - Downloads and installs packages and dependencies
- `go install` - Compiles and installs packages/binaries to `$GOPATH/bin`
- `go list` - Lists packages and their dependencies
- `go mod` - Manages module dependencies (init, tidy, download, vendor, etc.)
- `go run` - Compiles and runs Go programs in one step (useful for development)
- `go test` - Runs tests and benchmarks for packages

### Additional testing tools

But the Go CLI goes further than the usual `go test` I would have expected; through the `go tool` command, it brings built-in coverage analysis, profiling and performance analysis:

- `go tool pprof` - Analyzes CPU and memory profiling data for performance optimization
- `go tool cover` - Analyzes and visualizes code coverage from tests
- `go tool trace` - Visualizes execution traces for concurrency and performance

### Code quality tools

Finally, the Go CLI features built-in code quality tools; most notably a standard, opinionated formatter (no more style wars!):

- `go fmt` - Automatically formats Go code to the official style standard
- `go vet` - Examines code for common mistakes and suspicious constructs
- `go fix` - Updates code to use new APIs after Go version upgrades

## Coding patterns

Let's get to the fun part: how do we actually code in Go?

### Fundamentals

The main syntax elements I encountered are variables (`var`), functions (`func`), types (`type`), structures (`struct`) and interfaces (`interface`).

**Variables** and **functions** are quite self-explanatory (although there's a couple of syntax tricks that are worth specifying later in this section).

A **type** identifies a named type entity, defined from either an underlying type, a structure (see below) or an interface (also see below), and optionally equipped with a set of methods. There are actually some additional constructs that a type can be defined from, but we'll ignore them for now.

**Structures** are primarily meant as data transfer objects; behavior (i.e. methods) can be attached to them, but it's not part of the definition.

**Interfaces** on the other hand _are_ behavioral contracts i.e. they define the signatures of the methods that a type needs to implement in order to qualify as an implementation of the interface.

```go
package main

import (
	"fmt"
	"strconv"
)

// type based on a primitive type
type nickname string

// type based on a structure
// the structure defines what kind of data
// will be carried around
type User struct {
	ID   int
	Name string
	Nick nickname
}

// Describer is a behavioral contract
// the interface 
type Describer interface {
	Describe() string
}

// Method attaches behavior to the User type.
func (u User) Describe() string {
	return u.Name + " (" + string(u.Nick) + " for friends)"
}

// Function depends on behavior, not concrete data.
func printDescription(d Describer) {
	fmt.Println(d.Describe())
}

func main() {
	var u = User{ID: 1, Name: "John", Nick: "Johnny"}
	printDescription(u)
}
```

Notice the syntax of function `func (u User) Describe() string`, it means:

- this function can be referenced outside its package (it's named `Describe` with a capital `D`)
- this function doesn't expect any input, since the input bit is empty `Describe()`
- this function returns a value of type `string` - keep in mind that functions in Go may return more than one value, e.g. we could have `Describe() (string, int)`
- this function is a method os the `User` type (the `(u User)` bit is called "receiver")

The above snippet, other than demonstrating the fundamental concepts I listed, is actually a working Go script: running the `go run` command on it will output `John(Johnny for friends)`.

Notice how the package is named `main`, and there's also a `main` function at the end of the file. "main" is a special name, that marks the main package of the application; within the `main` package, Go will expect one of the files to contain a `main` function: that's the execution entry point.

### Pointers

#### Refresher on pointers

Go makes aliasing and mutation explicit using pointers; as a consequence, I had to refresh my understanding of them. Consider the following snippet:

```go
package main

import "fmt"

func changeByValue(v int) {
	v += 1
}

func changeByPointer(v *int) {
	*v += 1
}

func main() {
	var x int = 1
	fmt.Println("-----\nDEMO STARTS")
	fmt.Println("x is", x)
	var p *int = &x
	fmt.Println("p is", p, "*p is", *p)

	fmt.Println("-----")
	x += 1
	fmt.Println("value of x has been increased in the main function:\nx is now", x, "\np is now", p, "\n*p is now", *p)

	fmt.Println("-----")
	changeByValue(x)
	fmt.Println("changeByValue func invoked:\nx is now", x, "\np is now", p, "\n*p is now", *p)

	fmt.Println("-----")
	changeByPointer(p)
	fmt.Println("changeByPointer func invoked:\nx is now", x, "\np is now", p, "\n*p is now", *p)
}
```

Let's go over the syntax first: while writing `var x int = 1` obviously tells the compiler "I am creating a variable of type `int` having `1` as value", writing `var p *int = &x` means "I am creating _a pointer to the value of that variable, of type int_ (i.e. `*int`); the value of the pointer is `&x` (i.e. _the pointer_ to `x`, or more precisely _the address_ of `x`)". Furthermore, if `p` is a pointer to `x`, then the value of `x` can be accessed from `p` through the syntax `*p`; the syntax `*x` on the other hand makes no sense (`x` is not a pointer).

In other words:

- `*int` represents the type of a pointer that points to the address of a value having type `int`; if we defined a custom type e.g. `type nickname string`, then a pointer to the address of a variable of such a type would have type `*nickname`.
- `&x` represents the address of `x`; if `x` is of type `mytype`, then `&x` is of type `*mytype`.
- `*p` represents the value that `p` points to.


Let's skip to how pointers behave. Running the above script returns the following output:

```text
-----
DEMO STARTS
x is 1
p is 0x1400010e040 *p is 1
-----
value of x has been increased in the main function:
x is now 2 
p is now 0x1400010e040 
*p is now 2
-----
changeByValue func invoked:
x is now 2 
p is now 0x1400010e040 
*p is now 2
-----
changeByPointer func invoked:
x is now 3 
p is now 0x1400010e040 
*p is now 3
```

First, we store the _value_ of `x` in the variable `x`, and a _pointer_ to `x` in the variable `p`.
Before any manipulation, the value of `x` is `1`; the value of `p` is _an address_; the value of `x` obtained through `*p` is also `1`.

Then we increase `x` by `1`, acting directly on `x` itself (`x+=1`). The value of `x` changed, hence printing `x` now yields `2`; printing `*p` (the value of `x` through its pointer `p`) also yields `2`; the address to `x` is of course unchanged.

Then we pass `x` to a function that receives it and also increases it by `1`, and we print everything again; we might expect the value of `x` to have changed, but that is not the case. That's because the function `changeValue` works on a mere copy of the value of `x`, and whatever modifications it makes to it are not reflected to the "original x". As a consequence, `x` is still `2`; `*p` is also `2`; the address stays unchanged.

Finally, we pass _a pointer to `x`_ to another function, that modifies its value _through the pointer_ (i.e. `*v += 1`); this time, the change _is_ reflected to `x` (the function can reach back to it, through the pointer).

#### Usage of pointers

Why are pointers back, and how are they used in Go?

As far as I understand, the purpose is to force developers to consciously decide whether the functions they design are supposed to perform mutations or not.

In fact, one instance in which I encountered the usage of pointers is in receivers (see above), i.e. when equipping types with methods.

Let's consider the first script we used in this article; in there, we wrote the following:

```go
func (u User) Describe() string {
	return u.Name + " (" + string(u.Nick) + " for friends)"
}
```

Which equips the type `User` with a `Describe` function, which has access to the value of the instance. We may have used a different syntax:

```go
func (u *User) Describe() string {
	return u.Name + " (" + string(u.Nick) + " for friends)"
}
```

In this case, the `Describe` function has access to a _pointer_ (the address of the value of the instance)! for this particular implementation it doesn't make much of a difference, but if the implementation included mutations then it would make _a lot_ of difference: using a value receiver (i.e. `(User u)`) would not propagate mutations, using a pointer receiver would!

> **In general, use pointers for methods that are supposed to apply mutations, don't use them otherwise.**

### Interfaces

As stated above, interfaces in Go define the signatures of the methods that a type needs to implement in order to qualify as an implementation of the interface.

For example, types that wish to qualify as implementing the `Describer` interface we used above

```go
type Describer interface {
	Describe() string
}
```

need to implement a method named `Describe`, that takes no input arguments and returns a `string`.

In Go, functions tyipically accept interfaces and return structures. In fact, one of the main roles of interfaces in Go is decoupling functions from specific implementations.

Consider the following snippet:

```go
package main

import "fmt"

type greeter interface {
	greet(name string) string
}

type englishGreeter struct{}

func (g englishGreeter) greet(name string) string {
	return "Hello, " + name
}

type italianGreeter struct{}

func (g italianGreeter) greet(name string) string {
	return "Ciao, " + name
}

func printWelcome(g greeter, name string) {
	fmt.Println(g.greet(name))
}

func main() {
	var dummyName string = "John"
	//var g = englishGreeter{}
	var g = italianGreeter{}

	printWelcome(g, dummyName)
}
```

- The `greeter` interface describes the expected _behavior_: concrete types shall implement a `greet` method accepting a `string` and returning a `string`.
- The `printWelcome` function accepts any concrete type implementing the `greeter` interface, it "does not know" which implementation it will work with.
- The `englishGreeter` and `italianGreeter` functions both implement the `greeter` interface (by defining a suitable `greet` method).
- The `main` function can invoke the `printWelcome` function using either the `englishGreeter` or the `italianGreeter` implementation interchangeably.

There are two things to highlight here:

1. The `englishGreeter` and `italianGreeter` functions implement the `greeter` interface _**implicitly**_: there is no keyword or explicit reference indicating that they implement it; they implement it by adhering to its contract.
2. The `greeter` interface is actually useless here, and introduced for educational purposes; a more idiomatic form would be the following:

```go
package main

import "fmt"

type englishGreeter struct{}

func (g englishGreeter) greet(name string) string {
	return "Hello, " + name
}

type italianGreeter struct{}

func (g italianGreeter) greet(name string) string {
	return "Ciao, " + name
}

func printWelcome(g interface {
	greet(string) string
}, name string) {
	fmt.Println(g.greet(name))
}

func main() {
	var dummyName string = "John"
	//var g = englishGreeter{}
	var g = italianGreeter{}

	printWelcome(g, dummyName)
}
```

Here, the `greeter` interface has been removed; in its stead, the `printWelcome` function uses an inline, anonymous definition. This for underlines a key design difference between Go and Java: in Go, interfaces are discovered bottom-up, not designed top-down. In other words, in Go interfaces are owned by the consumer!

### Composition

In Go there is no inheritance. Instead, structures embed other structures; behavior is _composed_, rather than extended/overridden; in other words behavior is additive, not substitutive (no Java `@Override`).

Let's introduce composition in the snippet we just used in the [Interfaces](#interfaces) section:

```go
package demo

import "fmt"

type englishGreeter struct{}

func (g englishGreeter) greet(name string) string {
	return "Hello, " + name
}

type italianGreeter struct{}

func (g italianGreeter) greet(name string) string {
	return "Ciao, " + name
}

type prefixer struct {
	prefix string
}

func (p prefixer) apply(s string) string {
	return p.prefix + s
}

type composedGreeter struct {
	prefixer
}

func (g composedGreeter) greet(name string) string {
	return g.apply(name)
}

func printWelcome(g interface {
	greet(string) string
}, name string) {
	fmt.Println(g.greet(name))
}

func InterfacesDemo() {
	var dummyName string = "John"
	var greetingPrefixer prefixer = prefixer{prefix: "Salutations, "}
	//var g = englishGreeter{}
	//var g = italianGreeter{}
	var g = composedGreeter{greetingPrefixer}

	printWelcome(g, dummyName)
}
```

In this version:

- we defined a simple `prefixer` structure, holding a `prefix` string property
- we equipped it with an `apply` method (it applies the `prefix` property to an input string)
- we defined `composedGreeter`, a new candidate  implementation of the (implicit) `greeter` interface; notice how we defined it as composed with the `prefixer` structure
- we equipped the `composedGreeter` structure with the `greet` method expected by the `greeter` interface
- finally, in the `main` function, we executed the new `composedGreeter` rather than the "old" `englishGreeter` or `italianGreeter` options - notice how the `printWelcome` function doesn't care one bit, that's what we mean when we say that the purpose of interfaces is to decouple consumers from specific implementations

## Concurrency

## Style quirks