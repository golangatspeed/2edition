# Chapter 2 - The Go command line interface (CLI)

When you download Go you get the Go standard library packages and the Go CLI which is a tool for managing Go source code. You'll use it for much of what you do with Go, with one or two exceptions, for example, *Godoc* as we've seen.

Exploring all the functionality of the *Go* CLI is left as an exercise for the reader, this section aims to show you how to get started with the tool and highlight specific commands you'll use most often.

You can list all the top-level options of the Go tool using this command.

```bash
go help
```

If you want to get more help for a specific option use the same command followed by the option. For example, to learn more about the `version` option run the following command.

```bash
go help version
```

## 2.1 Version information

Go has a backwards compatibility guarantee which means that code written in Go version 1.0 can still be compiled and run on the latest Go 1.x version. This doesn't work the other way, so code written in Go 1.25 may not compile on earlier versions of Go, so you'll sometimes need to check the version you're using.

To get the current installed Go version run this.

```bash
go version
```

The output will be similar to below, showing the version, OS, and architecture.

```
go version go1.25 darwin/amd64
```

## 2.2 Environment information

You can inspect the configuration for your installed Go environment using this command.

```bash
go env
```

This outputs all the configuration variables for your installation and can be useful if you are having issues with the Go tool.

The variables worth highlighting here are:

- GOPATH - the location of your Go src code including the standard library
- GOMODCACHE - the location of modules which have been downloaded and cached
- GOPROXY - the proxy from which new packages are downloaded

For information on other environment variables [consult this page](https://pkg.go.dev/cmd/go#hdr-Environment_variables).

## 2.3 Module and workspace management

In a later chapter, we'll cover dependency management in Go and discuss the roles of modules and workspaces. For now, you can familiarise yourself with the module and workspace options via the `help` subcommand.

```bash
go mod help
go work help
```

## 2.4 Format your code

If you're using an IDE like Visual Studio Code with the Go plugin installed, or Goland, then it's likely your code is formatted automatically upon saving. If you don't have this facility the `fmt` subcommand will format your Go source files according to Go standards. Running the tool against a Go source file is straightforward.

```bash
go fmt main.go
```

## 2.5 Testing

We'll get to testing later in the book, for now, it's enough to know that testing is initiated from the Go tool also. For example to run all the tests in your project use this command from within the project folder.

```bash
go test ./...
```

## 2.6 Cleanup

*Clean* removes object files from package source directories. It's useful if you build an executable and it is placed in the project workspace along with your source. It can be removed, so that it is not committed to version control, with the following command.

```bash
go clean
```

## 2.7 Downloading packages

We can use the `get` command to download package dependencies and install them. We cover this in a later chapter but here's an example which will add a dependency for a package, or if it is already available, it will upgrade the package to the latest version.

```bash
go get example.com/pkg
```

## 2.8 Running a program

Assume you've written a program and all the code is saved in file `main.go`. To run the code navigate to the folder which contains `main.go` and execute this command.

```bash
go run main.go
```

If package `main` was made up of multiple Go source files, as is often done to help organise the source code, we could pass all the files to the `run` command like this.

```bash
go run *.go
```

> The wildcard filename may not work on Windows systems, and you may even run into problems on Mac/Linux when external files for configuration or data are used by your program, in that the paths are not resolvable. So, it's recommended to build the program and run the resulting executable if you encounter any problems.

## 2.9 Building your program

To build an executable for your current operating system, simply use this command.

```bash
go build -o helloWorld ./...
```

The `-o` flag is used to provide a specific name for the built executable.

By default, if no name is specified and the main package is at the root of your project folder the binary created will be named using the *module* name and the executable will be placed at the root of your project folder.

Later we'll talk about project organisation and how the `cmd` folder is helpful.

If you use the `cmd` folder in your project executables will be built using their sub-folder names unless names are specified with the `-o` flag.

If we take an example project which contains the source for two executables, *api* and *cli*, the below commands would be needed to build them.

```bash
go build ./cmd/api
go build ./cmd/cli
```

The built executable files are again placed at the root of your project folder. It can be run from the root folder like this.

```bash
./api
```

### 2.9.1 Profile-Guided Optimization builds

Go 1.21 introduced Profile-Guided Optimization (PGO) as a stable feature. This allows the compiler to optimize your builds using runtime profile data, typically providing 2-7% performance improvements with minimal effort.

To use PGO, first collect a CPU profile from your running application, then rebuild with the profile:

**Example 5 - Using PGO with go build**
```bash
# Build your application normally
go build -o server ./cmd/server

# Run and collect a profile (production workload recommended)
./server &
SERVER_PID=$!
curl http://localhost:8080/debug/pprof/profile?seconds=30 > default.pgo
kill $SERVER_PID

# Rebuild with PGO optimization
go build -pgo=default.pgo -o server-optimized ./cmd/server
```

The Go compiler automatically uses a `default.pgo` file if it exists in the main package directory, so you can also simply place your profile there and build normally:

```bash
# Compiler automatically detects default.pgo
go build -o server-auto ./cmd/server
```

PGO works best with CPU-intensive applications that have predictable execution patterns. For I/O-bound or highly variable workloads, the benefits may be minimal.

## 2.10 Building for other operating systems and architectures

As we've seen, it's simple to build a program for your operating system using `go build` but one of the best features of Go is that we can just as easily compile our code to run on other systems too.

It's very straightforward to cross-compile source code to alternative targets. We use the same *build* command as before but additionally specify both the *OS* and *Architecture* we want to build for using the `GOOS` and `GOARCH` environment variables.

By default, these variables are set to our local OS and architecture so we're overriding them to build for a different target.

You can inspect all of the target platforms your Go version can compile for using this command.

```bash
go tool dist list
```

In the console output, the text before the forward slash is the operating system, and the text afterwards is the architecture.

For example, on *macOS*, to build the *helloWorld* program as we did in section 2.9 but for Linux OS and a 64-bit AMD architecture we'd do this.

```bash
env GOOS=linux GOARCH=amd64 go build -o helloWorld ./...
```

To compile for Windows and a 64-bit ARM architecture we'd use this command instead.

```bash
env GOOS=windows GOARCH=arm64 go build -o helloWorld ./...
```

### 2.10.1 Enhanced ARM64 support

Go 1.20+ significantly improved ARM64 support across platforms. The `GOARM64` environment variable provides more granular control over ARM64 target features:

**Example 6 - Advanced ARM64 cross-compilation**
```bash
# Build for ARM64 with specific version features
env GOOS=linux GOARCH=arm64 GOARM64=v8.1 go build -o app-arm64 ./...

# Build for macOS ARM64 (Apple Silicon)
env GOOS=darwin GOARCH=arm64 go build -o app-macos-arm ./...

# Build for Windows ARM64 with performance optimizations
env GOOS=windows GOARCH=arm64 GOARM64=v8.2 go build -o app-windows-arm.exe ./...
```

The `GOARM64` values include:
- `v8.0` - ARMv8.0 (default, maximum compatibility)
- `v8.1` - ARMv8.1 with additional instruction set features
- `v8.2` - ARMv8.2 with further performance enhancements

This enhanced ARM support reflects the growing importance of ARM processors in both server and client environments, particularly with Apple Silicon and AWS Graviton processors.

## 2.11 Tool directives and build configuration

Go 1.20+ introduced enhanced tool directives that provide more control over the build process. These directives appear as specially formatted comments in your Go source files.

**Example 7 - Modern tool directives**
```go
package main

//go:build linux && (amd64 || arm64)

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Printf("Running on %s/%s\n", runtime.GOOS, runtime.GOARCH)
}

// Go Playground: https://go.dev/play/p/buildDirectives
```

Common build constraints have been enhanced for better performance and maintainability:

```bash
# Build only files with specific build tags
go build -tags "production,logging" ./...

# Use build constraints for environment-specific code
go build -tags "development" ./...
```

**Example 8 - Environment-specific builds**
```go
//go:build production
// +build production

package config

const (
    LogLevel = "error"
    Debug    = false
    APIUrl   = "https://api.production.com"
)
```

```go
//go:build development
// +build development

package config

const (
    LogLevel = "debug"
    Debug    = true
    APIUrl   = "http://localhost:8080"
)
```

These improvements make it easier to maintain different build configurations for development, testing, and production environments while keeping the build process transparent and predictable.

The newer `//go:build` syntax is preferred over the legacy `// +build` format, though both are supported for backward compatibility. The modern syntax supports more intuitive boolean expressions and better tooling integration.