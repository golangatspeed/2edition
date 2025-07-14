# Chapter 3 - Structure of a Go program

In this section, we're going to walk through the structure of a Go program using a typical *Hello World* program.

The program itself only prints out a couple of strings - but it does more than you might first expect - run it now on the Go Playground to see for yourself.

**Example 5 - Hello World in Go**
```go
package main

import (
    "fmt"
    _ "github.com/golangatspeed/pkg/sideeffect"
)

func init() {
    fmt.Println("Hello World!")
}

func main() {
    fmt.Println("Hello World again!")
}

// Go Playground: https://go.dev/play/p/HbshOM2vDed
```

> There's probably a bit more syntax in the above example than you might expect in a simple Hello World program, but the extra pieces will be explained and should be useful.

## 3.1 Packages and importing

All Go code resides in a package. An application's package is always `main`.

The `import` keyword is used to include other packages, from either the Go standard library as is the case with `fmt` or from external packages.

It's common to see external packages following a semantic naming convention which represents their location on the web e.g. GitHub repository URL. With Go modules, these import paths are now standardized and resolve directly to the module's declared name in its `go.mod` file.

Rather than use the `import` keyword for each package we can use brackets around all packages, as we have done in the example.

The underscore which prefixes the imported external package is known as the *blank identifier*. This is how we tell the compiler that we don't need to refer to the package contents in our code but we do need to run the `init()` function (see Section 3.2).

We call this importing a package for its *side-effects*. Without the blank identifier, the compiler would complain about the unused import.

The blank identifier has other uses which we'll cover later.

### 3.1.1 Aliasing

Packages can be prefixed with an alias which is useful when we experience naming collisions between similarly named packages, or wish to work with the package contents using a more semantic name than the package itself would otherwise allow.

Aliasing should be used judiciously to add clarity, and not as in the mischievous example below!

**Example 6 - How not to use aliasing**
```go
package main

import (
    log "fmt"
    fmt "log"
)

func main() {
    fmt.Println("This looks like log.Println output?")
    log.Println("This looks like fmt.Println output?")
}

// Go Playground: https://go.dev/play/p/zuXJ0If5OFn
```

### 3.1.2 The 'dot' import prefix

There is also a *dot* prefix - a period - which can be used to promote the functionality of an imported package to the current package namespace, allowing the package prefix to be omitted when making calls to its contents.

This is rarely seen in practice as it mainly detracts from clarity. To illustrate, in the code example below, the `Println()` function could also be from the `fmt` package as well as the `log` package. Only by inspecting the imported packages are we able to determine which it is.

**Example 7 - The dot import prefix**
```go
package main

import (
    . "log"
)

func main() {
    Println("This looks like log.Println output?")
}

// Go Playground: https://go.dev/play/p/SEUbOnOb6wH
```

## 3.2 Main and Init functions

The `main()` function is considered to be the entry point of the application - the top of the call stack. There is only one per executable. It takes no arguments and expects no return values.

Every package including `main` can have an `init()` function. Every included package with an `init()` function will have that function executed *once* when the application starts.

`init()` functions execute before `main()` but after package level variables are evaluated. Their use should be constrained to the initial setup, for example reading environment variables or a config file.

If you ran the *Hello World* program on the Go Playground you will have noticed the additional line (below) was printed before any of our program output?

```
This is the side effect?
```

This is the output of the `init()` function in the package `github.com/golangatspeed/pkg/sideeffect`, executing before our `init()` function which itself executes before our `main()` function.

## 3.3 Developing a package

If you are developing a package for use by other applications and not an executable, you will not have a `main()` function and your package name will be something other than `main`.

Below is a very simple package example. It's the content of the package we imported from Github in the *Hello World* example.

**Example 8 - Simple package**
```go
package sideeffect

import "fmt"

func init() {
    fmt.Println("This is the side effect?")
}
```

A package may contain a combination of exported and unexported functionality as we discussed in Chapter 1, although this example package exports nothing.

## 3.4 Project structure patterns

When developing Go applications, you have flexibility in how to organize your project structure. There are two common patterns, each suited to different project complexities and requirements.

### 3.4.1 Simple project structure

For straightforward applications with a single executable, placing `main.go` at the root of your project is perfectly acceptable and often preferred.

**Example 9 - Simple project layout**
```
my-app/
├── main.go
├── go.mod
├── go.sum
├── config.json
└── README.md
```

This approach offers several advantages:
- **Simplicity**: Clear and minimal structure
- **go get compatibility**: Users can install your application directly with `go install`
- **Ideal for**: CLI tools, simple services, learning projects

**Example project with root main.go:**
```go
// main.go
package main

import (
    "fmt"
    "os"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Usage: my-app <command>")
        return
    }
    
    fmt.Printf("Executing command: %s\n", os.Args[1])
}

// Go Playground: https://go.dev/play/p/simpleProjectRoot
```

### 3.4.2 Complex project structure

For projects requiring multiple executables or more sophisticated organization, the `cmd/` folder pattern provides better scalability and code reuse.

**Example 10 - Complex project layout**
```
my-project/
├── cmd/
│   ├── api/
│   │   └── main.go
│   ├── cli/
│   │   └── main.go
│   └── worker/
│       └── main.go
├── internal/
│   ├── config/
│   │   └── config.go
│   └── database/
│       └── db.go
├── pkg/
│   ├── logger/
│   │   └── logger.go
│   └── utils/
│       └── helpers.go
├── go.mod
├── go.sum
└── README.md
```

This structure enables:
- **Multiple executables**: Each `cmd/` subdirectory produces a separate binary
- **Code sharing**: Common packages in `pkg/` and `internal/`
- **Clear boundaries**: Public (`pkg/`) vs private (`internal/`) code separation

**Example with shared functionality:**
```go
// cmd/api/main.go
package main

import (
    "my-project/internal/config"
    "my-project/pkg/logger"
)

func main() {
    cfg := config.Load()
    log := logger.New(cfg.LogLevel)
    log.Info("API server starting...")
}

// cmd/cli/main.go  
package main

import (
    "my-project/internal/config"
    "my-project/pkg/logger"
)

func main() {
    cfg := config.Load()
    log := logger.New(cfg.LogLevel)
    log.Info("CLI tool starting...")
}

// Go Playground: https://go.dev/play/p/complexProjectStructure
```

### 3.4.3 Choosing the right pattern

**Use simple structure when:**
- Building a single-purpose tool or service
- Learning Go or prototyping
- The project is unlikely to need multiple executables
- You want `go install` to work directly on your module

**Use complex structure when:**
- You need multiple related executables (API + CLI + worker)
- You have substantial shared code between components
- You want clear public/private code boundaries
- The project will likely grow in complexity

Both patterns are valid and widely used in the Go community. The choice depends on your specific requirements and how you anticipate your project evolving. You can always start simple and refactor to the complex structure as your needs grow.

### 3.4.3 Advanced folder patterns

For more sophisticated projects, two additional folder patterns provide enhanced organization and access control.

#### The internal/ folder

The `internal/` folder receives special treatment from the Go tool, which uses it to limit access to packages contained within it. It effectively makes code invisible to another package unless it shares a common ancestor.

Most often this is used to prevent external modules from accessing code which a developer does not want to share or maintain a public API for. Similar to using the unexported visibility modifier in the code itself, we should incorporate `internal/` into our project organization to hide entire packages which are for *internal* use only.

**Example 11 - Understanding internal/ access control**
```
// From within my-project module - this works
my-project/
├── cmd/api/main.go         → can import "my-project/internal/config" 
├── internal/config/        → shared ancestor allows access
└── pkg/utils/helpers.go    → can import "my-project/internal/config"

// From external module - this fails  
other-project/
└── main.go                 → cannot import "my-project/internal/config"
                              Build error: use of internal package not allowed
```

#### The pkg/ folder

So we've covered the benefits of `cmd/` and `internal/`, what might we gain from using a `pkg/` folder in our project?

The folder name has no special significance in Go, but this name and structure provide two benefits.

First, the name `pkg` is semantic and tells other developers what packages in our module are intended to be reusable outside of the module. Of course, this would be the case for any package not in the `internal/` folder, which brings us nicely to the second benefit.

Most projects will have numerous other folders which reside at the root level inside the project folder. Folders for config, for build automation, maybe data folders or folders for assets such as images and scripts, to name just a few examples.

The point is that typically a Go project will have many other folders which will be nothing to do with your Go code. If we structure all Go packages at the same level as these folders it becomes difficult to determine what is a Go package and what is not. Developers coming to your project for the first time will need to establish this by interrogating each folder.

Instead, locating all the Go packages in the `pkg/` folder solves this problem very simply. We can see where all the public Go packages can be found without needing to inspect all the folders at the root level.

**Example 12 - Complete project structure**
```
my-project/
├── cmd/
│   ├── api/
│   │   └── main.go
│   └── cli/
│       └── main.go
├── internal/
│   └── hidden-package/
│       └── secrets.go
├── pkg/
│   ├── database/
│   │   └── db.go
│   └── logger/
│       └── logger.go
├── configs/
├── scripts/
├── assets/
├── go.mod
├── go.sum
└── README.md
```

This comprehensive structure demonstrates how `cmd/`, `internal/`, and `pkg/` work together to create clear boundaries and organization.

### 3.4.4 Choosing your approach

None of this is set in stone - you may wish to organize your code differently, and not use the `internal/` folder. However, if you adopt the practices outlined here, other developers will be able to identify where your Go code is located, if it contains executables, as well as understand which packages you intend for others to use, and which you do not.

The key principle is clarity - other developers should be able to quickly understand your project structure and locate relevant code. Whether you choose simplicity or organization, consistency throughout your project is what matters most.