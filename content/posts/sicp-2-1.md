---
title: "Introduction to Data Abstraction"
date: 2016-12-31
slug: "sicp-2-1"
math: true
tags:
  - "SICP"
---

## Section 2.1  Introduction to Data Abstraction

Exercises in this section of [SCIP](https://mitpress.mit.edu/books/structure-and-interpretation-computer-programs).

**Exercise 2.1.**  Define a better version of `make-rat` that handles both positive and negative arguments. `Make-rat` should normalize the sign so that if the rational number is positive, both the numerator and denominator are positive, and if the rational number is negative, only the numerator is negative.

**Answer 2.1.**

```plain
(define (numer x) (car x))

(define (denom x) (cdr x))

(define (print-rat x)
    (newline)
    (display (numer x))
    (display "/")
    (display (denom x))
)

(define (make-rat n d)
    (let ((g ((if (< d 0) - +) (abs (gcd n d)))))
        (cons (/ n g) (/ d g))
    )
)
```

**Exercise 2.2.**  Consider the problem of representing line segments in a plane. Each segment is represented as a pair of points: a starting point and an ending point. Define a constructor `make-segment` and selectors `start-segment` and `end-segment` that define the representation of segments in terms of points. Furthermore, a point can be represented as a pair of numbers: the $x$ coordinate and the $y$ coordinate. Accordingly, specify a constructor `make-point` and selectors `x-point` and `y-point` that define this representation. Finally, using your selectors and constructors, define a procedure `midpoint-segment` that takes a line segment as argument and returns its midpoint (the point whose coordinates are the average of the coordinates of the endpoints). To try your procedures, you’ll need a way to print points:

```plain
(define (print-point p)
  (newline)
  (display "(")
  (display (x-point p))
  (display ",")
  (display (y-point p))
  (display ")"))
```

**Answer 2.2.**

```plain
(define (x-point x) (car x))

(define (y-point x) (cdr x))

(define (print-point p)
    (newline)
    (display "(")
    (display (x-point p))
    (display ",")
    (display (y-point p))
    (display ")")
)

(define (make-point x y)
    (cons x y)
)

(define (start-segment x) (car x))

(define (end-segment x) (cdr x))

(define (make-segment x y)
    (cons x y)
)

(define (midpoint-segment s)
    (define (average a b) (/ (+ a b) 2.0))
    (let (
            (a (start-segment s))
            (b (end-segment s))
         )
         (make-point (average (x-point a) (x-point b)) (average (y-point a) (y-point b)))
    )
)
```

**Exercise 2.3.**  Implement a representation for rectangles in a plane. (Hint: You may want to make use of exercise 2.2.) In terms of your constructors and selectors, create procedures that compute the perimeter and the area of a given rectangle. Now implement a different representation for rectangles. Can you design your system with suitable abstraction barriers, so that the same perimeter and area procedures will work using either representation?

**Answer 2.3.** Here, it talks about axis-aligned rectangles.

```plain
(define (x-point x) (car x))

(define (y-point x) (cdr x))

(define (print-point p)
    (newline)
    (display "(")
    (display (x-point p))
    (display ",")
    (display (y-point p))
    (display ")")
)

(define (make-point x y)
    (cons x y)
)

(define (make-rect p1 p2)
    (let (
            (x1 (x-point p1))
            (x2 (x-point p2))
            (y1 (y-point p1))
            (y2 (y-point p2))
         )
         (cond 
            ((and (< x1 x2) (< y1 y2)) (cons p1 p2))
            ((and (> x1 x2) (> y1 y2)) (cons p2 p1))
            ((and (< x1 x2) (> y1 y2)) (cons (make-point (x1 y2)) (make-point (x2 y1))))
            (else (cons (make-point (x2 y1)) (make-point (x1 y2))))
         )
    )
)

(define (bottom-left r) (car r))

(define (top-right r) (cdr r))

(define (print-rect r)
    (print-point (bottom-left r))
    (print-point (top-right r))
)

(define (perimeter r)
    (let (
            (p1 (bottom-left r))
            (p2 (top-right r))
         )
         (let (
                (x1 (x-point p1))
                (x2 (x-point p2))
                (y1 (y-point p1))
                (y2 (y-point p2))
              )
              (* 2 (+ (- x2 x1) (- y2 y1)))
         )
    )    
)

(define (area r)
     (let (
            (p1 (bottom-left r))
            (p2 (top-right r))
         )
         (let (
                (x1 (x-point p1))
                (x2 (x-point p2))
                (y1 (y-point p1))
                (y2 (y-point p2))
              )
              (* (- x2 x1) (- y2 y1))
         )
    )    
)

(define r (make-rect (make-point 1 1)
                     (make-point 4 5)))
```

**Exercise 2.4.**  Here is an alternative procedural representation of pairs. For this representation, verify that `(car (cons x y))` yields $x$ for any objects $x$ and $y$.

```plain
(define (cons x y)
  (lambda (m) (m x y)))

(define (car z)
  (z (lambda (p q) p)))
```

What is the corresponding definition of `cdr`? (Hint: To verify that this works, make use of the substitution model of section 1.1.5.)

**Answer 2.4.**

```plain
(define (cdr z)
	(z (lambda (p q) q))
)
```

Substitution model:

```plain
=> (cdr z)
=> (z (lambda (p q) p))
=> ((lambda (m) (m x y) (lambda (p q) q)))
=> ((lambda (p q) q) x y)
=> y
```

**Exercise 2.5.**  Show that we can represent pairs of nonnegative integers using only numbers and arithmetic operations if we represent the pair $a$ and $b$ as the integer that is the product $2^a 3^b$. Give the corresponding definitions of the procedures `cons`, `car`, and `cdr`.

**Answer 2.5.**

```plain
(define (cons a b)
    (* (expt 2 a) (expt 3 b))
)

(define (logb b n) (floor (/ (log n) (log b))))

(define (car x)
    (logb 2 (/ x (gcd x (expt 3 (logb 3 x)))))
)

(define (cdr x)
    (logb 3 (/ x (gcd x (expt 2 (logb 2 x)))))
)
```

**Exercise 2.6.**  In case representing pairs as procedures wasn’t mind-boggling enough, consider that, in a language that can manipulate procedures, we can get by without numbers (at least insofar as nonnegative integers are concerned) by implementing $0$ and the operation of adding $1$ as

```plain
(define zero (lambda (f) (lambda (x) x)))

(define (add-1 n)
  (lambda (f) (lambda (x) (f ((n f) x)))))
```

This representation is known as *Church numerals*, after its inventor, Alonzo Church, the logician who invented the $\lambda$ calculus.

Define `one` and `two` directly (not in terms of `zero` and `add-1`). (Hint: Use substitution to evaluate (`add-1 zero`)). Give a direct definition of the addition procedure `+` (not in terms of repeated application of `add-1`).

**Answer 2.6.**

```plain
=> (one)
=> (add-1 zero)
=> (lambda (f) (lambda (x) (f ((zero f) x))))
```

`zero` is a function of one argument, that returns a function of one argument that returns the argument.

```plain
=> ((zero f) x)
=> ((lambda(x) x) x)
=> x
```

So, `one` and `two` are:

```plain
=> (one)
=> (lambda (f) (lambda (x) (f x)))

=> (two)
=> (add-1 two)
=> (lambda (f) (lambda (x) (f (f x))))
```

Thus,

```plain
(define one (lambda (f) (lambda (x) (f x))))

(define two (lambda (f) (lambda (x) (f (f x)))))
```

And `add` using the same approach in `add-1`:

```plain
(define (add a b)
	(lambda (f)
		(lambda (x) (a f) ((b f) x))
	)
)
```

**Exercise 2.7.**  Alyssa’s program is incomplete because she has not specified the implementation of the interval abstraction. Here is a definition of the interval constructor:

```plain
(define (make-interval a b) (cons a b))
```

Define selectors `upper-bound` and `lower-bound` to complete the implementation.

**Answer 2.7.**

```plain
(define (upper-bound interval)
	(max (car interval) (cdr interval))
)

(define (lower-bound interval)
	(min (car interval) (cdr interval))
)
```

**Exercise 2.8.**  Using reasoning analogous to Alyssa’s, describe how the difference of two intervals may be computed. Define a corresponding subtraction procedure, called `sub-interval`.

**Answer 2.8.**

```plain
(define (sub-interval x y)
	(make-interval (- (lower-bound x) (upper-bound y))
				   (- (upper-bound x) (lower-bound y)))
)
```

**Exercise 2.9.**  The *width* of an interval is half of the difference between its upper and lower bounds. The width is a measure of the uncertainty of the number specified by the interval. For some arithmetic operations the width of the result of combining two intervals is a function only of the widths of the argument intervals, whereas for others the width of the combination is not a function of the widths of the argument intervals. Show that the width of the sum (or difference) of two intervals is a function only of the widths of the intervals being added (or subtracted). Give examples to show that this is not true for multiplication or division.

**Answer 2.9.** For multiplication and division, the story is different. If the width of the result was a function of the widths of the inputs, then multiplying different intervals with the same widths should give the same answer. For example, multiplying a width 5 interval with a width 1 interval.

**Exercise 2.10.**  Ben Bitdiddle, an expert systems programmer, looks over Alyssa’s shoulder and comments that it is not clear what it means to divide by an interval that spans zero. Modify Alyssa’s code to check for this condition and to signal an error if it occurs.

**Answer 2.10.** We should guarantee:

$$
R_p = \frac{1}{1/R_1+1/R_2} = \frac{R_1+R_2}{R_1 R_2}, \text{where } R_1 R_2 \neq 0.
$$

```plain
(define (div-interval x y)
	(if (>= 0 (* (lower-bound y) (upper-bound y)))
		(error "Divison 0 error")
		(mul-interval x (make-interval (/ 1 (upper-bound y))
							(/ 1 (lower-bound y))
						))
	)
)
```

**Exercise 2.11.**  In passing, Ben also cryptically comments: “By testing the signs of the endpoints of the intervals, it is possible to break `mul-interval` into nine cases, only one of which requires more than two multiplications.” Rewrite this procedure using Ben’s suggestion.

After debugging her program, Alyssa shows it to a potential user, who complains that her program solves the wrong problem. He wants a program that can deal with numbers represented as a center value and an additive tolerance; for example, he wants to work with intervals such as $3.5 \pm 0.15$ rather than $[3.35, 3.65]$. Alyssa returns to her desk and fixes this problem by supplying an alternate constructor and alternate selectors:

```plain
(define (make-center-width c w)
  (make-interval (- c w) (+ c w)))
(define (center i)
  (/ (+ (lower-bound i) (upper-bound i)) 2))
(define (width i)
  (/ (- (upper-bound i) (lower-bound i)) 2))
```

Unfortunately, most of Alyssa’s users are engineers. Real engineering situations usually involve measurements with only a small uncertainty, measured as the ratio of the width of the interval to the midpoint of the interval. Engineers usually specify percentage tolerances on the parameters of devices, as in the resistor specifications given earlier.
