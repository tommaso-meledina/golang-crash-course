# Notes about my Golang crash course

This repository hosts the notes and snippets I wrote while familiarizing with Golang, coming from a Java background.

## Index

- [Abstract](#abstract)
- [Project Structure](#project-structure)
- [Features](#features)
- [Coding patterns](#coding-patterns)

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

**Structures** are primarily meant as data carriers; behavior (i.e. methods) can be attached to them, but it's not part of the definition.

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
	u := User{ID: 1, Name: "John", Nick: "Johnny"}
	printDescription(u)
}
```

`TODO`:

- fundamental bits: functions, variables, structs (only a mention to channels and mutex - or maybe not even that)
- `main` function in the `main` package
- pointers are back
- interfaces: definition VS implementation
- implicit implementation of interfaces
- composition over inheritance