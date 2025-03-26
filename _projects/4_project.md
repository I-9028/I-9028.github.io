---
layout: page
title: Speeding Up Scalar Multiplication Over Elliptic Curves
description: 
img:
importance: 4
category: work
---


This project was a group project as a part of my course **AE6102 Parallel Scientific Computing** in my BTech at IIT Bombay in the academic year 2022-23, under the guidance of **Prof. Prabhu Ramachandran**.

## <u><b>Introduction</b></u>
Elliptic Curves form an important basis for existing encryption algorithms. It generates security between key pairs for public key encryption by using the mathematics of elliptic curves. All elliptic curve cryptography is based on the difficulty of finding a secret integer n given the scalar multiple ``Q = [n]P = P + P + . . . + P`` of a base point P on an elliptic curve, say E/k where k is the prime field.


An example would be the curve **Curve25519** represented as $$y^2 = x^3 + 486662x^2 + x$$ 

where k is the prime field ``GF(2^{255} − 19)``


Scalar multiplication is an essential building block for public-key elliptic curve cryptography and has a significant influence on the execution time of ECC algorithms. Therefore, optimized scalar multiplication methods are vital for good ECC performance


## <b><u>Outline</u></b>

The goal of this project was to optimize an elliptic curve-based scalar multiplication algorithm using an appropriate parallelization strategy. We aimed to accelerate elliptic curve scalar multiplication by parallelizing computations. However, our analysis of existing implementations[^1] revealed that they relied on Python’s native *int* class or class-based structures, making them incompatible with parallelization using NumPy and Compyle. 

## <u><b>Methodologies</b></u>
We first designed a specialized data type using NumPy arrays, ensuring compatibility with Numba and Compyle for optimized performance. 

Given this limitation, we decided to develop a custom implementation designed for efficient parallel computation.

After finalizing this data structure, we implemented fundamental elliptic curve algorithms, testing them rigorously to ensure correctness and benchmarking their performance across the five Weierstrass NIST P curves[^3] using the Automan[^2] package.  

Following this, we focused on implementing basic arithmetic functions, including division and the multiplicative inverse, which proved particularly challenging. We identified Python’s *divmod* function as a key component and worked on adapting it to be compatible with our data type. Successfully implementing this function would enable efficient computation of the multiplicative inverse, which is crucial for optimizing scalar multiplication algorithms. 

Finally, with this foundation, we aimed to measure the speedup achieved through parallelization, leveraging our custom array-based approach to improve performance over existing implementations.

## <u><b>Theory</b></u>

### <b>LongNum</b>

We implemented a custom data type to store numbers ranging from 192 bits (P-192 curve) to 521 bits (P-521 curve). This data type uses a **32-element numpy array** where each element is a 32-bit integer (`numpy.uint32`).

#### <u>Implementation Details</u>
- Located in the `arithmetic` folder of the [GitHub repository](https://github.com/ae6102-project).
- Example:  
  `bin(999904040455042) = '0b11100011010110100001001101001001000000101110000010'`  
  Split into 32-bit chunks:
  - `'0b1001101001001000000101110000010'` → stored as `1294207874`
  - `'0b111000110101101000'` → stored as `232808`

#### Performance Insights
- Built on numpy arrays for compatibility with **numba** and **compyle** acceleration.
- Parallelization benefits for operations like multiplication.
- Benchmark results (vs. native Python):
  - **Addition/Subtraction**: ~5× slower
  - **Multiplication**: ~10× slower
  - **Divmod**: Significantly slower → Optimized via **scalene** profiling.

---

### Multiplicative Inverse

We benchmarked three algorithms for inverse computation in GF(*p*):

| Algorithm                     | Complexity   |
|-------------------------------|--------------|
| Trivial                       | *O(p)*       |
| Extended Euclidean Algorithm  | *O(log p)*   |
| Fermat's Little Theorem       | *O(log p)*   |


### <b>Automation for Benchmarking</b>
#### <u>Algorithms Tested</u>
- Scalar multiplication methods:
  - Double-and-Add
  - Montgomery Ladder
  - Joye's Double-and-Add (recursive)

#### <u>Test Setup</u>
1. **Basic Curve**:  
   $$y^2 = x^3 + 2x + 2$$  
   Over GF(17) with generator point *(10, 6)*.
2. **NIST P-Curves**:  
   Tested on P-192, P-224, P-256, and P-384 (parameters loaded via `json`, the parameters located as *json* files in the *curves/* folder in the Github repo).

   Structure of P-521 is as:
   ```json
    {
        "name": "P-521",
        "desc": "",
        "oid": "1.3.132.0.35",
        "form": "Weierstrass",
        "field": {
            "type": "Prime",
            "p": "0x01ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
            "bits": 521
        },
        "params": {
            "a": {
                "raw": "0x01fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffc"
            },
            "b": {
                "raw": "0x0051953eb9618e1c9a1f929a21a0b68540eea2da725b99b315f3b8b489918ef109e156193951ec7e937b1652c0bd3bb1bf073573df883d2c34f1ef451fd46b503f00"
            }
        },
        "generator": {
            "x": {
                "raw": "0x00c6858e06b70404e9cd9e3ecb662395b4429c648139053fb521f828af606b4d3dbaa14b5e77efe75928fe1dc127a2ffa8de3348b3c1856a429bf97e7e31c2e5bd66"
            },
            "y": {
                "raw": "0x011839296a789a3bc0045c8a5fb42c7d1bd998f54449579b446817afbd17273e662c97ee72995ef42640c550b9013fad0761353c7086a272c24088be94769fd16650"
            }
        },
        "order": "0x01fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffa51868783bf2f966b7fcc0148f709a5d03bb5c9b8899c47aebb6fb71e91386409",
        "cofactor": "0x1",
        "aliases": [
            "secg/secp521r1",
            "x963/ansip521r1"
        ],
        "characteristics": {
            "seed": "D09E8800291CB85396CC6717393284AAA0DA64BA",
            "j_invariant": "3619090631887053412807272747807643016060282478111249168973675223587770705025281286979867546071268566958111997954788345609183745222693618155278831649044785613",
            "discriminant": "2687853087729004331535582886185403114835754464152651523509230634031161977750238608042000458607319784141115468556368066113806987449553072575343372028907331922",
            "trace_of_frobenius": "657877501894328237357444332315020117536923257219387276263472201219398408051703"
        }
    }
   ```
3. **Framework**:  
   Automated 80+ test cases using `automan` python package[^2] to measure time for computing *kP* (scalar multiples of point *P*).

### <b><u>Results</u></b>
When we benchmarked *mulyiplicative inverse*, we found that Extended Euclidean Algorithm outperformed others and was selected for implementation.

Time taken by various algorithms over various curves, first we note the effect on all the curves as we scale the value of the scalar *k*, 

        {% include figure.liquid loading="eager" path="assets/img/elliptic/Change_k_curves.png" title="Changes over various curves" class="img-fluid rounded z-depth-1" %}

Now, showing the changes for every algorithm,

        {% include figure.liquid loading="eager" path="assets/img/elliptic/Change_k_algos.png" title="Changes over various Algorithms" class="img-fluid rounded z-depth-1" %}

Thus we see that the time taken by all the algorithms is larger as the number of bits in the elliptic curve field increases.

## <u>References</u>
[^1]: Gerwin Gsenger. (Year 2014). Speeding Up Elliptic Curve Cryptography by Optimizing Scalar Multiplication in Software Implementations. Unpublished master's thesis, Graz University of Technology

[^2]: Prabhu Ramachandran, “automan: A Python-Based Automation Framework for Numerical Computing,” in Computing in Science I\& Engineering, vol. 20, no. 5, pp. 81-97, 2018. doi:10.1109/MCSE.2018.05329818

[^3]: P-192. Accessed May 3, 2023. https://neuromancer.sk/std/nist/P-192

