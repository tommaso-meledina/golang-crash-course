# Notes about my Golang crash course

This repository hosts the notes and snippets I wrote while familiarizing with Golang, coming from a Java background.

## Index

- [Abstract](#abstract)
- [Project Structure](#project-structure)
- [Features](#features)
- [Coding patterns](#coding-patterns)

## Abstract

Compared to Java, Go feels more "essential". Its coding patterns are more direct; simpler, in a way. It deliberately lacks the complexity that allows a Java developer to represent a rich domain model through hierarchically-organized classes, exchanging it for coding patterns that aim to make every snippet make sense, often even if read in isolation. Its coding style feels a bit like that of a scripting language - which we may argue that Go is.

While some of the most acclaimed features in Java development come from external libraries, and the presence of a myriad of alternative third-party options and tools is considered among the strengths of the Java ecosystem, Go is tightly self-complete and self-consistent: installing the Go runtime also brings the full set of tools and libraries that developers need in order to ship features, including: testing tools, performance testing tools, formatting/styling tools, HTTP libraries, filesystem libraries and many more, most notably the libraries supporting concurrency.

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

`internal/` directory: code in internal/ can only be imported by code in the parent tree; it enforces true privacy beyond package-level visibility.

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