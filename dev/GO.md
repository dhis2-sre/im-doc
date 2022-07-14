# Go

## Audience

This document is intended for developers of the Instance Manager.

## Style

The instance manager is developed in the programming language [Go](https://go.dev/). We strive to
use a consistent style :bowtie:. This documents the rules and conventions we have settled on and the
tools we use to support this.

Code is formatted using [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) and linted
using [golangci-lint](https://golangci-lint.run/). Every service has a `Makefile` that initializes a
[pre-commit](https://pre-commit.com/) git hook for you using `make init`.

Not all rules and conventions are automated. The following

* [Effective Go](https://go.dev/doc/effective_go)
* [Go code review](https://github.com/golang/go/wiki/CodeReviewComments)

are guides we mostly follow. If we diverge, we will list why and how below :neckbeard:

### nolint errcheck

Use the [blank identifier](https://go.dev/doc/effective_go#blank) when ignoring a returned error
instead of a [nolint directive](https://golangci-lint.run/usage/false-positives/#nolint-directive)

So

```go
_ = os.Remove("file.go") 
```

instead of

```go
_ = os.Remove("file.go") // nolint:errcheck
```

Reason: IntelliJ does not support nolint directives. Using the blank identifier removes linting
errors in IntelliJ when we deliberatly want to ignore the error.

### Naming

[Short names](https://go.dev/doc/effective_go#names) are good but lets not overdo it.

Prefer
* `user` over `usr`

### Initializing constructor

> Sometimes the zero value isn't good enough and an initializing constructor is necessary
https://go.dev/doc/effective_go#composite_literals

Put the `New` function before the type declaration of the struct it creates.

