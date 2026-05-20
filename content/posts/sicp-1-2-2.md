---
title: "Building Abstractions with Procedures Section 2 - Part 2"
date: 2016-12-20
slug: "sicp-1-2-2"
tags:
  - "SICP"
---

## Section 1.2 Procedures and the Processes They Generate

Exercises in this section of [SCIP](https://mitpress.mit.edu/books/structure-and-interpretation-computer-programs).

**Exercise 1.20.** The process that a procedure generates is of course dependent on the rules used by the interpreter. As an example, consider the iterative `gcd` procedure given above. Suppose we were to interpret this procedure using normal-order evaluation, as discussed in section 1.1.5. (The normal-order-evaluation rule for if is described in exercise 1.5.) Using the substitution method (for normal order), illustrate the process generated in evaluating `(gcd 206 40)` and indicate the `remainder` operations that are actually performed. How many `remainder` operations are actually performed in the normal-order evaluation of `(gcd 206 40)`? In the applicative-order evaluation?

**Answer 1.20.**

*Normal-order evaluation*: fully expand and then reduce. 18 total `remainder`.

*Applicative-order evaluation*: evaluate arguments and then apply operands. 4 total `remainder`.

**Exercise 1.21.** Use the smallest-divisor procedure to find the smallest divisor of each of the following numbers: $199$, $1999$, $19999$.

**Answer 1.21.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 ``` | ``` (define (smallest-divisor n)     (find-divisor n 2) )  (define (find-divisor n test-divisor)     (cond ((> (square test-divisor) n) n)           ((divides? test-divisor n) test-divisor)           (else (find-divisor n (+ test-divisor 1)))     ) )  (define (divides? a b)     (= (remainder b a) 0) )  (smallest-divisor 199) => 199 (smallest-divisor 1999) => 1999 (smallest-divisor 19999) => 7 ``` |

**Exercise 1.22.** Most Lisp implementations include a primitive called `runtime` that returns an integer that specifies the amount of time the system has been running (measured, for example, in microseconds). The following `timed-prime-test` procedure, when called with an integer $n$, prints $n$ and checks to see if $n$ is prime. If $n$ is prime, the procedure prints three asterisks followed by the amount of time used in performing the test.

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 ``` | ``` (define (timed-prime-test n)   (newline)   (display n)   (start-prime-test n (runtime))) (define (start-prime-test n start-time)   (if (prime? n)       (report-prime (- (runtime) start-time)))) (define (report-prime elapsed-time)   (display " *** ")   (display elapsed-time)) ``` |

Using this procedure, write a procedure `search-for-primes` that checks the primality of consecutive odd integers in a specified range. Use your procedure to find the three smallest primes larger than $1000$; larger than $10,000$; larger than $100,000$; larger than $1,000,000$. Note the time needed to test each prime. Since the testing algorithm has order of growth of $\theta(\sqrt n)$, you should expect that testing for primes around $10,000$ should take about $\sqrt{10}$ times as long as testing for primes around $1000$. Do your timing data bear this out? How well do the data for $100,000$ and $1,000,000$ support the $\sqrt n$ prediction? Is your result compatible with the notion that programs on your machine run in time proportional to the number of steps required for the computation?

**Answer 1.22.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 ``` | ``` (define runtime real-time-clock)  (define (timed-prime-test n)   (newline)   (display n)   (start-prime-test n (runtime)))  (define (start-prime-test n start-time)   (if (prime? n)       (report-prime (- (runtime) start-time))))  (define (report-prime elapsed-time)   (display " *** ")   (display elapsed-time))  (define (find-divisor n test-divisor) 	(cond  		((> (square test-divisor) n) n) 		((divides? test-divisor n) test-divisor) 		(else (find-divisor n (+ test-divisor 1))) 	) )  (define (square x) 	(* x x) )  (define (divides? a b) 	(= (remainder b a) 0) )  (define (smallest-divisor n) 	(find-divisor n 2) )  (define (prime? n) 	(= n (smallest-divisor n)) )  (define (search-for-primes start end) 	(if (even? start) 		(search-for-primes (+ start 1) end) 		(cond  			((< start end) (timed-prime-test start) 			(search-for-primes (+ start 2) end)) 		) 	) )  (search-for-primes 1000000 1000100) ``` |

**Exercise 1.23.** The `smallest-divisor` procedure shown at the start of this section does lots of needless testing: After it checks to see if the number is divisible by $2$ there is no point in checking to see if it is divisible by any larger even numbers. This suggests that the values used `for test-divisor` should not be $2, 3, 4, 5, 6, …$, but rather $2, 3, 5, 7, 9, …$. To implement this change, define a procedure next that returns $3$ if its input is equal to $2$ and otherwise returns its input plus $2$. Modify the `smallest-divisor` procedure to use `(next test-divisor)` instead of `(+ test-divisor 1)`. With `timed-prime-test` incorporating this modified version of `smallest-divisor`, run the test for each of the $12$ primes found in exercise 1.22. Since this modification halves the number of test steps, you should expect it to run about twice as fast. Is this expectation confirmed? If not, what is the observed ratio of the speeds of the two algorithms, and how do you explain the fact that it is different from $2$?

**Answer 1.23.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 ``` | ``` (define (find-divisor n test-divisor)     (cond         ((> (square test-divisor) n) n)         ((divides? test-divisor n) test-divisor)         (else (find-divisor n (next test-divisor)))     ) )  (define (next test-divisor)     (if (= test-divisor 2) (+ test-divisor 1)                            (+ test-divisor 2)     ) )  (define (square x)     (* x x) )  (define (divides? a b)     (= (remainder b a) 0) )  (define (smallest-divisor n)     (find-divisor n 2) )  (define (prime? n)     (= (smallest-divisor n) n) ) ``` |

**Exercise 1.24.** Modify the timed-prime-test procedure of exercise 1.22 to use `fast-prime?` (the Fermat method), and test each of the $12$ primes you found in that exercise. Since the Fermat test has $\theta (\log n)$ growth, how would you expect the time to test primes near $1,000,000$ to compare with the time needed to test primes near 1000? Do your data bear this out? Can you explain any discrepancy you find?

**Answer 1.24.**

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 ``` | ``` (define (expmod base exp m)     (cond          ((= exp 0) 1)         ((even? exp) (remainder (square (expmod base (/ exp 2) m)) m))         (else (remainder (* base (expmod base (- exp 1) m)) m))     ) )  (define (fermat-test n)     (define (try-it a)         (= (expmod a n n) a)     )     (try-it (+ 1 (random (- n 1)))) )  (define (fast-prime? n times)     (cond         ((= times 0) true)         ((fermat-test n) (fast-prime? n (- times 1)))         (else false)     ) )  (define (prime? n)     (fast-prime? n 100 ) ) ``` |

**Exercise 1.25.** Alyssa P. Hacker complains that we went to a lot of extra work in writing `expmod`. After all, she says, since we already know how to compute exponentials, we could have simply written

|  |  |
| --- | --- |
| ``` 1 2 ``` | ``` (define (expmod base exp m)   (remainder (fast-expt base exp) m)) ``` |

Is she correct? Would this procedure serve as well for our fast prime tester? Explain.

**Answer 1.25.** Scheme is able to handle arbitrary-precision arithmetic, but arithmetic with arbitrarily long numbers is computationally expensive. This means that we get the correct results, but it takes considerably longer.

**Exercise 1.26.** Louis Reasoner is having great difficulty doing exercise 1.24. His `fast-prime?` test seems to run more slowly than his `prime?` test. Louis calls his friend Eva Lu Ator over to help. When they examine Louis’s code, they find that he has rewritten the `expmod` procedure to use an explicit multiplication, rather than calling square:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 ``` | ``` (define (expmod base exp m)   (cond ((= exp 0) 1)         ((even? exp)          (remainder (* (expmod base (/ exp 2) m)                        (expmod base (/ exp 2) m))                     m))         (else          (remainder (* base (expmod base (- exp 1) m))                     m)))) ``` |

“I don’t see what difference that could make,” says Louis. “I do.” says Eva. “By writing the procedure like that, you have transformed the $\theta (\log n)$ process into a $\theta (n)$ process.’’ Explain.

**Answer 1.26.** Let $n$ is exponent which can generate a complete binary tree recursion. The depth of the tree is $\log n + 1$, the total number of nodes is $2^{\log n + 1} - 1$, is $2n -1 = \theta (n)$.

**Exercise 1.27.** Demonstrate that the Carmichael numbers listed in footnote 47 really do fool the Fermat test. That is, write a procedure that takes an integer $n$ and tests whether $a^n$ is congruent to $a$ modulo $n$ for every $a < n$, and try your procedure on the given Carmichael numbers.

**Answer 1.27.** Carmichael numbers fools Fermat test.

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 ``` | ``` (define (expmod base exp m)     (cond          ((= exp 0) 1)         ((even? exp) (remainder (square (expmod base (/ exp 2) m)) m))         (else (remainder (* base (expmod base (- exp 1) m)) m))     ) )  (define (fermat-test n test-value)     (define (try-it a)         (= (expmod a n n) a)     )     (try-it test-value) )  (define (fast-prime? n times)     (cond         ((= times 0) true)         ((fermat-test n times) (fast-prime? n (- times 1)))         (else false)     ) )  (define (prime? n)     (fast-prime? n (- n 1)) )  (prime? 561) (prime? 1105) (prime? 1729) (prime? 2465) ``` |

**Exercise 1.28.** One variant of the Fermat test that cannot be fooled is called the *Miller-Rabin test* (Miller 1976; Rabin 1980). This starts from an alternate form of Fermat’s Little Theorem, which states that if $n$ is a prime number and $a$ is any positive integer less than $n$, then $a$ raised to the $(n - 1)$st power is congruent to $1$ modulo $n$. To test the primality of a number $n$ by the Miller-Rabin test, we pick a random number $a<n$ and raise $a$ to the $(n - 1)$st power modulo $n$ using the `expmod` procedure. However, whenever we perform the squaring step in `expmod`, we check to see if we have discovered a “nontrivial square root of $1$ modulo $n$,” that is, a number not equal to $1$ or $n - 1$ whose square is equal to $1$ modulo $n$. It is possible to prove that if such a nontrivial square root of $1$ exists, then $n$ is not prime. It is also possible to prove that if $n$ is an odd number that is not prime, then, for at least half the numbers $a<n$, computing $a^{n-1}$ in this way will reveal a nontrivial square root of $1$ modulo $n$. (This is why the Miller-Rabin test cannot be fooled.) Modify the `expmod` procedure to signal if it discovers a nontrivial square root of $1$, and use this to implement the Miller-Rabin test with a procedure analogous to `fermat-test`. Check your procedure by testing various known primes and non-primes. Hint: One convenient way to make `expmod` signal is to have it return $0$.

**Answer 1.28.** I got stuck in the exercise for hours. After reading other resources about Miller-Rabin test, I find out we don’t worry about the proof that “if such a nontrivial square root of $1$ exists, then $n$ is not prime”. Just know about the condition apply to the procedure.

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 ``` | ``` (define (expmod base exp m)     (cond         ((= exp 0) 1)         ((even? exp)             (let ((x (expmod base (/ exp 2) m)))                 (if (non-trivial-sqrt? x m) 0 (remainder (square x) m))             )         )         (else (remainder (* base (expmod base (- exp 1) m)) m))     ) )  (define (non-trivial-sqrt? n m)     (cond         ((= n 1) false)         ((= n (- m 1)) false)         (else (= (remainder (square n) m) 1))     ) )  (define (square x)     (* x x) )  (define (miller-rabin-test a n)     (cond         ((= a 0) true)         ((= (expmod a (- n 1) n ) 1) (miller-rabin-test (- a 1) n))         (else false)     ) )  (define (prime? n)     (miller-rabin-test (- n 1) n) )  (prime? 561) (prime? 1105) (prime? 1729) (prime? 2465) ``` |
