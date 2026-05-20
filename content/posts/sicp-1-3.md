---
title: "Formulating Abstractions with Higher-Order Procedures"
date: 2016-12-25
slug: "sicp-1-3"
math: true
tags:
  - "SICP"
---

## Section 1.3 Formulating Abstractions with Higher-Order Procedures

Exercises in this section of [SCIP](https://mitpress.mit.edu/books/structure-and-interpretation-computer-programs), and Merry Xmas.

**Exercise 1.29.**  Simpson’s Rule is a more accurate method of numerical integration than the method illustrated above. Using Simpson’s Rule, the integral of a function $f$ between $a$ and $b$ is approximated as

$$
\frac{h}{3}[y_0+4y_1+2y_2+4y_3+…+2y_{n-2}+4y_{n-1}+y_n]
$$

where $h = (b - a)/n$, for some even integer $n$, and $y_k = f(a + kh)$. (Increasing $n$ increases the accuracy of the approximation.) Define a procedure that takes as arguments $f$, $a$, $b$, and $n$ and returns the value of the integral, computed using Simpson’s Rule. Use your procedure to integrate cube between $0$ and $1$ (with $n = 100$ and $n = 1000$), and compare the results to those of the integral procedure shown above.

**Answer 1.29.** The Simpson’s Rule can also be written as follow:

$$
\frac{h}{3}[y_0+4y_1+2y_2+4y_3+…+2y_{n-2}+4y_{n-1}+y_n] = \frac{h}{3}\sum_{j=2,4…}^{n}[y_{j-2} + 4y_{j-1} + y_{j}]
$$

```plain
(define (sum term a next b)
    (if (> a b)
        0
        (+ (term a)
           (sum term (next a) next b)
        )
    )
)

(define (simpson-rule f a b n)
    (define (simpson-next x)
        (+ x 2)
    )
    
    (define h (/ (- b a) n))
    
    (define (y k)
        (f (+ a (* k h)))
    )

    (define (simpson-term x)
        (+
            (y x)
            (* 4 (y (+ x 1)))
            (y (+ x 2))
        )
    )

    (* (sum simpson-term 0 simpson-next n ) (/ h 3))
)

(define (cub x)
    (* x x x)
)

(exact->inexact(simpson-rule cub 0 1 100))
(exact->inexact(simpson-rule cub 0 1 1000))
```

**Exercise 1.30.**  The `sum` procedure above generates a linear recursion. The procedure can be rewritten so that the sum is performed iteratively. Show how to do this by filling in the missing expressions in the following definition:

```plain
(define (sum term a next b)
  (define (iter a result)
      (if <??>
	      <??>
		  (iter <??> <??>))
  (iter <??> <??>)
```

**Answer 1.30.**

```plain
(define (sum term a next b)
	(define (iter a result)
		(if (> a b)
			result
			(iter (next a) (+ result (term a)))
		)
	)
	(iter a 0)
)
```

**Exercise 1.31.**

a.  The `sum` procedure is only the simplest of a vast number of similar abstractions that can be captured as higher-order procedures. Write an analogous procedure called `product` that returns the product of the values of a function at points over a given range. Show how to define `factorial` in terms of product. Also use product to compute approximations to $\pi$ using the formula

$$
\frac{\pi}{4}=\frac{2 \cdot 4 \cdot 4 \cdot 6 \cdot 6 \cdot 8 …}{3 \cdot 3 \cdot 5 \cdot 5 \cdot 7 \cdot 7 …}
$$

b.  If your product procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.

**Answer 1.31.**

$$
\frac{\pi}{4}=\frac{2 \cdot 4 \cdot 4 \cdot 6 \cdot 6 \cdot 8 …}{3 \cdot 3 \cdot 5 \cdot 5 \cdot 7 \cdot 7 …} =
\frac{1}{n+1}\prod_{i=1}^{n}[(\frac{2(i+1)}{2i})^2]
$$

```plain
(define (product term a next b)
    (if (> a b)
        1
        (* (term a) (product term (next a) next b))
    )
)

(define (factorial n)
    (define (identity x)
        x
    )
    (define (next x)
        (+ x 1)
    )
    (product identity 1 next n)
)

(define (product term a next b)
    (define (iter a result)
        (if (> a b)
            result
            (iter (next a) (* result (term a)))
        )
    )
    (iter a 1)
)

(define (square x)
    (* x x)
)

(define (pi-product a b)
    (define (pi-term x)
       (square (/ (* 2 (+ x 1)) (+ (* 2 x) 1)))
    )
    (define (pi-next x)
        (+ x 1)
    )
    (* 4 (/ (product-iter pi-term a pi-next b) (+ b 1)))
)

(exact->inexact(pi-product 1 10000))
```

**Exercise 1.32.**

a. Show that `sum` and `product` (exercise 1.31) are both special cases of a still more general notion called `accumulate` that combines a collection of terms, using some general accumulation function:

```plain
(accumulate combiner null-value term a next b)
```

`Accumulate` takes as arguments the same term and range specifications as `sum` and `product`, together with a `combiner` procedure (of two arguments) that specifies how the current term is to be combined with the accumulation of the preceding terms and a `null-value` that specifies what base value to use when the terms run out. Write `accumulate` and show how `sum` and `product` can both be defined as simple calls to `accumulate`.

b. If your `accumulate` procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.

**Answer 1.32.**

```plain
(define (accumulate combiner null-value term a next b)
    (if (> a b)
        null-value
        (combiner (term a) (accumulate combiner null-value term (next a) next b))
    )
)

(define (accumlate-iter combiner null-value term a next b)
    (define (iter a result)
        (if (> a b) res
            (iter (next a) (combiner res (term a)))
        )
    )
    (iter a null-value)
)

(define (sum term a next b)
    (accumulate + 0 term a next b)
)

(define (product term a next b)
    (accumlate * 1 term a next b)
)
```

**Exercise 1.33.**  You can obtain an even more general version of `accumulate` (exercise 1.32) by introducing the notion of a *filter* on the terms to be combined. That is, combine only those terms derived from values in the range that satisfy a specified condition. The resulting `filtered-accumulate` abstraction takes the same arguments as accumulate, together with an additional predicate of one argument that specifies the filter. Write `filtered-accumulate` as a procedure. Show how to express the following using `filtered-accumulate`:

a. the sum of the squares of the prime numbers in the interval $a$ to $b$ (assuming that you have a `prime`? predicate already written)

b. the product of all the positive integers less than $n$ that are relatively prime to $n$ (i.e., all positive integers $i < n$ such that $GCD(i,n) = 1)$.

**Answer 1.33.**

```plain
(define (filtered-accumulate combiner null-value term a next b filter)
    (if (> a b)
        null-value
        (if (filter a)
            (combiner (term a) (filtered-accumulate combiner null-value (next a) next b filter))
            (combiner null-value (filtered-accumulate combiner null-value (next a) next b filter))
        )
    )
)
```

**Exercise 1.34.**  Suppose we define the procedure

```plain
(define (f g)
  (g 2))
```

Then we have

```plain
(f square)
4

(f (lambda (z) (* z (+ z 1))))
6
```

What happens if we (perversely) ask the interpreter to evaluate the combination `(f f)`? Explain.

**Answer 1.34.**

```plain
=> (f f)
=> (f 2)
=> (2 2)
```

The third invocation will attempt to apply its argument, `2`, to `2`, resulting in error.

**Exercise 1.35.**  Show that the golden ratio $\phi$ (section 1.2.2) is a fixed point of the transformation $x \mapsto 1 + 1/x$, and use this fact to compute $\phi$ by means of the `fixed-point` procedure.

**Answer 1.35.**

```plain
(define tolerance 0.00001)

(define (fixed-point f first-guess)
    (define (close-enough? v1 v2)
        (< (abs (- v1 v2)) tolerance)
    )

    (define (try guess)
        (let ((next (f guess)))
            (if (close-enough? guess next)
                next
                (try next)
            )
        )
    )

    (try first-guess)
)

(fixed-point (lambda (x) (+ 1 (/ 1 x))) 1.0)
```

**Exercise 1.36.**  Modify `fixed-point` so that it prints the sequence of approximations it generates, using the `newline` and `display` primitives shown in exercise 1.22. Then find a solution to $x^x = 1000$ by finding a fixed point of $x \mapsto \log(1000)/\log(x)$. (Use Scheme’s primitive `log` procedure, which computes natural logarithms.) Compare the number of steps this takes with and without average damping. (Note that you cannot start fixed-point with a guess of $1$, as this would cause division by $\log(1) = 0$.)

**Answer 1.36.**

```plain
(define tolerance 0.00001)

(define (fixed-point f first-guess)
    (define (close-enough? v1 v2)
        (< (abs (- v1 v2)) tolerance)
    )

    (define (try guess)
        (display guess)
        (newline)
        (let ((next (f guess)))
            (if (close-enough? guess next)
                next
                (try next)
            )
        )
    )

    (try first-guess)
)

(fixed-point (lambda(x) (/ (log 1000.0) (log x))) 2.0)
```

**Exercise 1.37.**

a. An infinite continued fraction is an expression of the form

$$
f=\frac{N_1}{D_1+\frac{N_2}{D_2+\frac{N_3}{D_3+…}}}
$$

As an example, one can show that the infinite continued fraction expansion with the $N_i$ and the $D_i$ all equal to $1$ produces $1/\phi$, where $\phi$ is the golden ratio (described in section 1.2.2). One way to approximate an infinite continued fraction is to truncate the expansion after a given number of terms. Such a truncation – a so-called *$k$-term finite continued fraction* – has the form

$$
\frac{N_1}{D_1+\frac{N_2}{…+\frac{N_k}{D_k}}}
$$

Suppose that `n` and `d` are procedures of one argument (the term index $i$) that return the $N_i$ and $D_i$ of the terms of the continued fraction. Define a procedure `cont-frac` such that evaluating `(cont-frac n d k)` computes the value of the $k$-term finite continued fraction. Check your procedure by approximating $1/\phi$ using

```plain
(cont-frac (lambda (i) 1.0)
           (lambda (i) 1.0)
           k)
```

for successive values of `k`. How large must you make `k` in order to get an approximation that is accurate to $4$ decimal places?

b. If your `cont-frac` procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.

**Answer 1.37.**

```plain
(define (con-frac n d k)
    (define (frac i)
        (if (< i k)
            (/ (n i) (+ (d i) (frac (+ i 1))))
            (/ (n i) (d i))
        )
    )
    (frac 1)
)

(define (con-frac-iter n d k)
    (define (frac-iter i result)
        (if (= i 0)
            result
            (frac-iter (- i 1) (/ (n i) (+ (d i) result)))
        )
        (frac-iter (- k 1) (/ (n k) (d k)))
    )
)

(con-frac (lambda(i) 1.0) (lambda(i) 1.0) 10)
```

**Exercise 1.38.**  In 1737, the Swiss mathematician Leonhard Euler published a memoir *De Fractionibus Continuis*, which included a continued fraction expansion for $e - 2$, where $e$ is the base of the natural logarithms. In this fraction, the $E_i$ are all $1$, and the $D_i$ are successively $1, 2, 1, 1, 4, 1, 1, 6, 1, 1, 8, …$. Write a program that uses your `cont-frac` procedure from exercise 1.37 to approximate $e$, based on Euler’s expansion.

**Answer 1.38.**

$$
D_i=
\begin{cases}
 2(i+1)/3,  & \text{if } i\bmod 3 = 2;\
 1, & \text{if } i\bmod 3 \neq  2.
\end{cases}
$$

```plain
(define (con-frac n d k)
    (define (frac i)
        (if (< i k)
            (/ (n i) (+ (d i) (frac (+ i 1))))
            (/ (n i) (d i))
        )
    )
    (frac 1)
)

(define (eular k)
    (+ 2.0 (con-frac 
        (lambda(i) 1)
        (lambda(i)
            (if (= (remainder i 3) 2)
                (/ (+ i 1) 1.5)
                1
            )
        )
        k
    ))
)

(eular 100)
```

**Exercise 1.39.**  A continued fraction representation of the tangent function was published in 1770 by the German mathematician J.H. Lambert:

$$
\tan x  = \frac{x}{1-\frac{x^2}{3-\frac{x^2}{5-…}}}
$$

where $x$ is in radians. Define a procedure `(tan-cf x k)` that computes an approximation to the tangent function based on Lambert’s formula. `K` specifies the number of terms to compute, as in exercise 1.37.

**Answer 1.39.**

```plain
(define (con-frac n d k)
    (define (frac i)
        (if (< i k)
            (/ (n i) (- (d i) (frac (+ i 1))))
            (/ (n i) (d i))
        )
    )
    (frac 1)
)

(define (tan-cf x k)
    (con-frac 
        (lambda (i) (
            if (= i 1)
                x
                (* x x)
        ))
        (lambda (i) (
            - (* 2 i) 1
        ))
        k
    )
)
```

**Exercise 1.40.**  Define a procedure `cubic` that can be used together with the `newtons-method` procedure in expressions of the form

```plain
(newtons-method (cubic a b c) 1)
```

to approximate zeros of the cubic $x^3 + ax^2 + bx + c$.

**Answer 1.40.**

```plain
(define (cubic a b c)
    (lambda(x)
        (+
            (cube x)
            (* a (square x))
            (* b x)
            c
        )
    )
)

(define (cube x)
    (* x x x)
)

(define (square x)
    (* x x)
)
```

**Exercise 1.41.**  Define a procedure `double` that takes a procedure of one argument as argument and returns a procedure that applies the original procedure twice. For example, if `inc` is a procedure that adds $1$ to its argument, then `(double inc)` should be a procedure that adds $2$. What value is returned by

```plain
(((double (double double)) inc) 5)
```

**Answer 1.41.**

We guess the procedure `double` is:

```plain
(define (double f)
	(lambda (x) (f (f x)))
)
```

Then:

```plain
=> (((double (double double)) inc) 5) 
=> (((double (lambda (x) (double (double x)))) inc) 5) 
=> ((double (double (double (double inc)))) 5)
```

Since there are 4 `double` procedures, `call` procedure is invoked $2^4 = 16$ times. Thus, result is $5 + 16 = 21$

**Exercise 1.42.**  Let $f$ and $g$ be two one-argument functions. The composition $f$ after $g$ is defined to be the function $x \mapsto  f(g(x))$. Define a procedure `compose` that implements composition. For example, if inc is a procedure that adds $1$ to its argument,

```plain
((compose square inc) 6)
49
```

**Answer 1.42.**

```plain
(define (compose f g)
    (lambda(x)
        (f (g x))
    )
)
```

**Exercise 1.43.**  If $f$ is a numerical function and $n$ is a positive integer, then we can form the $n$th repeated application of $f$, which is defined to be the function whose value at $x$ is $f(f(…(f(x))…))$. For example, if $f$ is the function $x \mapsto  x + 1$, then the $n$th repeated application of $f$ is the function $x \mapsto  x + n$. If $f$ is the operation of squaring a number, then the $n$th repeated application of $f$ is the function that raises its argument to the $2^n$th power. Write a procedure that takes as inputs a procedure that computes $f$ and a positive integer $n$ and returns the procedure that computes the $n$th repeated application of $f$. Your procedure should be able to be used as follows:

```plain
((repeated square 2) 5)
625
```

Hint: You may find it convenient to use `compose` from exercise 1.42.

**Answer 1.43.**

```plain
(define (compose f g)
    (lambda(x)
        (f (g x))
    )
)

(define (repeated f n)
   (if (< n 1)
        (lambda(x) x)
        (compose f (repeated f (- n 1)))
   ) 
)
```

**Exercise 1.44.**  The idea of *smoothing* a function is an important concept in signal processing. If $f$ is a function and $dx$ is some small number, then the smoothed version of $f$ is the function whose value at a point $x$ is the average of $f(x - dx)$, $f(x)$, and $f(x + dx)$. Write a procedure smooth that takes as input a procedure that computes $f$ and returns a procedure that computes the smoothed $f$. It is sometimes valuable to repeatedly smooth a function (that is, smooth the smoothed function, and so on) to obtained the *$n$-fold smoothed function*. Show how to generate the $n$-fold smoothed function of any given function using $smooth$ and $repeated$ from exercise 1.43.

**Answer 1.44.**

```plain
(define (compose f g)
    (lambda(x)
        (f (g x))
    )
)

(define (repeated f n)
   (if (< n 1)
        (lambda(x) x)
        (compose f (repeated f (- n 1)))
   ) 
)

(define (smooth f)
    (lambda (x)
        (/ (+
                (f (- x dx))
                (f x)
                (f (+ x dx))
           )
           3 
        )
    )
)

(define (n-fold-smooth f n)
    ((repeated smooth n) f)
)
```

**Exercise 1.45.**  We saw in section 1.3.3 that attempting to compute square roots by naively finding a fixed point of $y \mapsto x/y$ does not converge, and that this can be fixed by average damping. The same method works for finding cube roots as fixed points of the average-damped $y \mapsto x/y^2$. Unfortunately, the process does not work for fourth roots – a single average damp is not enough to make a fixed-point search for $y \mapsto x/y^3$ converge. On the other hand, if we average damp twice (i.e., use the average damp of the average damp of $y \mapsto  x/y^3$) the fixed-point search does converge. Do some experiments to determine how many average damps are required to compute $n$th roots as a fixed-point search based upon repeated average damping of $y \mapsto x/y^{n-1}$. Use this to implement a simple procedure for computing nth roots using `fixed-point`, `average-damp`, and the `repeated` procedure of exercise 1.43. Assume that any arithmetic operations you need are available as primitives.

**Answer 1.45.** Here reference [SICP 1.45: Computing nth roots](http://www.billthelizard.com/2010/08/sicp-145-computing-nth-roots.html)

To fix the problem, we need to increase the number of times invoking `average-damp`. And we find out the number of average damps, $a$, we can say:

$$
n_{\text{max}} = 2^{a+1} - 1 \\
a =  \lfloor \log _2{n} \rfloor
$$

```plain
(define (nth-root x n)
    (fixed-point
        ((repeated average-damp (floor (/ (log x) (log 2))))
            (lambda (y) (/ x (expt y (- n 1))))
        )
        1.0
    )
)

(define (compose f g)
    (lambda(x)
        (f (g x))
    )
)

(define (repeated f n)
   (if (< n 1)
        (lambda(x) x)
        (compose f (repeated f (- n 1)))
   ) 
)

(define tolerance 0.00001)

(define (fixed-point f first-guess)
    (define (close-enough? v1 v2)
        (< (abs (- v1 v2)) tolerance)
    )

    (define (try guess)
        (let ((next (f guess)))
            (if (close-enough? guess next)
                next
                (try next)
            )
        )
    )

    (try first-guess)
)

(define (average-damp f)
    (define (average x y)
        (/ (+ x y) 2)
    )
    (lambda (x) (average x (f x)))
)

(define (expt b n)
    (define (even? n) (= (remainder n 2) 0))
    (define (square x) (* x x))
    (cond 
        ((= n 0) 1)
        ((even? n) (square (expt b (/ n 2))))
        (else (* b (expt b (- n 1))))
    )
)
```

**Exercise 1.46.**  Several of the numerical methods described in this chapter are instances of an extremely general computational strategy known as *iterative improvement*. Iterative improvement says that, to compute something, we start with an initial guess for the answer, test if the guess is good enough, and otherwise improve the guess and continue the process using the improved guess as the new guess. Write a procedure `iterative-improve` that takes two procedures as arguments: a method for telling whether a guess is good enough and a method for improving a guess. `Iterative-improve` should return as its value a procedure that takes a guess as argument and keeps improving the guess until it is good enough. Rewrite the sqrt procedure of section 1.1.7 and the `fixed-point` procedure of section 1.3.3 in terms of `iterative-improve`.

```plain
(define (close-enough? v1 v2) 
   (define tolerance 1.e-6) 
   (< (/ (abs (- v1 v2)) v2)  tolerance)) 
  
(define (iterative-improve improve close-enough?) 
   (lambda (x) 
     (let ((xim (improve x))) 
       (if (close-enough? x xim) 
           xim 
         ((iterative-improve improve close-enough?) xim)) 
       ))) 
  
(define (sqrt x) 
   ((iterative-improve   
     (lambda (y) 
       (/ (+ (/ x y) y) 2)) 
     close-enough?) 1.0))
```
