# Chapter 5 - Dependency management

In this chapter, we're going to take a deep dive into dependency management in Go, which changed radically in Go 1.13 with the default adoption of modules and was refined further with the introduction of workspaces in Go 1.18.

After reading this section you'll be comfortable creating and working with package dependencies in Go as well as managing modules and workspaces.

## 5.1 Modules

Go modules were introduced as an experimental feature in Go version 1.11, and later became the default approach to dependency management in version 1.13.

Before the introduction of modules, as we've stated, package management was largely dependent on the `go get` command, and Go source code was managed in the `$GOPATH/src` folder.

Modules allow developers to organise their Go projects locally as they please. Projects no longer need to be located within the system `GOPATH` directory structure - an approach that was often inconvenient.

So what is a module?

A module comprises one or more Go packages managed by a `go.mod` file. This file defines the module's identifier, the minimum version of Go required, and any external modules - together with their versions - on which the module depends.

The module's identifier is also the import path for the module. Remote modules need to be resolvable by that path identifier.

Where external dependencies are a part of a project, an additional `go.sum` file sits alongside `go.mod`. This file contains hashes - or checksums - of dependencies and is used to fix versions and determine where dependencies have been modified. Unlike the `go.mod` file, `go.sum` should never be manually edited.

Below is an example of a very basic `go.mod` file for a module which has no dependencies outside of the standard library.

```go
module github.com/golangatspeed/pkg/sideeffect

go 1.13
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

```go
module github.com/my-module

go 1.17

require (
	github.com/spoonboy-io/koan v0.1.0
	github.com/TwiN/go-color v1.1.0 // indirect
	golang.org/x/sys v0.0.0-20210630005230-0f9fa26af87c // indirect
)
```

### 5.1.2 Creating and updating a module

Module creation is straightforward via the `go mod init` command.

```bash
// create the module
go mod init *module-identifier*
```

Remote modules included in your code with the `import` keyword, and which are not already available on your system, can generally be downloaded and added to `go.mod` with the `go mod tidy` command.

```bash
// add dependencies and update go.mod
go mod tidy
```

Alternatively, use the traditional `go get` command from within your project workspace to fetch the module. This command is module aware so `go.mod` is automatically updated to include the new dependency.

```bash
go get *module-identifier*
```

By default, both `go mod tidy` and `go get` will fetch the latest version of the module, equivalent to this command with the `@latest` suffix.

```bash
// we don't need to explicitly add @latest, that is the default
go get *module-identifier*@latest
```

If you require a specific version of a module it is possible to obtain it by specifying the *tag*, *branch* or even *commit reference*.

Below are three examples which show how to download and use specific module versions by tag, branch and commit. In all cases `go.mod` is updated with the requested version and `go.sum` will be used to track changes to that version.

```bash
// get a specific tag (version)
go get *module-identifier*@v1.0.0

// get the development branch 
go get *module-identifier*@my-dev-branch

// get a specific commit reference @07434ea
go get *module-identifier*@07434ea
```

### 5.1.3 The replace directive

The *replace* directive provides a simple mechanism to substitute a required module with another version of that module. This code can either be stored locally on your machine or at an alternative remote URL.

Imagine finding a bug in a third-party module, reporting it and then being reliant on the code owner to fix their code. Until it's fixed your project can't use that code or must accept the buggy code. Imagine another scenario, needing a new feature in a module, and asking the maintainer to add it. Until it is added, your project has to do without the feature.

In both above scenarios, the codebase can be forked and amended to fix the bug or add the feature. You can send a pull request to the respective maintainer(s) as a good citizen, but you then have to wait for it to be accepted before your project can use the updated code.

The *replace* directive presents a way around the above problem. We simply add a replace directive to `go.mod` which tells Go to use code in the forked repository in place of the original module.

When the original module is fixed or has the new feature added, the *replace* directive can simply be removed. There is no impact on your go codebase: no need to find and replace every import statement which contains that module.

Replacements can be made using the `go mod replace` command but it is very simple to include them in `go.mod` directly. The example `go.mod` file below shows two replacements being made.

The first *replace* directive replaces the module with a locally available version using the absolute path to the module's location on disk.

> **Note**: It's worth stating the obvious here - that local replacements will break builds for other users who don't have the module available at that location so they should probably be removed before the code is shared.

The second *replace* directive substitutes in a forked repository and is safer to share. It achieves the same end, but other users will be able to build the project because the forked version is accessible on the Internet.

```go
module github.com/golangatspeed/replace-example

replace github.com/original/module => /Users/olliephillips/module-my-version
replace github.com/original/module-two => github.com/golangatspeed/module-two

require (
	github.com/original/module v1.0.0
	github.com/original/module-two v1.0.0
)
```

## 5.2 Workspaces

The *workspace* feature was added in Go version 1.18 and furthered the flexibility of modules and dependency management generally.

*Workspaces* allow replacements of modules to be made with modules found locally, **without** making changes to the `go.mod` file.

This avoids the problem of having to add and subsequently remove the replace directives, when working with local module versions, before committing code to version control.

A workspace is defined by a single `go.work` file and is best used for local development, so excluded from version control.

To create a workspace in your project use the commands below.

```bash
// create a workspace file
go work init 

// or create the file and add a local path to the workspace
go work init *path/to/module/or/modules*
```

A `go.work` file created with the second of the two commands would have the following contents.

```go
go 1.18

use *path/to/module/or/modules*
```

Modules which are located within that path will be used in preference to any online versions or locally cached versions.

We can also use the *replace* directive to replace specific modules and versions in the `go.work` file instead of `go.mod`.

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