# FactCheck.jl

### A test framework for [Julia](http://julialang.org)

[![Build Status](https://travis-ci.org/JuliaLang/FactCheck.jl.png)](https://travis-ci.org/JuliaLang/FactCheck.jl)
[![Coverage Status](https://img.shields.io/coveralls/JuliaLang/FactCheck.jl.svg)](https://coveralls.io/r/JuliaLang/FactCheck.jl)
[![FactCheck](http://pkg.julialang.org/badges/FactCheck_release.svg)](http://pkg.julialang.org/?pkg=FactCheck&ver=release)

`FactCheck.jl` is a [Julia](http://julialang.org) testing framework inspired by the [Midje](https://github.com/marick/Midje) library for Clojure. It aims to add more functionality over the basic [Base.Test](http://docs.julialang.org/en/latest/stdlib/test/).

MIT Licensed - see LICENSE.md

**Installation**: `julia> Pkg.add("FactCheck")`

## Documentation

To get started with `FactCheck`, simply place `using FactCheck` at the top of your test files.

> Note: `FactCheck` produces colored output, but only if you run Julia with the `--color` option, e.g. `julia --color test/runtests.jl`

### Basics

You can use `FactCheck` to do basic assertions like you would with `Base.Test`, e.g.
```julia
using FactCheck

@fact 1 => 1
@fact 2*2 => 4
@fact uppercase("foo") => "FOO"
@fact_throws 2^-1
@fact 2*[1,2,3] => [2,4,6]
```

A `FactCheck` `=>` is more general than the `==` of `Base.Test.@test`.
We refer to the value to the left of the `=>` as the *expression*, and the value to the right of as the *assertion*.
If the assertion is a literal value, like `1`, `"FOO"`, or `[2,4,6]`, then `@fact` checks if the expression is equal to the assertion.
However if the assertion is a *function*, then function will be applied to the expression, e.g.
```julia
@fact 2 => iseven
#...is equivalent to...
@fact iseven(2) => true

@fact Int[] => isempty
#..is equivalent to...
@fact isempy(Int[]) => true
```

`FactCheck` provides several helper functions to make more complicated assertions:

#### `not`
Logical not for literal values and functions.
```julia
@fact 1 => not(2)
# is equivalent to
@fact (1 != 2) => true

@fact 1 => not(iseven)
# is equivalent to
@fact !iseven(1) => true
```

#### `anything`
Anything but `nothing`.
```julia
@fact sin(π) => anything
```

#### `truthy`, `falsey`, `falsey`
To be truthy is to be not `nothing`, false, or 0. To be falsy (or falsey) is to be not truthy.
```julia
@fact 1 => truthy
@fact nothing => falsey
```

#### `exactly`
Test equality in the same way that `Base.is`/`Base.===` do. For example, two distinct objects with the same values are not `exactly` the same e.g.
```julia
a = [1,2,3]
b = [1,2,3]
@fact a => b
@fact a => not(exactly(b))
```

#### `approx`/`roughly`
Test approximate equality of numbers and arrays of numbers using `Base.isapprox`, and accepts same keyword arguments as that function.
```julia
@fact 2 + 1e-5 => roughly(2.0)
@fact 9.5 => roughly(10; atol=1.0)
A = [2.0, 3.0]
B = (1 + 1e-6)*A
@fact A => roughly(B)
```

### Grouping tests

The top-level function `facts` describes the scope of your tests and does the setup required by the test runner.
It can be called with or without a description:
```julia
facts("With a description") do
    # ...
end

facts() do
    # ...
end
```

Related facts can also be grouped inside of a `context`:

```jl
facts("Simple facts") do

    context("numbers are themselves") do
        @fact 1 => 1
        @fact 2 => 2
        @fact 3 => 3
    end

end
```

### Exit status

When a program ends it returns an [exit status](http://en.wikipedia.org/wiki/Exit_status). This is used by other programs to figure out how a program ended. For example, [Travis CI](https://travis-ci.org/) looks at Julia exit code to determine if your tests passed or failed. Because `FactCheck` catches all the test errors, it will return `0` even if a test fails. To address this you can use `exitstatus()` at the end of your tests. This will throw a error, so Julia terminates in an error state.

```jl
module MyPkgTests
    using FactCheck
    # Your tests...
    FactCheck.exitstatus()
end
```

### Workflow

You can run your tests simply by calling them from the command line, e.g. `julia --color test/runtests.jl`, but another option is to place your tests in a module, e.g.

```jl
module MyPkgTests
    # Your tests...
end
```

then repeatedly reload your tests using `reload`, e.g. `julia> reload("test/runtests")`
