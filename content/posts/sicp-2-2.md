---
title: "Hierarchical Data and the Closure Property"
date: 2017-01-02
slug: "sicp-2-2"
tags:
  - "SICP"
---

## Section 2.2  Hierarchical Data and the Closure Property

Exercises in this section of [SCIP](https://mitpress.mit.edu/books/structure-and-interpretation-computer-programs).

**Exercise 2.17.**  Define a procedure `last-pair` that returns the list that contains only the last element of a given (nonempty) list:

```plain
(last-pair (list 23 72 149 34))
(34)
```

**Answer 2.17.**

```plain
(define (last-pair items)
    (if (null? (cdr items))
        items
        (last-pair (cdr items))
    )
)
```

**Exercise 2.18.**  Define a procedure `reverse` that takes a list as argument and returns a list of the same elements in reverse order:

```plain
(reverse (list 1 4 9 16 25))
(25 16 9 4 1)
```

**Answer 2.18.**

```plain
(define (reverse items)
    (if (null? (cdr items))
        items
        (append (reverse (cdr items)) (cons (car items) ()))
    )
)
```

**Exercise 2.19.**  Consider the change-counting program of section 1.2.2. It would be nice to be able to easily change the currency used by the program, so that we could compute the number of ways to change a British pound, for example. As the program is written, the knowledge of the currency is distributed partly into the procedure `first-denomination` and partly into the procedure `count-change` (which knows that there are five kinds of U.S. coins). It would be nicer to be able to supply a list of coins to be used for making change.

We want to rewrite the procedure cc so that its second argument is a list of the values of the coins to use rather than an integer specifying which coins to use. We could then have lists that defined each kind of currency:

```plain
(define us-coins (list 50 25 10 5 1))
(define uk-coins (list 100 50 20 10 5 2 1 0.5))
```

We could then call cc as follows:

```plain
(cc 100 us-coins)
292
```

To do this will require changing the program `cc` somewhat. It will still have the same form, but it will access its second argument differently, as follows:

```plain
(define (cc amount coin-values)
  (cond ((= amount 0) 1)
        ((or (< amount 0) (no-more? coin-values)) 0)
        (else
         (+ (cc amount
                (except-first-denomination coin-values))
            (cc (- amount
                   (first-denomination coin-values))
                coin-values)))))
```

Define the procedures `first-denomination`, `except-first-denomination`, and `no-more?` in terms of primitive operations on list structures. Does the order of the list `coin-values` affect the answer produced by `cc`? Why or why not?

**Answer 2.19.**

```plain
(define (cc amount coin-values)
  (cond ((= amount 0) 1)
        ((or (< amount 0) (no-more? coin-values)) 0)
        (else
         (+ (cc amount
                (except-first-denomination coin-values))
            (cc (- amount
                   (first-denomination coin-values))
                coin-values)))))

(define (no-more? coin-values)
    (null? coin-values)
)

(define (except-first-denomination coin-values)
    (cdr coin-values)
)

(define (first-denomination coin-values)
    (car coin-values)
)

(define us-coins (list 50 25 10 5 1))
(define uk-coins (list 100 50 20 10 5 2 1 0.5))

(cc 100 us-coins)
```

**Exercise 2.20.**  The procedures `+`, `*`, and list take arbitrary numbers of arguments. One way to define such procedures is to use define with *dotted-tail notation*. In a procedure definition, a parameter list that has a dot before the last parameter name indicates that, when the procedure is called, the initial parameters (if any) will have as values the initial arguments, as usual, but the final parameter’s value will be a *list* of any remaining arguments. For instance, given the definition

```plain
(define (f x y . z) <body>)
```

the procedure `f` can be called with two or more arguments. If we evaluate

```plain
(f 1 2 3 4 5 6)
```

then in the body of `f`, `x` will be $1$, `y` will be $2$, and `z` will be the list `(3 4 5 6)`. Given the definition

```plain
(define (g . w) <body>)
```

the procedure `g` can be called with zero or more arguments. If we evaluate

```plain
(g 1 2 3 4 5 6)
```

then in the body of `g`, `w` will be the list `(1 2 3 4 5 6)`.

Use this notation to write a procedure `same-parity` that takes one or more integers and returns a list of all the arguments that have the same even-odd parity as the first argument. For example,

```plain
(same-parity 1 2 3 4 5 6 7)
(1 3 5 7)

(same-parity 2 3 4 5 6 7)
(2 4 6)
```

**Answer 2.20.**

```plain
(define (same-parity first . rest)
    (define (filter x)
        (if (even? x)
            even?
            odd?
        )
    )

    (define (parity f l)
        (cond ((null? l) ())
              ((f (car l)) (cons (car l) (parity f (cdr l))))
              (else (parity f (cdr l)))
        )
    )
    (parity (filter first) rest)
)
```

**Exercise 2.21.**  The procedure `square-list` takes a list of numbers as argument and returns a list of the squares of those numbers.

```plain
(square-list (list 1 2 3 4))
(1 4 9 16)
```

Here are two different definitions of `square-list`. Complete both of them by filling in the missing expressions:

```plain
(define (square-list items)
  (if (null? items)
      nil
      (cons <??> <??>)))
(define (square-list items)
  (map <??> <??>))
```

**Answer 2.21.**

```plain
(define (square-list items)
    (if (null? items)
        `()
        (cons (square (car items)) (square-list (cdr items)))
    )
)

(define (square x) (* x x))

(define (square-list-map items)
    (map square items)
)
```

**Exercise 2.22.**  Louis Reasoner tries to rewrite the first `square-list` procedure of exercise 2.21 so that it evolves an iterative process:

```plain
(define (square-list items)
  (define (iter things answer)
    (if (null? things)
        answer
        (iter (cdr things) 
              (cons (square (car things))
                    answer))))
  (iter items nil))
```

Unfortunately, defining square-list this way produces the answer list in the reverse order of the one desired. Why?

Louis then tries to fix his bug by interchanging the arguments to `cons`:

```plain
(define (square-list items)
  (define (iter things answer)
    (if (null? things)
        answer
        (iter (cdr things)
              (cons answer
                    (square (car things))))))
  (iter items nil))
```

This doesn’t work either. Explain.

**Answer 2.22.** The first procedure doesn’t work because it constructs the last item from the front of the list to the answer. The second procedure seems right at first glance, but result is:

`(square-list (list 1 2 3 4)) ((((() 1) 4) 9) 16)`

Let’s look the definition of `list`:

```plain
(list <a_1> <a_2> ... <a_n>)
(cons <a_1> (cons <a_2> ... (cons <a_n> nil) ... ))
```

And the substitution of the procedure:

```plain
(square-list (list 1 2 3 4))

(iter (list 1 2 3 4) '())

(iter (list 2 3 4) (cons '() (square 1)))

(iter (list 3 4) (cons (cons '() 1)
                       (square 2)))

(iter (list 4)  (cons (cons (cons '() 1) 4)
                      (square 3)))

(iter '() (cons (cons (cons (cons '() 1) 4) 9)
                (square 4)))

(iter '() (cons (cons (cons (cons '() 1) 4) 9) 16))

(cons (cons (cons (cons '() 1) 4) 9) 16)
```

Therefore, the structure of answer is wrong.

**Exercise 2.23.**  The procedure `for-each` is similar to `map`. It takes as arguments a procedure and a list of elements. However, rather than forming a list of the results, `for-each` just applies the procedure to each of the elements in turn, from left to right. The values returned by applying the procedure to the elements are not used at all – `for-each` is used with procedures that perform an action, such as printing. For example,

```plain
(for-each (lambda (x) (newline) (display x))
          (list 57 321 88))
57
321
88
```

The value returned by the call to `for-each` (not illustrated above) can be something arbitrary, such as true. Give an implementation of `for-each`.

**Answer 2.23.**  I find a feature between `cond` and `if` in Lisp. If we want to use `if` instead of `cond`, one things should be concerned. Here are definitions:

```plain
(cond (<predicate> <expression> ...)
	  ...
	  (else <expression> ..)
)

(if <predicate>
	<consequent>
	<alternative>
)
```

In `cond`, one clause has one predicate and several expressions, meanwhile consequent and alternative of `if` only have one expression for each. Thus, in this exercise we should use `cond` because `(newline)` and `(display x)` are two expressions.

```
(define (for-each f l)
    (cond ((null? l) '()) 
          (else (f (car l))
                (for-each f (cdr l)))
    )
)

```

**Exercise 2.24.**  Suppose we evaluate the expression `(list 1 (list 2 (list 3 4)))`. Give the result printed by the interpreter, the corresponding box-and-pointer structure, and the interpretation of this as a tree (as in figure 2.6).
