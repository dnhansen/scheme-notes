# Scheme notes

## Installation

To install MIT/GNU Scheme, simply run

    sudo apt-get install mit-scheme


## Usage

The REPL can be started using the `scheme` command. There is also the `mit-scheme` command, and they seem to be virtually identical.

To load a file inside the REPL, use

    (load <path>)

where `<path>` is the relative path of the file.

To run a file with `scheme`, redirect the file to `scheme` as follows:

    scheme < <path>

The flag `--quiet` can be specified before `<` to omit REPL output in the terminal. It may be appropriate to define a shell function such as

    runscheme () {
        scheme --quiet < "$1"
    }

and place this in e.g. `.bashrc` (if using Bash).


## The Scheme language

I go through some of the things that I found confusing when first learning Scheme. The discussion below refers to the R6RS standard, which can be found [here](http://www.r6rs.org/). Note the page of errata.

### Objects, values and types

Scheme programs manipulate *objects*, also known as *values*. *Types* are associated with objects, not variables. Furthermore, variables are not bound to the objects specified in the relevant bindings, but to the location containing these objects.

There are eight mutually disjoint *base types*:

1. boolean
2. pair
3. symbol
4. number
5. char
6. string
7. vector
8. procedure

Furthermore, the empty list is a special object of its own type. If `T` is a base type, then the predicate `T?` can be used to test the type of an object. (In fact, it seems like these predicates can be taken to *define* the base types.) The predicate `null?` tests for the empty list.

Since types are associated with objects and neither to the locations of objects nor to variables bound to these locations, it is possible to replace the object at a location with an object for a different type. In general, the contents of a location can be modified by *assignment* using a `set!` expression. For example, the program

    (define x 2)
    (set! x "hello")

first binds the variable `x` to a new location, then assigns the value `2` to that location. Next the value at this location is changed to the string `"hello"`.

Since variables do not have types, it is impossible to do static type checking. Instead, types are only checked at runtime, so e.g. trying to evaluate the expression

    (+ 2 "hello")

results a runtime type error. On the other hand, when attempting to apply a procedure to the wrong number of arguments, e.g.

    ((lambda (x y) (+ x y)) 2)

Scheme raises an error, saying that the procedure has been called with 1 argument, but it requires 2. This does not seem to be a type error, so the type of a procedure does not include the number of arguments, nor the types of its arguments or return value(s).

Furthermore, since variables do not have types there is nothing that prevents us from writing code that conditionally assigns values of different types to a single location. For instance, the program

    ; b is a boolean previously computed
    (define x
      (if b 2 "hello"))

assigns to the location to which `x` is bound either the number `2` or the string `"hello"`. Notice that this would not be possible in other functional programming languages like Haskell.

Expressions can be *evaluated*, producing a *value*. Literal expressions such as `42` or `#t` evaluate to 'themselves'. Compound expressions such as `(+ 42 5)` evaluates to `3`. (More precisely, the expression given by the sequence of characters '`(+ 42 5)`' evaluates to the object whose external representation is the sequence of characters '`47`'. See [Syntax](#syntax) below.)

### Equivalence predicates

An *equivalence predicate* is essentially a computational equivalence relation. The three most important equivalence predicates are `eq?`, `eqv?` and `equal?`. These have the property that, for objects *obj1* and *obj2*, if *obj1* and *obj2* are `eq?`-equivalent, then they are `eqv?`-equivalent, and if they are `eqv?`-equivalent, then they are `equal?`-equivalent.

#### `eqv?`

R6RS requires that `eqv?` returns `#t` if *obj1* and *obj2* are

1. *booleans*, and they are the same according to the procedure `boolean=?`, which returns `#t` just when its arguments are the same,
2. *symbols*, and they are the same according to the procedure `symbol=?`, which returns `#t` just when its arguments are the same, i.e. they are spelt the same,
3. *exact number objects*, and they are the same according to the procedure `=` (numerical equality) (note that the converse is not required, i.e. `eqv?` may return `#t` while `=` does not),
4. *inexact number objects* [TODO],
5. *characters*, and they are the same according to the procedure `char=?`,
6. *the empty list*,
7. objects such as pairs, vectors, bytevectors, strings, records, ports, or hashtables that refer to the same locations ('locations' plural, since these refer to multiple locations at once) in the store, or
8. *record-type descriptors* [TODO].

Furthermore, R6RS requires that `eqv?` returns `#f` if *obj1* and *obj2* are

1. of different types,
2. *booleans* for which `boolean=?` returns `#f`,
3. *number objects*, but one is exact and the other inexact,
4. different according to `eqv?` when passed through a finite composition of standard arithmetic procedures,
5. *characters* for which `char=?` returns `#f`.
6. If precisely one of *obj1* and *obj2* is the empty list; furthermore, if they are
7. objects such as pairs, vectors, bytevectors, strings, records, ports, or hashtables that refer to distinct locations in the store,
8. pairs, vectors, strings, records, or hashtables, where applying the same accessor (e.g. `car` or `cdr`) to both yields results for which `eqv?` returns false. In particular, two lists with different elements would make `eqv?` return false, since we may recursively apply accessors to access all elements.
9. Finally, if they are procedures that would behave differently for some arguments.

Note that this does *not* fully specify the behaviour of `eqv?`. For instance, in the following situations its behaviour is unspecified:

- `(eqv? "" "")`. Presumably this is because the empty string does not occupy a location.
- When *obj1* and *obj2* are procedures with the exact same behaviour on all inputs (e.g. if they are the same procedure, even occupying the same location). Notice that the specification does not require `eqv?` to ever return `#t` when one of its arguments is a procedure.
- `(eqv? "a" "a")`. Note that strings are compared by `eqv?` based on their location in the store. But the locations containing e.g. strings may be shared between different constants.

#### `eq?`

[TODO]

#### `equal?`

Two objects are equivalent with respect to `equal?` if its arguments, represented as ordered trees, are equal. Here `string=?` is used to compare strings, `bytevector=?` is used to compare bytevectors, and `eqv?` is used to compare other nodes.


### Syntax

The syntax of Scheme is organised in several levels, some of which are:

1. The *lexical syntax*. This describes how a program is divided into lexemes. This includes e.g. literals (such as booleans, numbers, symbols), delimiters, and identifiers.
2. The *datum syntax*, which organises the lexical syntax into *syntactic data*. A datum can either be a *lexeme datum*, which includes literals and identifiers, or a *compound datum*, which can be pairs, lists, vectors, etc. A syntactic datum is also called an *external representation*. A value that can be represented with a syntactic datum (i.e. that has an external representation) is called a *datum value*.

For instance, an external representation for the list containing the numbers `42` and `117` might be `(42 117)` or `(042 117 )`. In particular, datum values can have different external representations. Note that e.g. `(a b c)` and `(a . (b . c . ()))` are equivalent.

It is important to distinguish between a sequence of characters that is the *external representation* of a value, and an expression that *evaluates* to a value. For instance, `(+ 1 2)` is a sequence of characters that is an external representation of the list whose elements are the three data `+`, `1` and `2`, the first of which is an identifier (in fact a so-called *peculiar* identifier), and the latter two are numbers. On the other hand, `(+ 1 2)` is not an expression that evaluates to this list; rather, it is an expression that evaluates to the number `3`. However, if `<datum>` is a syntactic datum, then the expression

    (quote <datum>)

evaluates to the datum value represented by `<datum>`. For instance, `(quote (+ 1 2))` evaluates to the list `(+ 1 2)`. Note that since `(+ 1 2)` is not evaluated, `quote` cannot be a procedure. It is instead a *keyword*, like `define`, `let` and `lambda`. Furthermore, we use the abbreviation `'<datum>` to mean `(quote <datum>)`.

Note that since numbers and strings are their own external representations, they are 'self-quoting'. For instance, `'42` evaluates to the datum value of the sequence of characters `42`, but that is just the number `42`. Furthermore, the external representation of a symbol is an identifier, so `'<identifier>` evaluates to the symbol `<identifier>`.


[TODO expressions, which of the above sequences are expressions?]