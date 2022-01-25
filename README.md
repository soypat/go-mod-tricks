# go-mod-tricks
go.mod file and `go mod` command tricks to make using modules a breeze.

1. [go.mod file tricks](#go.mod-file)
    * [Replace a module with local copy](#replace-a-module-with-local-copy)

2. [`go mod` command tricks](#go-mod-command)
    * [Updating an untagged module to latest version](#updating-an-untagged-module-to-latest-version)

# go.mod file

## Replace a module with local copy
The following line in a `go.mod` file has the effect of compiling the module with a local version of the module.
```
replace github.com/author/dependency => /path/to/local/copy
```

# `go mod` Command

## Updating an untagged module to latest version
To update an untagged module (no vX.X.X semantic versioning in place) in a module run the following:

```bash
cd my-module
go get -u github.com/author/dependency@branch-name
```
where `branch-name` is the name of the branch with the changes you wish to have

