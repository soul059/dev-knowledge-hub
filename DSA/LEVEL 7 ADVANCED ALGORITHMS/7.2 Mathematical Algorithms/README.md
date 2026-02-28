# 📐 Mathematical Algorithms — Complete Study Guide

> **Level 7 · Advanced Algorithms**
> Master the mathematics that powers competitive programming, cryptography, and computer science.

---

## Course Overview

Mathematical Algorithms form the backbone of efficient problem-solving in computer science. This guide covers everything from basic number theory to computational geometry, providing you with the tools to tackle complex algorithmic challenges.

```
 ┌─────────────────────────────────────────────────────────────┐
 │               MATHEMATICAL ALGORITHMS ROADMAP               │
 ├─────────────────────────────────────────────────────────────┤
 │                                                             │
 │  Number Theory ──► Primes ──► GCD/LCM ──► Modular Arith.  │
 │       │                                        │            │
 │       ▼                                        ▼            │
 │  Combinatorics ◄── Fast Exponentiation ◄── Mod Inverse     │
 │       │                                                     │
 │       ▼                                                     │
 │  Probability ──► Geometry ──► Computational Geometry        │
 │                                      │                      │
 │                                      ▼                      │
 │                          Advanced Number Theory             │
 │                      (Cryptography Applications)            │
 │                                                             │
 └─────────────────────────────────────────────────────────────┘
```

### Why Mathematical Algorithms Matter

| Domain               | Application                                    |
|----------------------|------------------------------------------------|
| Competitive Programming | 30-40% problems need math foundations       |
| Cryptography         | RSA, Diffie-Hellman, Digital Signatures         |
| Computer Graphics    | Geometry, transformations, rendering            |
| Machine Learning     | Probability, statistics, linear algebra         |
| Game Development     | Physics engines, collision detection            |
| Data Science         | Combinatorics, probability distributions        |

---

## 📚 Complete Table of Contents

### [Unit 1 — Number Theory Basics](01-Number-Theory-Basics/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Divisibility](01-Number-Theory-Basics/01-divisibility.md) | Division rules, divisibility tests, properties |
| 2 | [Factors and Multiples](01-Number-Theory-Basics/02-factors-and-multiples.md) | Factor finding, common multiples, factor pairs |
| 3 | [Prime Numbers](01-Number-Theory-Basics/03-prime-numbers.md) | Definition, properties, prime detection basics |
| 4 | [Composite Numbers](01-Number-Theory-Basics/04-composite-numbers.md) | Composite identification, factorization trees |
| 5 | [GCD and LCM](01-Number-Theory-Basics/05-gcd-and-lcm.md) | Basic computation, relationship, applications |
| 6 | [Euclidean Algorithm](01-Number-Theory-Basics/06-euclidean-algorithm.md) | Algorithmic GCD, proof, step-by-step traces |

### [Unit 2 — Prime Numbers](02-Prime-Numbers/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Primality Check (Naive)](02-Prime-Numbers/01-primality-check-naive.md) | Brute force, trial division |
| 2 | [Optimized Primality Check](02-Prime-Numbers/02-optimized-primality-check.md) | √n optimization, 6k±1 trick |
| 3 | [Sieve of Eratosthenes](02-Prime-Numbers/03-sieve-of-eratosthenes.md) | Bulk prime generation, implementation |
| 4 | [Segmented Sieve](02-Prime-Numbers/04-segmented-sieve.md) | Memory-efficient sieve for large ranges |
| 5 | [Prime Factorization](02-Prime-Numbers/05-prime-factorization.md) | Factorization algorithms, SPF sieve |
| 6 | [Count of Divisors](02-Prime-Numbers/06-count-of-divisors.md) | Divisor function, formulas from factorization |

### [Unit 3 — GCD and LCM](03-GCD-and-LCM/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Euclidean Algorithm](03-GCD-and-LCM/01-euclidean-algorithm.md) | Iterative & recursive, proof of correctness |
| 2 | [Extended Euclidean Algorithm](03-GCD-and-LCM/02-extended-euclidean-algorithm.md) | Bézout's identity, finding coefficients |
| 3 | [LCM Computation](03-GCD-and-LCM/03-lcm-computation.md) | Using GCD, overflow handling, properties |
| 4 | [GCD of Array](03-GCD-and-LCM/04-gcd-of-array.md) | Reducing arrays, segment tree GCD |
| 5 | [Properties and Theorems](03-GCD-and-LCM/05-properties-and-theorems.md) | Key theorems, identities, proofs |
| 6 | [Binary GCD](03-GCD-and-LCM/06-binary-gcd.md) | Stein's algorithm, bit manipulation approach |

### [Unit 4 — Modular Arithmetic](04-Modular-Arithmetic/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Modular Addition](04-Modular-Arithmetic/01-modular-addition.md) | Rules, overflow prevention, properties |
| 2 | [Modular Subtraction](04-Modular-Arithmetic/02-modular-subtraction.md) | Handling negatives, safe subtraction |
| 3 | [Modular Multiplication](04-Modular-Arithmetic/03-modular-multiplication.md) | Overflow handling, Russian peasant method |
| 4 | [Modular Division (Inverse)](04-Modular-Arithmetic/04-modular-division-inverse.md) | Modular inverse, Fermat's method, Extended GCD |
| 5 | [Modular Exponentiation](04-Modular-Arithmetic/05-modular-exponentiation.md) | Fast power with mod, binary exponentiation |
| 6 | [Properties of Modulo](04-Modular-Arithmetic/06-properties-of-modulo.md) | Distribution, congruence classes, residues |

### [Unit 5 — Fast Exponentiation](05-Fast-Exponentiation/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Exponentiation by Squaring](05-Fast-Exponentiation/01-exponentiation-by-squaring.md) | Core idea, divide & conquer approach |
| 2 | [Iterative Approach](05-Fast-Exponentiation/02-iterative-approach.md) | Bit-based iteration, implementation |
| 3 | [Recursive Approach](05-Fast-Exponentiation/03-recursive-approach.md) | Recursive halving, stack analysis |
| 4 | [Matrix Exponentiation](05-Fast-Exponentiation/04-matrix-exponentiation.md) | Matrix power, linear recurrence solving |
| 5 | [Fibonacci in O(log n)](05-Fast-Exponentiation/05-fibonacci-in-log-n.md) | Matrix method, fast doubling |
| 6 | [Applications](05-Fast-Exponentiation/06-applications.md) | Modular power, large number handling |

### [Unit 6 — Combinatorics](06-Combinatorics/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Permutations](06-Combinatorics/01-permutations.md) | nPr, arrangements, circular permutations |
| 2 | [Combinations](06-Combinatorics/02-combinations.md) | nCr, selection problems, identities |
| 3 | [Pascal's Triangle](06-Combinatorics/03-pascals-triangle.md) | Construction, properties, applications |
| 4 | [nCr Computation](06-Combinatorics/04-ncr-computation.md) | Efficient methods, DP approach |
| 5 | [nCr with Modulo](06-Combinatorics/05-ncr-with-modulo.md) | Lucas' theorem, factorial mod inverse |
| 6 | [Catalan Numbers](06-Combinatorics/06-catalan-numbers.md) | Formula, applications, generation |

### [Unit 7 — Probability Basics](07-Probability-Basics/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Basic Probability](07-Probability-Basics/01-basic-probability.md) | Sample space, events, rules |
| 2 | [Expected Value](07-Probability-Basics/02-expected-value.md) | Linearity, computation, applications |
| 3 | [Random Algorithms](07-Probability-Basics/03-random-algorithms.md) | Las Vegas, Monte Carlo, randomized search |
| 4 | [Reservoir Sampling](07-Probability-Basics/04-reservoir-sampling.md) | Algorithm R, proof of uniformity |
| 5 | [Shuffle Algorithm](07-Probability-Basics/05-shuffle-algorithm.md) | Fisher-Yates, correctness proof |
| 6 | [Probabilistic Data Structures](07-Probability-Basics/06-probabilistic-data-structures.md) | Bloom filter, Count-Min sketch intro |

### [Unit 8 — Geometry Basics](08-Geometry-Basics/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Point Representation](08-Geometry-Basics/01-point-representation.md) | 2D/3D points, vector basics |
| 2 | [Distance Formulas](08-Geometry-Basics/02-distance-formulas.md) | Euclidean, Manhattan, Chebyshev |
| 3 | [Line Equations](08-Geometry-Basics/03-line-equations.md) | Slope-intercept, standard form, parametric |
| 4 | [Slope Calculations](08-Geometry-Basics/04-slope-calculations.md) | Handling edge cases, parallel/perpendicular |
| 5 | [Intersection Points](08-Geometry-Basics/05-intersection-points.md) | Line-line, line-segment intersection |
| 6 | [Area Calculations](08-Geometry-Basics/06-area-calculations.md) | Shoelace formula, triangle area, polygon area |

### [Unit 9 — Computational Geometry](09-Computational-Geometry/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Convex Hull](09-Computational-Geometry/01-convex-hull.md) | Graham scan, Jarvis march, Andrew's monotone |
| 2 | [Line Intersection](09-Computational-Geometry/02-line-intersection.md) | Cross product method, segment intersection |
| 3 | [Point in Polygon](09-Computational-Geometry/03-point-in-polygon.md) | Ray casting, winding number |
| 4 | [Closest Pair of Points](09-Computational-Geometry/04-closest-pair-of-points.md) | Divide & conquer O(n log n) |
| 5 | [Sweep Line Algorithm](09-Computational-Geometry/05-sweep-line-algorithm.md) | Event-based processing, applications |
| 6 | [Rectangle Overlap](09-Computational-Geometry/06-rectangle-overlap.md) | Overlap detection, intersection area |

### [Unit 10 — Advanced Number Theory](10-Advanced-Number-Theory/)
| # | Chapter | Key Topics |
|---|---------|------------|
| 1 | [Fermat's Little Theorem](10-Advanced-Number-Theory/01-fermats-little-theorem.md) | Statement, proof, applications |
| 2 | [Chinese Remainder Theorem](10-Advanced-Number-Theory/02-chinese-remainder-theorem.md) | CRT system solving, implementation |
| 3 | [Euler's Totient Function](10-Advanced-Number-Theory/03-eulers-totient-function.md) | φ(n), computation, sieve approach |
| 4 | [Multiplicative Inverse](10-Advanced-Number-Theory/04-multiplicative-inverse.md) | Extended GCD method, Fermat's method |
| 5 | [Miller-Rabin Test](10-Advanced-Number-Theory/05-miller-rabin-test.md) | Probabilistic primality, witnesses |
| 6 | [Applications in Cryptography](10-Advanced-Number-Theory/06-applications-in-cryptography.md) | RSA, Diffie-Hellman, key exchange |

---

## 🎯 How to Use This Guide

```
 ┌─────────────────────────────────────────────────────────────┐
 │                    STUDY APPROACH                            │
 ├─────────────────────────────────────────────────────────────┤
 │                                                             │
 │  1. READ the concept and understand the "why"               │
 │          │                                                  │
 │          ▼                                                  │
 │  2. TRACE through the step-by-step examples                 │
 │          │                                                  │
 │          ▼                                                  │
 │  3. IMPLEMENT the pseudocode in your language               │
 │          │                                                  │
 │          ▼                                                  │
 │  4. SOLVE the practice problems                             │
 │          │                                                  │
 │          ▼                                                  │
 │  5. REVIEW using summary tables & revision questions        │
 │                                                             │
 └─────────────────────────────────────────────────────────────┘
```

### Difficulty Progression

| Level | Units | Prerequisites |
|-------|-------|---------------|
| Beginner | 1, 2, 3 | Basic programming |
| Intermediate | 4, 5, 6 | Units 1-3 |
| Advanced | 7, 8, 9 | Units 1-6 |
| Expert | 10 | All previous units |

---

## 🔑 Key Symbols Used

| Symbol | Meaning |
|--------|---------|
| ⚡ | Important concept |
| 🔑 | Key insight |
| ⚠️ | Common pitfall |
| 💡 | Pro tip |
| 📝 | Note |
| 🎯 | Practice problem |

---

> **Start your journey:** [Unit 1 — Number Theory Basics →](01-Number-Theory-Basics/01-divisibility.md)
