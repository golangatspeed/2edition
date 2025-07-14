# Chapter 5 - Dependency management

In this chapter, we're going to take a deep dive into dependency management in Go, which changed radically in Go 1.13 with the default adoption of modules and was refined further with the introduction of workspaces in Go 1.18. Recent Go versions through 1.25 have continued to enhance these capabilities with improved security, performance, and developer experience features.

After reading this section you'll be comfortable creating and working with package dependencies in Go as well as managing modules and workspaces with the latest tools and best practices.

## 5.1 Modules

Go modules were introduced as an experimental feature in Go version 1.11, and later became the default approach to dependency management in version 1.13.

Before the introduction of modules, as we've stated, package management was largely dependent on the `go get` command, and Go source code was managed in the `$GOPATH/src` folder.

Modules allow developers to organise their Go projects locally as they please. Projects no longer need to be located within the system `GOPATH` directory structure - an approach that was often inconvenient.

So what is a module?

A module comprises one or more Go packages managed by a `go.mod` file. This file defines the module's identifier, the minimum version of Go required, and any external modules - together with their versions - on which the module depends.

The module's identifier is also the import path for the module. Remote modules need to be resolvable by that path identifier.

Where external dependencies are a part of a project, an additional `go.sum` file sits alongside `go.mod`. This file contains hashes - or checksums - of dependencies and is used to fix versions and determine where dependencies have been modified. Unlike the `go.mod` file, `go.sum` should never be manually edited.

Below is an example of a very basic `go.mod` file for a module which has no dependencies outside of the standard library.

```
module github.com/golangatspeed/pkg/sideeffect

go 1.25
```

Modules simplify and standardise the approach to dependency management and vendoring for everyone. Prior to module support, several third-party tools existed to solve the problem of dependency management.

Module management is a part of the standard Go tooling, and changes to `go.mod` can be initiated from the command line. The file can also be manually edited and this is often just as convenient.

To see all the available *go mod* commands run the below in your terminal.

```bash
go mod help
```

### 5.1.1 Direct versus indirect dependencies

A dependency on a module can be either *direct* or *indirect*. A direct dependency is simply a dependency that the module needs itself.

An indirect dependency is usually a dependency requirement of one of the direct dependencies or a dependency listed in `go.mod` which is not currently used in any of the module source files.

Indirect dependencies in `go.mod` are suffixed with `// indirect`.

Below is a more complete example of a `go.mod` file containing both direct and indirect external dependencies.

```
module github.com/my-module

go 1.25

require (
    github.com/spoonboy-io/koan v0.1.0
    github.com/TwiN/go-color v1.1.0 // indirect
    golang.org/x/sys v0.0.0-20210630005230-0f9fa26af87c // indirect
)
```

### 5.1.2 Creating and updating a module

Module creation is straightforward via the `go mod init` command.

```bash
# create the module
go mod init *module-identifier*
```

Remote modules included in your code with the `import` keyword, and which are not already available on your system, can generally be downloaded and added to `go.mod` with the `go mod tidy` command.

```bash
# add dependencies and update go.mod
go mod tidy
```

Alternatively, use the traditional `go get` command from within your project workspace to fetch the module. This command is module aware so `go.mod` is automatically updated to include the new dependency.

```bash
go get *module-identifier*
```

By default, both `go mod tidy` and `go get` will fetch the latest version of the module, equivalent to this command with the `@latest` suffix.

```bash
# we don't need to explicitly add @latest, that is the default
go get *module-identifier*@latest
```

If you require a specific version of a module it is possible to obtain it by specifying the *tag*, *branch* or even *commit reference*.

Below are three examples which show how to download and use specific module versions by tag, branch and commit. In all cases `go.mod` is updated with the requested version and `go.sum` will be used to track changes to that version.

```bash
# get a specific tag (version)
go get *module-identifier*@v1.0.0

# get the development branch 
go get *module-identifier*@my-dev-branch

# get a specific commit reference @07434ea
go get *module-identifier*@07434ea
```

### 5.1.3 Enhanced module commands

Go 1.20+ introduced several enhanced module management commands that improve the development experience and provide better control over dependencies.

**Example 13 - Enhanced module management**
```bash
# Download modules without updating go.mod
go mod download

# Edit go.mod programmatically
go mod edit -require=github.com/example/pkg@v1.2.0

# Edit go.mod to add exclude directive
go mod edit -exclude=github.com/problematic/pkg@v0.1.0

# Verify dependencies haven't been modified
go mod verify

# Clean module cache
go clean -modcache

# Show why a dependency is needed
go mod why github.com/example/pkg
```

These commands provide granular control over module management, particularly useful in automated environments and CI/CD pipelines.

### 5.1.4 The replace directive

The *replace* directive provides a simple mechanism to substitute a required module with another version of that module. This code can either be stored locally on your machine or at an alternative remote URL.

Imagine finding a bug in a third-party module, reporting it and then being reliant on the code owner to fix their code. Until it's fixed your project can't use that code or must accept the buggy code. Imagine another scenario, needing a new feature in a module, and asking the maintainer to add it. Until it is added, your project has to do without the feature.

In both above scenarios, the codebase can be forked and amended to fix the bug or add the feature. You can send a pull request to the respective maintainer(s) as a good citizen, but you then have to wait for it to be accepted before your project can use the updated code.

The *replace* directive presents a way around the above problem. We simply add a replace directive to `go.mod` which tells Go to use code in the forked repository in place of the original module.

When the original module is fixed or has the new feature added, the *replace* directive can simply be removed. There is no impact on your go codebase: no need to find and replace every import statement which contains that module.

Replacements can be made using the `go mod replace` command but it is very simple to include them in `go.mod` directly. The example `go.mod` file below shows two replacements being made.

The first *replace* directive replaces the module with a locally available version using the absolute path to the module's location on disk.

> It's worth stating the obvious here - that local replacements will break builds for other users who don't have the module available at that location so they should probably be removed before the code is shared.

The second *replace* directive substitutes in a forked repository and is safer to share. It achieves the same end, but other users will be able to build the project because the forked version is accessible on the Internet.

```
module github.com/golangatspeed/replace-example

replace github.com/original/module => /Users/olliephillips/module-my-version
replace github.com/original/module-two => github.com/golangatspeed/module-two

require (
    github.com/original/module v1.0.0
    github.com/original/module-two v1.0.0
)
```

### 5.1.5 Module security and authentication

Go 1.20+ enhanced module security features, providing better protection against supply chain attacks and unauthorized modifications.

**Example 14 - Module security configuration**
```bash
# Configure checksum database (enabled by default)
export GOSUMDB=sum.golang.org

# Disable checksum verification for private modules
export GOSUMDB=off

# Configure version control system access
export GOVCS=github.com:git,example.com:off

# Set private module patterns
export GOPRIVATE=*.corp.example.com,rsc.io/private
```

**Private module authentication:**
```bash
# Configure Git for private repositories
git config --global url."https://username:token@github.com/".insteadOf "https://github.com/"

# Or use SSH for private modules
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

The checksum database (`GOSUMDB`) verifies that modules haven't been tampered with, while `GOVCS` controls which version control systems can be used for different module paths.

## 5.2 Workspaces

The *workspace* feature was added in Go version 1.18 and furthered the flexibility of modules and dependency management generally.

*Workspaces* allow replacements of modules to be made with modules found locally, **without** making changes to the `go.mod` file.

This avoids the problem of having to add and subsequently remove the replace directives, when working with local module versions, before committing code to version control.

A workspace is defined by a single `go.work` file and is best used for local development, so excluded from version control.

To create a workspace in your project use the commands below.

```bash
# create a workspace file
go work init 

# or create the file and add a local path to the workspace
go work init *path/to/module/or/modules*
```

A `go.work` file created with the second of the two commands would have the following contents.

```
go 1.25

use *path/to/module/or/modules*
```

Modules which are located within that path will be used in preference to any online versions or locally cached versions.

We can also use the *replace* directive to replace specific modules and versions in the `go.work` file instead of `go.mod`.

### 5.2.1 Enhanced workspace management

Go 1.20+ introduced additional workspace commands that improve multi-module development workflows.

**Example 15 - Advanced workspace operations**
```bash
# Add module to existing workspace
go work use ./my-module

# Remove module from workspace
go work use -r ./old-module

# Edit workspace file
go work edit -use=./new-module

# Sync workspace dependencies with modules
go work sync

# View current workspace configuration
go work edit -print
```

**Example workspace for microservices development:**
```
go 1.25

use (
    ./services/api
    ./services/auth
    ./services/payment
    ./shared/config
    ./shared/logger
)

replace example.com/legacy-lib => ./vendor/legacy-lib
```

This workspace setup allows you to work on multiple related services simultaneously, with changes to shared packages immediately visible across all services.

In summary, workspaces help developers isolate changes that should only apply in their development environment, so mitigating the risk of committing changes which would detrimentally impact other developers and their environments.

## 5.3 Vendoring

Vendoring includes a local copy of external module dependencies inside the project itself in a `vendor` folder. Go then uses the vendored modules in builds and testing instead of their external equivalents.

Vendoring is useful in many situations. Where a project needs to be built on a machine with no internet access for example; or where strict dependency management has been adopted and submitting vendored modules to version control assists in that end.

Vendoring can also ensure that modules remain available to a project, even if the module is removed from the Internet. This risk is mitigated by the Go module proxy to an extent, but it can still happen - and a project which cannot locate a dependency cannot be built.

Finally, where builds are automated in some form of continuous integration process, build times can be reduced if modules are vendored locally and don't need to be downloaded from the Internet for each build.

Vendoring is very straightforward, simply run the below command from within your project.

```bash
go mod vendor
```

A `vendor` folder will be created with local copies of all external module dependencies. There is also a `modules.txt` file created which lists vendored modules and their versions and this is used as a manifest by the Go tool.

### 5.3.1 Modern vendoring practices

Recent Go versions have improved vendoring with better tooling and integration with modern development workflows.

**Example 16 - Advanced vendoring workflow**
```bash
# Create vendor directory with verification
go mod vendor

# Verify vendor directory matches go.mod
go mod verify

# Build using vendor directory (forced)
go build -mod=vendor

# Check vendor directory status
go list -m -f '{{.Dir}}' all | grep vendor

# Clean and recreate vendor directory
rm -rf vendor && go mod vendor
```

**Vendoring best practices:**
- **CI/CD integration**: Use vendoring in air-gapped environments
- **Security scanning**: Vendor directories enable offline security analysis
- **Reproducible builds**: Vendored dependencies ensure build consistency
- **Performance**: Reduces network dependency in automated builds

**Example vendoring in Docker:**
```dockerfile
# Dockerfile with vendoring
FROM golang:1.25-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
COPY vendor/ vendor/
COPY . .

# Build using vendored dependencies
RUN go build -mod=vendor -o app ./cmd/server

FROM alpine:latest
COPY --from=builder /app/app /usr/local/bin/
CMD ["app"]
```

This approach ensures that builds don't require internet access and are completely reproducible, making them ideal for secure or isolated environments.

## 5.4 Module proxy and performance

Go's module proxy system provides reliable, fast access to modules while offering security and availability guarantees.

**Example 17 - Module proxy configuration**
```bash
# Default proxy configuration
export GOPROXY=https://proxy.golang.org,direct

# Multiple proxy fallback
export GOPROXY=https://proxy.corp.com,https://proxy.golang.org,direct

# Corporate proxy with authentication
export GOPROXY=https://username:password@proxy.corp.com

# Disable proxy for specific patterns
export GONOPROXY=*.corp.example.com,github.com/company/*

# Check what proxy is being used
go env GOPROXY
```

The proxy system automatically caches modules, provides checksums for verification, and ensures availability even if the original repository becomes unavailable. This makes dependency management more reliable and builds faster.

Modern Go development benefits enormously from these dependency management improvements. The combination of modules, workspaces, enhanced security, and reliable proxying creates a robust foundation for building maintainable Go applications at any scale.