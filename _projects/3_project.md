---
layout: page
title: Practical Implementation of ElGamal and RSA Cryptosystems
description: 
img: 
importance: 3
category: work
related_publications: false
---

This project was developed as part of the CS789 Cryptography course at Boston University Metropolitan College for my MS in Computer Science program for Fall 2024.

While it also implements the encryption and decryption processes, this project also includes cryptanalysis techniques that demonstrate vulnerabilities when parameters are improperly chosen. For RSA, Pollard's Rho algorithm is implemented for factoring the modulus, while for ElGamal, the Baby-Step Giant-Step algorithm tackles the discrete logarithm problem.

## <b><u>Features</u></b>
This cryptography implementation provides the following capabilities:

- Complete RSA encryption scheme implementation:
    - Key generation with customizable bit length
    - Encryption and decryption functions
    - Cryptanalysis using Pollard's Rho factorization

- Complete ElGamal encryption scheme implementation:

    - Key generation with customizable bit length
    -Encryption and decryption functions
    -Cryptanalysis using Baby-Step Giant-Step algorithm

- Comprehensive utilities for cryptographic operations:
    - Prime number generation
    - Primality testing using Miller-Rabin
    - Fast modular exponentiation
    - Extended Euclidean algorithm for modular inverse
    - Blum-Blum-Shub and Naor-Reingold random number generators

- Extensive test suite for all components:
    - Unit tests for utilities
    - Functional tests for RSA and ElGamal
    - Cryptanalysis validation tests

## <u><b>Technical Structure</b></u>
The file structure is a simple collection of Python Files, 
```
├── Utils.py                  
├── Utils_Test.py             
├── ElGamal_Alice.py          
├── ElGamal_Bob.py            
├── ElGamal_Eve.py            
├── ElGamal_Test.py           
├── RSA_Alice.py              
├── RSA_Bob.py                
├── RSA_Eve.py                
└── RSA_Test.py               
```

<u>Utils.py</u>: This file provides fundamental number-theoretic and cryptographic utilities such as Fast modular exponentiation using "square-and-multiply", Extended Euclidean algorithm for GCD and modular inverses, Prime generation using BBS and Naor-Reingold algorithms, Miller-Rabin primality testing

<u>Utils_Test.py</u>: This file contains the testing function suite for functions present in *Utils.py*

<u>ElGamal_Alice</u>: Contains functions for the key generation of ElGamal for Alice, encryption of plaintext data, and writing the encrypted data to a file. It also prints the public key, private key, and ciphertext to standard output. Additionally, the data is saved in a text file, allowing Bob or Eve to retrieve and process it later if no command-line arguments are provided.

<u>ElGamal_Bob</u>: Implements decryption using the private key, for Bob. In case of absence of command line arguments, they are read as default as *-1* and the *ElGamal_Data.txt* text file is read, and the private key, public key and the ciphertext are read. BoB proceeds to decrypt the ciphertext and computes the plaintext.

<u>ElGamal_Eve</u>: Implements the Baby-Step Giant-Step algorithm to solve the discrete logarithm problem

<u>ElGamal_Test.py</u>: Contains the test suite for ElGamal cryptosystem.

<u>RSA_Alice</u>: Contains the functions for Key Generation of RSA for Alice, encryption of plaintext data, and writing it to a file, and prints the public key, private key and ciphertext to standard outputs. This data is written to a text file so that if at later stage for Bob or Eve, no command line arguments are given, the data can be read from the file to be processed.

<u>RSA_Bob</u>: Implements decryption using the private key.  In case of absence of command line arguments, they are read as default as *-1* and the *RSA_Data.txt* text file is read, and the private key, public key and the ciphertext are read. BoB proceeds to decrypt the ciphertext and computes the plaintext.

<u>RSA_Eve</u>: Implements Pollard's Rho factorization to break RSA encryption

<u>RSA_Test.py</u>: Contains the test suite for RSA cryptosystem.

## <b><u>Usage and Demonstrations</u></b>
### <b>Prerequisites</b>
There are no specific pre-requisites other than having a working Python 3.6+

### <b>Help</b>
Help screens can be running any of the files using the *-h* command line argument,
```python
$ python3 ElGamal_Alice.py -h
usage: ElGamal_Alice.py [-h] [-b BITS] [-m PLAINTEXT]

Generate ElGamal keys, perform encryption as Alice. Sample Input: python3 ElGamal_Alice.py -b 32 -m 42

options:
  -h, --help            show this help message and exit
  -b BITS, --bits BITS  Number of bits for prime generation.
  -m PLAINTEXT, --plaintext PLAINTEXT
                        Plaintext message to encrypt.
```

Another example:
```python
$ python3 RSA_Bob.py -h
usage: RSA_Bob.py [-h] [-p PUBLIC_KEY] [-x PRIVATE_KEY] [-c CIPHERTEXT]

Perform decryption as Bob. Sample Input: python3 RSA_Bob.py -p "(65537, 2558479687)" -x
"(1658196193,2558479687)" -c 469760534

options:
  -h, --help            show this help message and exit
  -p PUBLIC_KEY, --public-key PUBLIC_KEY
                        Public Key, e.g., (e, n)
  -x PRIVATE_KEY, --private-key PRIVATE_KEY
                        Private Key, e.g., (d, nx)
  -c CIPHERTEXT, --ciphertext CIPHERTEXT
                        Ciphertext message to decrypt, e.g., c
```

### <b>Sample Workflows</b>

#### <u>RSA</u>
I will first show a workflow for RSA, I will ask Alice to encrypt the message 66, using a 32bit key
```python
$ python3 RSA_Alice.py -b 32 -m 66
Generating BBS primes...
BBS primes are: (46663, 38723)
Generating two large RSA primes...
RSA primes are: 53239, 74759
Choosing public exponent e = 65537
Computing private exponent d...
Keys generated successfully!
Alice generates the
Public Key: (65537, 3980094401)
Private Key: (255059873, 3980094401)
Ciphertext: 1749118527

Time taken for Alice to
Generate Keys:  0.177573 s
Encrypt:  0.000007 s
```


Running it without arguments for Bob:
```python
$ python3 RSA_Bob.py
Using Public Key: (65537, 3980094401)
Private Key: (255059873, 3980094401) and
Ciphertext: 1749118527
Bob retrieves the plaintext as 66

Time taken for Bob to
 Decrypt:  0.000010 s
```

Bob retrieved the plaintext to be 66.

Now, I run by passing required arguments to Bob,
```python
$ python3 RSA_Bob.py -p "(65537, 3980094401)" -x "(255059873, 3980094401)" -c 1749118527
Using Public Key: (65537, 3980094401)
Private Key: (255059873, 3980094401) and
Ciphertext: 1749118527
Bob retrieves the plaintext as 66

Time taken for Bob to
 Decrypt:  0.000009 s
```

Thus Bob was successful in both the cases.

Breaking the encryption as Eve:
```python
$ python3 RSA_Eve.py -p "(65537, 3980094401)" -c 1749118527
Eve eavesdrop to obtain the Ciphertext and the Public Key...
Eve attempts to factorize n...
Eve found factors: p = 74759, q = 53239
Now Eve will decrypt...
Eve decrypted the plaintext to be 66
Time taken by Eve for
Factorizing n: 0.000239 s
Decrypt:  0.000012 s
```

Thus Eve was successful in extracting the plaintext.

#### <u>ElGamal</u>
As Alice, I proceed to encrpy the plaintext message 789 with a 32 bit key.
```python
$ python3 ElGamal_Alice.py -b 32 -m 789
Generating a large prime...
Keys generated successfully!
Generator: 959518521 and Modulus p : 3209565313
Alice generates the
Public Key: (3209565313, 959518521, 1507717166)
Private Key: (3209565313, 2283141543)
Ciphertext: (352015659, 2379400956)

Time taken for Alice to
Generate Keys:  0.050421 s
Encrypt:  0.000018 s
```

Without providing any arguments for Bob,

```python
$ python3 ElGamal_Bob.py
Using Public Key: (3209565313, 959518521, 1507717166)
Private Key: (3209565313, 2283141543) and
Ciphertext: (352015659, 2379400956)
Bob retrieves the plaintext as 789

Time taken for Bob to
 Decrypt:  0.000035 s
```


Passing the arguments,
```python
$ python3 ElGamal_Bob.py -p "(3209565313, 959518521, 1507717166)" -x "(3209565313, 2283141543)" -c "(352015659, 2379400956)"
Using Public Key: (3209565313, 959518521, 1507717166)
Private Key: (3209565313, 2283141543) and
Ciphertext: (352015659, 2379400956)
Bob retrieves the plaintext as 789

Time taken for Bob to
 Decrypt:  0.000028 s
```

Thus, Bob was successfully able to decrypt and obtain the plaintext as *789* in both the cases.

Now, Eve tries to eavesdrop,

```python
$ python3 ElGamal_Eve.py -p "(3209565313, 959518521, 150771
7166)" -c "(352015659, 2379400956)"
Eve eavesdrops to obtain the Ciphertext and Public Key...
Eve computes Private Key to be 2283141543
Eve was able to decrypt the message successfully as 789

Time taken by Eve for
Decrypt:  0.110319 s
```


Thus, Eve was able to eavesdrop successfully!

#### <u>Unit Tests</u>
Here are the outputs for the unit test suits,

##### Utils

```python
$ python3 Utils_Test.py
......
----------------------------------------------------------------------
Ran 6 tests in 0.000s

OK

The following test cases were run:
Testing Fast Exponentiation
Testing Extended GCD
Testing Multiplicative Inverse
Testing Random Number Generators
Testing Blum-Blum-Shub (BBS)
Testing Naor-Reingold
Testing Miller-Rabin Test
```


##### ElGamal

```python
$ python3 ElGamal_Test.py
Note: This may take a while to start, please do not exit.
................
----------------------------------------------------------------------
Ran 16 tests in 2.403s

OK

The following test cases were run:
Testing ElGamal Key Generation
Testing Key Sizes
Testing Generator Calculation

Testing ElGamal Encryption
Testing Valid Encryption
Testing Boundary Plaintext
Testing Invalid Plaintext

Testing ElGamal Decryption
Testing Valid Decryption
Testing Invalid Private Key
Testing Corrupted Ciphertext

Testing ElGamal Cryptanalysis
Testing Baby-Step Giant-Step Discrete Log

Testing Eve's Decryption
Testing Invalid Input
Testing Small Prime BSGS

Testing ElGamal Advanced Scenarios
Testing Small Bit Size for ElGamal
Testing Large Bit Size for ElGamal
Testing Random Plaintexts
Testing Timing of Key Generation
```

##### RSA
```python
$ python3 RSA_Test.py
.................
----------------------------------------------------------------------
Ran 17 tests in 2.179s

OK

The following test cases were run:
Testing Key Generation
Testing Key Sizes
Testing If Public exponent and Phi(n) are co-prime
Testing Private Exponent

Testing RSA Encryption
Testing Valid Encryption
Testing Invalid Plaintext
Testing Boundary Plaintext

Testing RSA Decryption
Testing Valid Decryption
Testing Invalid Private Key
Testing Corrupted Ciphertext
Testing Advanced RSA Scenarios

Testing Random Plaintexts
Testing Small Bit Size
Testing Large Bit Size
Testing Pollard's Rho Algorithm
Testing Pollard's Rho Edge Cases

Testing Eve's Decryption
Testing Correct Decryption
Testing Incorrect Decryption with Wrong Factors
Testing Edge Case Prime Numbers
Testing Decryption Failure with Non-coprime e
```

#### <u>Interactive Tests</u>
In a group of 3, we were each supposed to act as Alice Eve and Bob respectively, which we were able to demonstrate successfully.