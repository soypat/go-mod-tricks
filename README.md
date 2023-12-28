# go-mod-tricks
go.mod file and `go mod` command tricks to make using modules "a breeze".

1. [go.mod file tricks](#gomod-file)
    * [Replace a module with local copy](#replace-a-module-with-local-copy)
    * [List modules used in build](#list-modules-used-in-build)


2. [`go mod` command tricks](#go-mod-command)
    * [Updating an untagged module to latest version](#updating-an-untagged-module-to-latest-version)
    * [Why am I importing x module?](#why-am-i-importing-x-module)

3. [Workspace tricks](#workspace-tricks)
    * Workspace initialization
    * Replacing third party modules with local modules
    * Pitfalls

4. [Other tools](#other-tools)
    * [`gomod` dependency graph visualization](#gomod-dependency-graph-visualization)

> "a breeze", really?

No, of course not. Go modules is by far the most complex part of learning Go. The only person who has reached module enlightenment may be Bryan Mills. This document will serve as a cheatsheet to alleviate pain points caused by the complexity for the rest of us. We hope it helps you!

# go.mod file

## Replace a module with local copy
The following line in a `go.mod` file has the effect of compiling the module with a local version of the module.
```
replace github.com/author/dependency => /path/to/local/copy
```

## List modules used in build
The following command lists all modules and their versions used in your build excluding test-only dependencies.

```shell
go list -deps -f '{{with .Module}}{{.Path}} {{.Version}}{{end}}' ./... | sort -u
```

Where `sort -u` excludes repeated entries. Thanks to thepudds on slack for command.

# `go mod` Command

## Updating an untagged module to latest version
To update an untagged module (no vX.X.X semantic versioning in place) in a module run the following:

```shell
cd my-module
go get -u github.com/author/dependency@branch-name
```
where `branch-name` is the name of the branch with the changes you wish to have.If you have recent pushes to the module repository the go tool might miss them since it searches a proxy. To avoid this you can tell the go tool to search the direct source:
```shell
GOPROXY=direct go get -u github.com/author/dependency@branch-name
```

You can also specify a certain commit by replacing `branch-name` with the commit hash:
```shell
go get -u github.com/author/dependency@commit-hash
```


## Why am I importing x module?
To find out how a import is being pulled in you may run the following

```shell
cd my-module
go mod why -m github.com/author/dependency
```
The output of `go mod why` are newline separated import paths where the last one is the package being used in your project:

```
# github.com/author/dependency
my-module
github.com/foo/bar
github.com/author/dependency/some-package
```
In the case above, your module is importing `github.com/foo/bar` which in turn imports `github.com/author/dependency/some-package`.

# Workspace tricks
Go workspaces solve the following problem: **When developing there may come a time when you wish to edit a module you have imported to fix a bug on the side of a third party library, or to add a print statement. Go workspaces enable you to do this in a straightforward way.**

## Workspaces: Initialization
First run the following command to initialize your `go.work` file:

```go
go work init
```
This generates a file named "go.work". Its only contents are a single line with the go version i.e: `go 1.21.0`. A newly created "go.work" file has **no effect** on your program compilation, you must add directives to it first (see next section).

## Workspaces: Replacing a third party module with a local version

1. First you must have an initialized `go.work` file next to your existing `go.mod` file (see first section of [workspace tricks](#workspace-tricks)).

2. You should have also cloned the repository of the go module to your local machine and have it in a nearby directory. 

3. Replace the module by running: 
```bash
go work use $RELATIVE_PATH_TO_MODULE
```

4. You are done! You may start editing the module in `$RELATIVE_PATH_TO_MODULE` freely and changes will be reflected in your program!

### Practical example
We look at a practical example to build more intuition.

Below is an example directory structure. 
```s
parent
├── ours
│   ├── .gitignore
│   ├── go.mod
│   ├── go.work
│   └── main.go
├── module1
│   ├── go.mod
│   └── logic1.go
└── module2
    ├── go.mod
    └── logic2.go
```

For our example we are working in our project inside the `ours` directory. We've already initialized the go.work file in preparation to leverage Go's

We are currently using a remote version of `module1` which we wish to edit. So we've cloned module1 to the same parent directory where our `ours` module resides.

We run the following command from inside `ours` directory

```bash
go work use ../module1
```
The go tool will find the `go.mod` file within this directory, check we are actually using it in our project and finally once it confirms it can actually replace it, edit our `go.work` file adding a `use` directive entry.

Once this is done we may start editing the `module1/logic1.go` file and changes to it will be observed in the `ours` module.

If we wish to also replace `module2` we can just replace `../module1` with `../module2` in the command above.

## Workspaces: Pitfalls
It is important to realize once you start editing your workspaced modules your project will no longer have the same behaviour when someone clones your changes without also uploading your workspaced modules. **It is generally frowned upon to track versions of workspaced changes without including these workspaced modules in your versioning.** And even then, tracking workspaced changes is a very complex task which can lead to unecessary code churn and pain if not done correctly.

Ideally one matches versioning between branches in the module one is primarily working on and the workspaced module, trying to match commit names so that changes can be tracked and reproduced easily. When finally ready to generate a PR one proceeds to create a PR for both the workspaced module and the module using the former.  

It is suggested to not include the `go.work` file within versioning unless it is internally agreed upon in your organization. Add the following lines to your `.gitignore` to this effect:

```bash
go.work
go.work.sum
vendor # Excludes vendor directory from `go vendor` command.
```

# Other tools

## `gomod` dependency graph visualization
Generate visual dependency graphs using https://github.com/Helcaraxan/gomod.
