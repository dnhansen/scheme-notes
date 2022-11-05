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

I go through some of the things that I found confusing when first learning Scheme. The discussion below refers to the R6RS standard.

### Objects, values and expressions

Scheme programs manipulate *objects*, also known as *values*. Types are associated with objects, not variables. Furthermore, variables are not bound to the objects specified in the relevant bindings, but to the location containing these objects. The contents of these locations can be modified by *assignment* using a `set!` expression.

Expressions can be *evaluated*, producing a *value*. Literal expressions such as `42` or `#t` evaluate to 'themselves'. Compound expressions such as `(+ 42 5)` evaluates to `3`. (More precisely, the expression given by the sequence of characters '`(+ 42 5)`' evaluates to the object whose external representation is the sequence of characters '`47`'. See [Syntax](#syntax) below.)

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