# Chapter 1 - Introduction to Go

In this chapter, we'll cover some essentials. First, we'll lay out a quick introduction to Go explaining what makes it attractive as a programming language, then we'll address some broader Go concepts and terminology that we will make use of as we progress.

## 1.1 Why Go?

Go, or Golang is a programming language developed by a team at Google. Similar to C (or Clang), it draws inspiration from many other programming languages also.

Go is a high-performance language which runs directly against physical machine resources and not on a virtual machine, unlike Java for example. Further, asynchronous programming via concurrency and goroutines can fully utilise the multiple cores of modern CPUs better than many other multi-threaded languages.

Go is a compiled language. Your program code is built into an OS-specific executable and not interpreted at runtime. It is statically typed, so types are known - and fixed - at compile time. Memory management in Go is for the most part automatic. Memory allocation is done for us as required, and memory is deallocated when no longer in use via a *garbage collection* process. Together these traits help to eliminate many common programming errors.

Go is opinionated on many things, one of which is formatting. This opinion, together with its simple syntax, results in very readable and clear code. Go has a complete suite of built-in tooling which includes utilities for testing, building, profiling and more. It is the readable code and powerful tooling which attract many developers to the language.

Go modules were introduced in version 1.11 for dependency management and have become the standard approach, making package management simple and reproducible.

Generics were introduced in version 1.18, providing type safety and code reusability while maintaining Go's commitment to simplicity. Go workspaces were also introduced in 1.18, making it easier to work with multiple related modules.

As of Go 1.25, the language continues to evolve thoughtfully, with recent additions including enhanced iteration patterns, structured logging, Profile-Guided Optimization for performance improvements, and an evolving standard library that maintains backwards compatibility while providing modern capabilities. The language has matured significantly while preserving the clarity and productivity that made it attractive initially.

## 1.2 Language semantics

Go is not *object-oriented* although it feels similar at times.

Possibly because of that similarity, or more likely familiarity with object-oriented programming (OOP) in general, developers often find themselves conversing about Go using *object-oriented* semantics which are often not appropriate to Go.

You may see and hear terms like *instance*, *object* and *method* - even *inheritance* - being used in the context of Go code, and while it sort of works, it is mostly incorrect.

Better we establish the right Go terminology early on and use it. Especially as the correct semantics can help us understand what Go is doing differently from OOP-based languages.

For example, while OOP has *inheritance*, Go has *composition* and *embedding*. There's an overlap between these approaches but they are not the same thing.

Go does not have *objects* or *instances*, neither does it have *methods* in the OOP sense. Instead, we have *receivers* (or *receiver functions*), so named because they *receive* the value (or the address of the value) of the type on which they are bound. While conceptually similar to *methods*, Go *receivers* can be bound to any user-defined type, they are not constrained to a *class* hierarchy.

That said, *receivers* and *methods* are terms that are used interchangeably in Go. I generally use the term *receivers* and will do it exclusively throughout *Go Faster*, purely to break the link with OOP and class-based property access.

Like OOP, Go has *interfaces* - a very powerful aspect of the language when used appropriately - and of course, we have *functions*.

In Go, everything is a value. Even *pointers* are just a value, that value being the *address* of another value in memory. There's no magic, but this concept is core to understanding how to work with values and pointers and avoid some common mistakes.

Errors are just values too, Go does not have exceptions. Errors must be checked and handled in your code, which despite being a little verbose and repetitive, contributes greatly to readability and comprehension.

So, Go has some specific terminology such as *types*, *receivers* and *pointers* and we should use these. For anything else, it is more correct to refer to it as a *value* of something and not an *instance* of something - or indeed an *object*.

There's quite a lot to unpack there, but don't worry we will cover all of the above in more detail as we progress.

## 1.3 Visibility

Unlike *object-oriented programming*, Go does not have the concept of *public* and *private*. Instead, it uses the notion of *exported* and *unexported* visibility modifiers.

Any *variable*, *constant*, *function*, *receiver*, *type* or indeed *struct field*, declared with a lowercase first letter in its name, is considered *unexported* and cannot be directly accessed outside of its package namespace.

The same, when capitalised, is considered to be *exported* and fully visible to the code that imports the package.

The example code below illustrates the principle when applied to types, variables, functions and struct fields. Note the exported struct field in the unexported struct type at line 9 which is a bit odd: an exported field on an unexported struct would appear to be completely redundant.

**Example 1 - Visibility modifiers**
```go
package exporting

type ExportedStruct struct {
	unexportedField string 
	ExportedField   string
}

type unexportedStruct struct {
	ExportedField string // how is this useful?
}

var ExportedVariable string
var unexportedVariable string

func ExportedFunc() { }

func unexportedFunc() { }
```

Well, it is redundant when it comes to exporting that field outside of the package because the struct itself is unexported, but you may often see code written like this, simply because there is a need to perform some type of conversion of that struct field into an alternative data format, such as JSON.

We call this conversion process *marshalling* and Go will only *marshal* exported struct fields when creating the output. Unexported struct fields are ignored.

Why does *unexported* not mean *private*? Because unexported values can still be made available outside of the package. For example, there is nothing which prevents an exported function or receiver from returning an unexported value from the package namespace to the caller. While code like this can be a source of confusion, so it is not recommended practice, it's perfectly possible to do this.

So when building packages for others to use, we should make decisions about *exporting* and *unexporting*, not on a need for concealment, but instead as a way of communicating which parts of our package should comprise the API that developers use, and which parts should not.

## 1.4 Comments and documentation

The single-line and multi-line comment styles of Go are probably already familiar and are used in many languages inspired by C. See the example below.

**Example 2 - Comment styles**
```go
package main

func main() {
	// This is a single-line comment

	/*
		This is a multi-line comment
		which spans more than one
		line.
	*/
}
```

For comments to be useful, they should be concise and precise and must be updated in line with the code. Stale comments which are not maintained with the codebase, detract from the code and may even confuse matters.

In Go, comments are especially useful since they are used to generate documentation. Developers write comments inline which then automatically form the basis of their software's instruction manual.

The documentation itself is built from the code comments using a command line tool called *Godoc*. We'll cover installing and using the tool shortly, but first, let's outline some *standards* to help you get the most from the *Godoc* tool.

### 1.4.1 Comment standards

1. All exported functionality of a package is expected to have a comment. The comment should start with the *function*, *variable* or *type* name for which it is written, and should succinctly explain its purpose. Only comments in this format on exported functionality will be used to build documentation.

2. The package itself should include a comment above the package name, which provides an overview of what the package does. This is used in the Overview section of the built documentation.

3. Links can be included in comments via an absolute URL which includes the schema e.g. HTTPS. Relative URLs are ignored. The *Godoc* tool will parse and include links which meet the requirements in the documentation.

4. Comments separated by a commented blank line will be interpreted as paragraphs and output as such in the package documentation.

The example below shows a package with these requirements satisfied. The code is unimportant for the moment.

**Example 3 - Making the most of comments in documentation**
```go
// Package user implements functionality for working with users.
//
// This content will be in a new paragraph.
package user

// User is a representation of a single user
type User struct {
	Name string
}

// NewUser is a factory that allows us to configure the properties
// of a new User.
//
// See https://someurl.com for more information.
func NewUser(name string) *User {
	return &User{
		Name: name,
	}
}
```

### 1.4.2 Installing and using Godoc

*Godoc* is an external package which can be downloaded and installed using the Go tool. Assuming you have installed Go locally, the following command will fetch the latest version of the package and install it in your workspace.

```bash
go install golang.org/x/tools/cmd/godoc@latest
```

Once installed, the tool can display documentation for the Go standard library - useful when offline - from any terminal window with this command.

```bash
godoc
```

The command starts a document server on http://localhost:6060. You can specify a different port if 6060 is in use.

```bash
godoc -http=:9090
```

For more information on using *Godoc* run the command with the help flag.

```bash
godoc -h
```

You can also preview how your code comments will appear as documentation using *Godoc*. For any package with *Go Modules* support, simply navigate to the folder which contains `go.mod` in a terminal window and run *Godoc*.

The documentation for the package will be created and listed under the "Third Party" section in the documentation server's index.

### 1.4.3 Including example code

We can include code examples as part of the generated documentation. These appear as text "code" and "output" sections in the documentation. However, if you run *Godoc* with the `-play` option, any such examples become interactive.

When interactive, not only can you run code snippets on the Go Playground using the additional *Play* button which appears top right, but all examples included in the documentation become executable - and editable - programs and not just static text. This is a really useful feature when learning about the standard library.

*Godoc* examples are actually a type of test, and they are run and verified like other tests. We cover testing later in the book, but just like other tests, examples must reside in files named with the `_test.go` suffix.

Within those files, examples are differentiated from normal tests as the function names use the `Example` and not the `Test` prefix as shown in the following example taken from the standard library *stringutil* package.

**Example 4 - An example function**
```go
package stringutil_test

import (
    "fmt"

    "golang.org/x/example/stringutil"
)

func ExampleReverse() {
    fmt.Println(stringutil.Reverse("hello"))
    // Output: olleh
}
```

Examples do not contain assertions normally associated with unit tests which determine pass or fail. Instead, they contain a commented `Output` line.

The Go test tool checks the actual output of an example function against the output which is on this line. If it matches it passes, if not it fails.

This is a useful and overlooked feature when you need to make assertions on functionality which outputs to Stdout. You can use an *Example* to check it, there's no need to pipe or redirect the output to a file or variable to compare it.

The online [Go package repository](https://pkg.go.dev) hosts documentation in interactive mode, so you can also experiment with example code there.