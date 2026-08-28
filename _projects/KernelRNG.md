---
layout: page
title: A Study of OS Kernel Random Number Generators
description: A comparative study of random number generation in the Windows, Linux, and macOS kernels, and its role in system security.
img:
importance: 2
category: Projects
related_publications: false
---

This project was completed for my course **CS575** in the **MS in Computer Science** program at **Boston University, Metropolitan College**. It studies how the Windows, Linux, and macOS kernels generate random numbers, and why this matters for system security.

## <u><b>Introduction</b></u>

Modern systems depend on secret, unpredictable values. The kernel uses random numbers for cryptographic keys, for Address Space Layout Randomization (ASLR), and for many other security mechanisms. If the random numbers are weak or predictable, an attacker can defeat these protections. A strong random number generator (RNG) is therefore a core part of kernel security.

## <u><b>Types of Random Number Generator</b></u>

There are two main types of RNG:

- **Pseudo-Random Number Generator (PRNG)** — a deterministic algorithm. It keeps a fixed internal state and computes its output from that state.
- **Hardware Random Number Generator (HRNG)** — a generator based on physical events that are hard to predict, for example hardware interrupts, thermal noise, or camera noise.

The quality of an RNG depends on **entropy**. Entropy is the true randomness that the system collects from unpredictable sources for use in cryptography.

## <u><b>Windows</b></u>

Windows holds the largest desktop market share, so its kernel RNG is important for many users. The Windows RNG is built on a PRNG design. It generates random numbers for the whole system from a single kernel seed. The main algorithm is the **SP800-90 AES-CTR-DRBG**.

Windows offers several interfaces, for example `SystemPrng`, `ProcessPrng`, `BCryptGenRandom`, `CryptGenRandom`, and `RtlGenRandom`. `SystemPrng` is exported by `CNG.SYS`. It is the primary interface to the per-processor kernel PRNG.

- **CryptGenRandom** — an early RNG from the Cryptographic API (CAPI). It collects entropy from hardware interrupts, network statistics, and user interactions (mouse and keyboard activity).
- **BCryptGenRandom** — a later RNG from the Cryptography API: Next Generation (CNG). It uses a wider set of entropy sources. In addition to the earlier sources, it can use the Trusted Platform Module (TPM), high-resolution performance counters, and system event logs.

{% include figure.liquid loading="eager" path="assets/img/rng/RNG_Windows.png" title="Entropy sources for CryptGenRandom and BCryptGenRandom in Windows" class="img-fluid rounded z-depth-1" %}

## <u><b>Linux</b></u>

The Linux kernel RNG provides cryptographically secure random numbers. In older 1.x kernels, it used a standard entropy pool of up to 4,096 bits. User space can obtain random data in three ways: the `getrandom(2)` system call, `/dev/random`, and `/dev/urandom`.

- **/dev/random** — for the highest-security needs. It gathers entropy from sources such as keyboard timing, mouse movement, and disk I/O. It blocks when the pool has too little entropy. Cryptographic hash functions mix the entropy before output.
- **/dev/urandom** — does not block. When entropy is low, it reuses the current internal state. It is faster and more convenient, but is generally considered less secure than `/dev/random`.

The Linux RNG has changed a lot over time:

- On 25 September 2016, kernel 4.8 moved `/dev/urandom` to a **ChaCha20**-based CPRNG.
- On 29 March 2020, kernel 5.6 removed the old entropy-pool limits. After boot, `/dev/random` and `/dev/urandom` then behave the same.
- In January 2022, kernel 5.17 changed the `/dev/random` hash function from SHA-1 to **BLAKE2s**.

For each session, the RNG chooses a unique nonce. It passes the nonce, a 256-bit secret key, and the input data to the ChaCha20 cipher block. The cipher mixes these through its rounds and produces a stream of pseudo-random bits. The RNG reseeds ChaCha20 with fresh entropy from time to time to keep the output reliable.

{% include figure.liquid loading="eager" path="assets/img/rng/RNG_chacha.png" title="ChaCha20 used for random number generation in Linux" class="img-fluid rounded z-depth-1" %}

## <u><b>macOS</b></u>

The core of the macOS RNG is a **Cryptographic Pseudo-Random Number Generator (CPRNG)**. It serves both the kernel and user-space applications. User space can reach it through the `getentropy` system call and the `/dev/random` device.

The macOS CPRNG is based on the **Fortuna** algorithm. Fortuna uses **32 entropy pools** to collect randomness from many sources, such as hardware interrupts, system events, and user input. The system reseeds these pools independently and in a staggered order. This design keeps a steady supply of entropy and stops any single source from controlling the output. It also stops the compromise of one pool from affecting the others. This is a large improvement over the earlier **Yarrow** algorithm, which relied on a single entropy pool.

The macOS CPRNG uses entropy sources such as:

- The Secure Enclave hardware TRNG.
- Timing jitter collected during boot.
- Entropy from hardware interrupts.
- A seed file that keeps entropy across reboots.
- Intel random instructions.

## <u><b>Conclusion</b></u>

The RNG designs in the three major operating systems are all sophisticated, multi-layered systems. They are built for the strict needs of cryptographic and security-critical work.

- **Windows** provides RNG through libraries such as CAPI and CNG.
- **Linux** offers three user-space interfaces (`/dev/random`, `/dev/urandom`, and `getrandom(2)`), and modern kernels have improved both throughput and entropy use.
- **macOS** uses the Fortuna algorithm with 32 entropy pools to produce high-quality random numbers.

## <u><b>References</b></u>

1. "Desktop Operating System Market Share Worldwide," StatCounter Global Stats, 2024.
2. N. Ferguson, "The Windows 10 Random Number Generation Infrastructure," 2019.
3. L. Dorrendorf, Z. Gutterman, and B. Pinkas, "Cryptanalysis of the Random Number Generator of the Windows Operating System," *ACM Trans. Inf. Syst. Secur.*, vol. 13, no. 1, Article 10, 2009.
4. S. Müller, S. Mayer, C. H. auf der Heide, and A. Hohenegger, "Documentation and Analysis of the Linux Random Number Generator," 2022.
5. "getrandom(2) — Arch Manual Pages," 2024.
6. D. J. Bernstein, "ChaCha, a Variant of Salsa20," 2008.
7. N. Ferguson, B. Schneier, and T. Kohno, "Fortuna," in *Cryptography Engineering* (2nd ed.), pp. 142–157, Wiley, 2010.
8. "Random Number Generation," Apple Platform Security, 2024.
