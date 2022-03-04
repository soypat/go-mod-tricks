# go-mod-tricks
go.mod file and `go mod` command tricks to make using modules "a breeze".

1. [go.mod file tricks](#gomod-file)
    * [Replace a module with local copy](#replace-a-module-with-local-copy)

2. [`go mod` command tricks](#go-mod-command)
    * [Updating an untagged module to latest version](#updating-an-untagged-module-to-latest-version)
    * [Why am I importing x module?](#why-am-i-importing-x-module)

3. [Other tools](#other-tools)
    * [`gomod` dependency graph visualization](#gomod-dependency-graph-visualization)

> "a breeze", really?

No, of course not. Go modules is by far the most complex part of learning Go. The only person who has reached module enlightenment may be Bryan Mills. This document will serve as a cheatsheet to alleviate pain points caused by the complexity for the rest of us. We hope it helps you!

# go.mod file

## Replace a module with local copy
The following line in a `go.mod` file has the effect of compiling the module with a local version of the module.
```
replace github.com/author/dependency => /path/to/local/copy
```

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

# Other tools

## `gomod` dependency graph visualization
Generate visual dependency graphs using https://github.com/Helcaraxan/gomod.
