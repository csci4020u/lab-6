# Calculating the first and follow sets

In this lab, we will focus on algorithms of computing of the FIRST and FOLLOW sets of a grammar.

Consider the following ANTLR grammar:

```
expr : expr '+' expr
     | expr '-' expr
     | '(' expr ')'
     | Number
     ;
```

## What are the terminals and syntactic variables?

The terminals are:

> - `+`
> - `-`
> - `(`
> - `)`
> - Number

The syntactic variables are:

> - `expr`

## Eliminate left recursion

In order to eliminate the left-recursion in the grammar, we first
need to perform left-factoring.

```
expr : expr z1
     | '(' expr ')'
     | Number
     ;

z1 : '+' expr
   | '-' expr
   ;
```

Now, we recognize that the grammar is immediately left recursive, so
we can apply the tranformation to eliminate the immediate left recursion.

```
expr : '(' expr ')' z2
     | Number z2
     ;

z2 : z1 z2
   |
   ;

z1 : '+' expr
   | '-' expr
   ;
```

Note we have a `z2`-production that has an empty body.

Now, the grammar no longer has left recursion.  It's also important to remind us
that we have *not* changed the underlying language.

## Computing FIRST sets

First we will only be computing the first-sets of the symbols.

The terminal symbols are easy:

> | Symbol $x$ | `first`$(x)$ |
> |------------|--------------|
> | `+` | {`+`} |
> | `-` | {`-`} |
> | `(` | {`(`} |
> | `)` | {`)`} |
> | `Number` | {`Number`} |

Moving on to the non-terminals:

> - `z1 : '+' expr` implies `first(z1)` contains `+`
> - `z1 : '-' expr` implies `first(z1)` contains `-`
> ---
> - `first(z1) = { +, -  }`

Let's work out `first(z2)`:

> - `z2 : z1 z2` implies `first(z2)` contains `first(z1) = {+,-}`.
> - `z2 : <epsilon>` implies `first(z2)` contains $ϵ$.
> ---
> - `first(z2) = {+, -, ϵ}`

Let's work out `first(expr)`:

> - `expr : '(' expr ')' z2` implies `first(expr)` contains `(`
> - `expr : Number z2` implies `first(expr)` contains `Number`
> ---
> - `first(expr) = { (, Number }`

This completes the FIRST sets for symbols.

| Symbol $x$ | `first`$(x)$ |
|------------|--------------|
| `+` | {`+`} |
| `-` | {`-`} |
| `(` | {`(`} |
| `)` | {`)`} |
| `Number` | {`Number`} |
| `expr` | {`(, Number`} |
| `z2` | {`+, -, ϵ`} |
| `z1` | {`+, -`} |

## Computing the FOLLOW sets

The follow sets are computed in an iterative fashion.

We start with the follow sets as:

```
expr : { $ }
z1   : {}
z2   : {}
```

Then, we progressively add symbols to these sets.

`expr : '(' expr ')' z2` updates the follow sets:

```
expr : { $ } ∪ { ) } = { $, ) }
z1   : {}
z2   : {} ∪ follow(expr) = { $ }
```

`expr : Number z2` updates the follow sets:

```
expr : { $, ) }
z1   : {}
z2   : { $ } ∪ follow(expr) = { $ }
```

`z2 : z1 z2` updates the follow sets:

```
expr : { $, ) }
z1   : {} ∪ first0(z2) ∪ follow(z2) = {+, -, $}
z2   : { $ }
```

`z1 : '+' expr` updates the follow sets:

```
expr : { $, ) } ∪ follow(z1) = { $, ), +, - }
z1   : { +, -, $ }
z2   : { $ }
```


`z1 : '-' expr` updates the follow sets:

```
expr : { $, ) } ∪ follow(z1) = { $, ), +, - }
z1   : { +, -, $ }
z2   : { $ }
```

`expr : '(' expr ')' z2` updates the follow sets:

```
expr : { $, ) } ∪ follow(z1) = { $, ), +, - }
z1   : { +, -, $ }
z2   : { $, ), +, - }
```

You can try to apply these rules again, and see that the follow sets are no
longer updated.  This means that we can terminate the iteration.  The follow
sets are:

| Syntactc variable $x$ | `follow`$(x)$ |
|-----------------------|---------------|
| expr | `{$, ), +, -}` |
| z1   | `{+, -, $}` |
| z2   | `{ $ }` |
