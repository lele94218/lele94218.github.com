---
title: "Building Abstractions with Procedures Section 1"
date: 2016-12-12
slug: "scip-1-1"
tags:
  - "SICP"
---

## Section 1.1 The Elements of Programing

Exercises in this section of [SICP](https://mitpress.mit.edu/books/structure-and-interpretation-computer-programs).

**Exercise 1.1.** Below is a sequence of expressions. What is the result printed by the interpreter in response to each expression? Assume that the sequence is to be evaluated in the order in which it is presented.

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 ``` | ``` 10 (+ 5 3 4) (- 9 1) (/ 6 2) (+ (* 2 4) (- 4 6)) (define a 3) (define b (+ a 1)) (+ a b (* a b)) (= a b) (if (and (> b a) (< b (* a b)))     b     a) (cond ((= a 4) 6)       ((= b 4) (+ 6 7 a))       (else 25)) (+ 2 (if (> b a) b a)) (* (cond ((> a b) a)          ((< a b) b)          (else -1))    (+ a 1)) ``` |

**Answer 1.1.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 ``` | ``` 10 12 8 3 6 a b 19 #f 4 16 6 16 ``` |

**Exercise 1.2.** Translate the following expression into prefix form

$$  
\frac{5+4+(2-(3-6 + \frac{4}{5}))}{3(6-2)(2-7)}  
$$

**Answer 1.2.**

|  |  |
| --- | --- |
| ``` 1 ``` | ``` (/ (+ 5 4 (- 2 (- 3 (+ 6 (/ 4 3))))) (* 3 (- 6 2) (- 2 7))) ``` |

**Exercise 1.3.** Define a procedure that takes three numbers as arguments and returns the sum of the squares of the two larger numbers.

**Answer 1.3.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 ``` | ``` (DEFINE (SOS x y) (+ (* x x) (* y y))) (DEFINE (F x y z) 	(COND ((AND (<= x y) (<= x z)) (SOS y z)) 	      ((AND (<= y x) (<= y z)) (SOS x z)) 	      ((AND (<= z x) (<= z y)) (SOS x y)) 	)	 ) ``` |

**Exercise 1.4.** Observe that our model of evaluation allows for combinations whose operators are compound expressions. Use this observation to describe the behavior of the following procedure:

|  |  |
| --- | --- |
| ``` 1 2 ``` | ``` (define (a-plus-abs-b a b)   ((if (> b 0) + -) a b)) ``` |

**Answer 1.4.**

$$  
a+|b|  
$$

\_a\_ plus the absolute value of \_b\_.

**Exercise 1.5.** Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 ``` | ``` (define (p) (p))  (define (test x y)   (if (= x 0)       0       y)) ``` |

Then he evaluates the expression:

|  |  |
| --- | --- |
| ``` 1 ``` | ``` (test 0 (p)) ``` |

What behavior will Ben observe with an interpreter that uses applicative-order evaluation? What behavior will he observe with an interpreter that uses normal-order evaluation? Explain your answer. (Assume that the evaluation rule for the special form `if` is the same whether the interpreter is using normal or applicative order: The predicate expression is evaluated first, and the result determines whether to evaluate the consequent or the alternative expression.)

**Answer 1.5.**

The applicative-order evaluation:

|  |  |
| --- | --- |
| ``` 1 2 3 ``` | ``` => (test 0 (p)) => (test 0 ((p))) => (test 0 (((p)))) ``` |

As we see, the evaluation of `(test 0 (p))` is infinitive.

The normal-order evaluation:

|  |  |
| --- | --- |
| ``` 1 2 3 4 ``` | ``` => (test 0 (p)) => (if (= 0 0) 0 (p)) => (if #t 0 (p)) => 0 ``` |

The expression evaluates to `0`.

**Exercise 1.6.** Alyssa P. Hacker doesn’t see why if needs to be provided as a special form. “Why can’t I just define it as an ordinary procedure in terms of `cond`?” she asks. Alyssa’s friend Eva Lu Ator claims this can indeed be done, and she defines a new version of `if`:

|  |  |
| --- | --- |
| ``` 1 2 3 ``` | ``` (define (new-if predicate then-clause else-clause)   (cond (predicate then-clause)         (else else-clause))) ``` |

Eva demonstrates the program for Alyssa:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 ``` | ``` (new-if (= 2 3) 0 5) 5  (new-if (= 1 1) 0 5) 0 ``` |

Delighted, Alyssa uses new-if to rewrite the square-root program:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 ``` | ``` (define (sqrt-iter guess x)   (new-if (good-enough? guess x)           guess           (sqrt-iter (improve guess x)                      x))) ``` |

What happens when Alyssa attempts to use this to compute square roots? Explain.

**Answer 1.6.** I believe this program is incorrect, because `new-if` uses applicative-order evaluation. Thus, it always expands the `else-clause` and causes an infinite recursion. In contrast, `if` uses normal-order evaluation.

**Exercise 1.7.** The `good-enough`? test used in computing square roots will not be very effective for finding the square roots of very small numbers. Also, in real computers, arithmetic operations are almost always performed with limited precision. This makes our test inadequate for very large numbers. Explain these statements, with examples showing how the test fails for small and large numbers. An alternative strategy for implementing `good-enough`? is to watch how guess changes from one iteration to the next and to stop when the change is a very small fraction of the guess. Design a square-root procedure that uses this kind of end test. Does this work better for small and large numbers?

**Answer 1.7.** The results are very inaccurate when calculating `sqrt(0.0001)` and `sqrt(100000000)`. We can modify `good-enough` using current guess and previous guess below:

$$  
\frac{|G-G\_{pre}|}{G}<0.001  
$$

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 ``` | ``` (define (good-enough? guess prev-guess)   (< (/ (abs (- guess prev-guess))         guess)      0.001))  (define (sqrt-iter guess prev-guess x)   (if (good-enough? guess prev-guess)     guess     (sqrt-iter (improve guess x)                guess                x)))  (define (sqrt x)     (sqrt-iter 1.0 0 x)) ``` |

**Exercise 1.8.** Newton’s method for cube roots is based on the fact that if y is an approximation to the cube root of x, then a better approximation is given by the value

$$  
\frac{x/y^2+2y}{3}  
$$

Use this formula to implement a cube-root procedure analogous to the square-root procedure.

**Answer 1.8.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 ``` | ``` (define (cub-iter guess prev-guess x)   (if (good-enough? guess prev-guess)     guess     (cub-iter (improve guess x) guess x)     )   )  (define (good-enough? guess prev-guess)   (< (/ (abs (- guess prev-guess)) guess) 0.001)   )  (define (improve guess x)   (/ (+ (/ x (square guess)) (* 2 guess)) 3)   )  (define (square x)   (* x x)   )  (define (cubroot x)    ((if (< x 0) - +) (cub-iter (improve 1.0 (abs x)) 1 (abs x)))   ) ``` |
