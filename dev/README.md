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

### Errors

#### General

Errors allow us to tell users what went wrong and what they might be able to do to remediate the
problem like refreshing their auth token. Additional context we need to fix errors which users
cannot fix by themselves should be logged. Ideally, the log entry contains an identifier tied to the
error returned to the user. Users should not have to deal with panics, SQL or any low level errors.

A great article I learned a lot from is
https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

We should isolate our code from libraries to make testing and refactoring easier. The same is true
for errors. Returning an error as is allows code further up the call stack to check
using for example `errors.Is`. With Gorm we sometimes do

```go
errors.Is(err, gorm.ErrRecordNotFound)
```

which should only be done in the repository. Gorm is an implementation detail of our repositories
that should not spread throughout our app. So be careful when
[wrapping](https://go.dev/blog/go1.13-errors) errors as this makes 3rd party errors become part of
our API.

Returning errors from 3rd party libraries as is also leads to cryptic errors. Go only prints stack
traces on panics but not when returning an error. So an error from Kubernetes is likely not going to
make sense to a user trying to deploy DHIS2. Instead of returning such an error as is create a new
one with `errors.New/fmt.Errorf` using an error message that makes sense to our users. If the error
contains context you want the user to see then adapt the error using `fmt.Errorf("failed to ...:
%v")`.

#### Guideline

This guide is specific to Gin and is based on https://github.com/gin-gonic/gin/issues/665#issuecomment-257451122

[errdef](https://github.com/dhis2-sre/im-manager/blob/9060e0af8ba4eaeae67309f45b7b4f29d8248ba3/internal/errdef/errdef.go)
contains generic and HTTP independent errors used throughout the app. Our HTTP middleware
[errorHandler](https://github.com/dhis2-sre/im-manager/blob/9060e0af8ba4eaeae67309f45b7b4f29d8248ba3/internal/middleware/errorHandler.go#L27)
sets the HTTP status code based on the type of error.

The following applies to code in handlers:

* if you are ok with the HTTP status code set by the middleware make sure to set the error in the
  Gin context through

```go
c.Error(err)
```

* if you want to return a specific HTTP status code set the error in the Gin context through

```go
c.AbortWithError(http.StatusSomeErrorCode, err)
```

The following applies to code in services:

* use a predefined error from `errdef`. Make sure to check that the error leads to the HTTP status
  code you want.

* if you have to provide some context for a handler to decide what HTTP status code to return return
  a private error. Let the service define an interface the error implements and check for it as
  described in https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

only do so if you have to!

##### nolint errcheck

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

