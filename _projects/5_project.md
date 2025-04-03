---
layout: page
title: Finding a Solution for A290250 for n=1
description: 
img:
importance: 1
category: fun
---
This was actually a fun thing I undertook but eventually turned out as a course project for my Analysis of Algorithms grad class.

## <u><b>Introduction</b></u>
The sequence [A290250](https://oeis.org/A290250) , for a given n denotes the smallest prime *a(n)* which divides the sum of the $2^n$ powers of factorials from 1 to *a(n)*, i.e.,

$$a(n) \mid \sum_{k=1}^{a(n)}(k!)^ {2^n}$$ 

I computed the solution for n=1, which was computed as, **1248829**.

## <u><b>Overview</b></u>
I divided the problem into two parts
- Computation of prime numbers
- Computation of the sum of the sequence.

I used Python for the implementation.

## <u><b>Implementation</b></u>

### <b>Prime Number Generation</b>
Three main methods werd utilized for this part namely

 1. Sieve of Eratosthenes
 2. Croft's Spiral Sieve
 3. Using `primerange(a,b)` from the `sympy` module in Python.

Once implemented, they were enchmarked using [scalene](https://github.com/plasma-umass/scalene)[^1] to choose the best algorithm, which I used to generate 150000 primes.

#### <u>Sieve of Eratosthenes</u>
The most basic algorithm for prime generation:

On the lines of:

- Make a list of numbers, starting with 2
- Repeat: The first number in the list is prime, cross off multiples of the most recent prime

**Benchmarking it using Scalene:**  
{% include figure.liquid path="assets/img/A290250/sieveScalene.png" title="Scalene Profiling: Sieve of Eratosthenes" %}  
→ Majority of time is spent in branching for checking divisibility, which becomes expensive as `n` increases.

#### <u>Croft's Spiral Sieve</u>
Based on the Croft's Spiral Sieve[^2], This sieve uses the idea that primes > 5 are congruent to 1, 7, 11, 13, 17, 19, 23, or 29 mod 30.  

A dictionary of composite indicators is maintained and wheel factorization helps skip known non-primes.

**Benchmarking it using Scalene:**  
{% include figure.liquid path="assets/img/A290250/croftScalene.png" title="Scalene Profiling: Croft's Spiral Sieve" %}  
→ Most time is spent searching for next candidate primes, but memory overhead is relatively small.

#### <u>sympy.primerange(a,b)</u>
Here, I use the `primerange()` function available in `sympy`. This inbuilt function in `sympy` is highly optimized and splits the interval smartly into blocks.

**Benchmarking it using Scalene:**  
{% include figure.liquid path="assets/img/A290250/sympyScalene.png" title="Scalene Profiling: sympy.primerange()" %}  
→ Fastest overall, but the overhead of importing the library is considerable and affects total runtime.

### <b>Computation of Sum of Prime Factorial powers</b>
For this, I implemented two basic algorithms and benchmarked them to scale up the more effcient algorithm of the two, which I used to compute the solution for our problem here.

#### <u>Algorithm 1</u>
The implementation is as follows:
```python
def algo1(pl): # pl is list of primes
    sum_fact = 0  # Sum of factorials
    fact_num = 1  # Factorial
    for i in range(2, pl[-1]+1):
        fact_num *= (i-1)
        sum_fact += fact_num**2
        if i in pl and sum_fact % i == 0:
            print(True, i) # Print the result as soon as we get the required p
```

**Benchmarking it using Scalene:**  
{% include figure.liquid path="assets/img/A290250/algo1Scalene.png" title="Scalene Profiling: Algorithm 1" %}  
→ 80%+ of time was consumed in factorial squaring and memory-heavy copying (~402 MB/s), making it inefficient for large `n`.

#### <u>Algorithm 2</u>
The implementation is as follows:
```python
def algo2(pl): # pl is list of primes
    sum_fact = 0  # Sum of factorials
    fact_num = 1  # Factorial
    for i in range(2, pl[-1]+1):
        fact_num *= (i-1)**2
        sum_fact += fact_num
        if i in pl and sum_fact % i == 0:
            print(True, i) # Print the result as soon as we get the required p
```

**Benchmarking it using Scalene:**  
{% include figure.liquid path="assets/img/A290250/algo2Scalene.png" title="Scalene Profiling: Algorithm 2" %}  
→ Improved time and memory efficiency; the step computing the square of factorials dominates but is manageable.

Thus I chose Algorithm-2

## <u><b>Results</b></u>
`sympy` is the fastest for the tests, the relative speed up between `sympy.primerange()` and Croft’s Spiral Sieve has also been shown. However due to the overhead caused by importing `sympy`, I chose Croft’s Spiral Sieve for generation of prime numbers.


{% include figure.liquid path="assets/img/A290250/allprimeBenchmark.png" title="Prime Generation Benchmark: sympy vs croft" %}


From benchmarking the algorithms, I see that Algirthm-2 is clearly the fastest,
{% include figure.liquid path="assets/img/A290250/algoBenchmarking.png" title="Algorithm Benchmarking: Algorithm 1 vs 2" %}


## <u><b>Solution</b></u>
I find a solution at n=100000, and verify that there are no more upto n=150000. The solution is the prime number **1248829**

## References
[^1]: Scalene: Berger, E. D., Stern, S., & Pizzorno, J. A. (2023, July). Triangulating Python Performance Issues with Scalene. 17th USENIX Symposium on Operating Systems Design and Implementation (OSDI 23), 51–64. Retrieved from https://www.usenix.org/conference/osdi23/presentation/berger


[^2]: Croft’s Sieve: https://discuss.python.org/t/sieve-of-eratosthenes-in-python/17130/4
