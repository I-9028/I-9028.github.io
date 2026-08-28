---
layout: page
title: Analysis of Spectre-Class Side-Channel Attacks
description: A study of speculative-execution side-channel attacks, their variants, kernel mitigations, and a reproduction of a public proof-of-concept.
img:
importance: 2
category: Projects
related_publications: false
---

This project was completed for the course **CS745: Principles of Data and System Security** during my B.Tech at **IIT Bombay** (Spring 2022–23), under the guidance of **Prof. Virendra Singh**, Department of Computer Science and Engineering.

The work surveys the **Spectre** class of side-channel attacks, reviews the kernel and microcode mitigations deployed against them, and reproduces a public proof-of-concept on an unpatched configuration.

## <u><b>Introduction</b></u>

When a cryptographic algorithm runs on real hardware, it can leak information about the intermediate values it processes. This leakage can appear in power consumption, electromagnetic emanations, or timing behaviour. **Side-channel attacks (SCA)** exploit these physical leakages to recover secret information, rather than attacking the mathematics of the algorithm itself.

Such attacks are a serious threat to systems that hold secret keys, for example the Advanced Encryption Standard (AES). **Meltdown** and **Spectre** are two related side-channel attacks against modern CPUs. They can let unprivileged code read memory it should not be able to access.

## <u><b>Side-Channel Attacks</b></u>

A side-channel attack does not target program code directly. Instead, it gathers information by measuring indirect effects of the system and its hardware. Common attack vectors include:

- **Timing attack** — analyses the time a system spends on cryptographic operations.
- **Electromagnetic (EM) attack** — analyses the electromagnetic radiation emitted by a device.
- **Simple power analysis (SPA)** — observes power and EM variation during operations.
- **Differential power analysis (DPA)** — uses statistical measurements across many operations.
- **Template attack** — recovers keys by building a profile on an identical reference device.

These techniques have been used against block ciphers (DES, AES, Camellia, IDEA, MISTY1), stream ciphers (RC4, RC6), and public-key ciphers (RSA-type schemes, ECC).

### A cache example on AES

One common cache attack is **Flush+Reload**. A spy process flushes shared cache lines, lets the victim run a decryption, and then measures how long it takes to reload each line. A fast reload means the victim accessed that line. By interleaving the spy and the victim on the same core, the attacker can infer which AES table entries the victim touched, which leaks information about the key. A highly efficient variant of this cache attack on AES was demonstrated in 2016.

## <u><b>Spectre</b></u>

**Spectre** is a class of side-channel attacks that exploit **branch prediction** and **speculative execution**. Modern CPUs execute instructions ahead of time along the predicted path. These speculative operations do not commit their results to memory, but they leave measurable traces in the cache. An attacker can read those traces to infer privileged data.

Spectre was first found by **Jann Horn** of Google in July 2017, and independently by others soon after. According to the Linux kernel documentation, affected processors include Intel Core, Atom, Pentium and Xeon; AMD Phenom, EPYC and Zen; IBM POWER and zSeries; higher-end ARM and MIPS cores; and Apple CPUs.

### Main variants

- **Bounds Check Bypass (BCB)** — uses valid kernel code to read the memory of another program.
- **Branch Target Injection (BTI)** — abuses the CPU's indirect-branch prediction instead of the data path.
- **Rogue Data Cache Load (RDCL / Meltdown)** — lets a user-space program read kernel memory.
  - **Rogue System Register Read (RSRE)** — abuses system-register reads to reach kernel or other-VM data.
- **Speculative Store Bypass (SSB)** — reads data beyond the expected store order.
- **Lazy Floating-Point State Restore (LazyFP)** — exploits the lazy floating-point restore path in the kernel.
- **L1 Terminal Fault (Foreshadow)** — reads data present in the L1 cache, breaking Intel SGX enclaves and crossing virtual-machine boundaries.

### Reported CVEs

The first two variants led to three main CVE reports:

| CVE            | Description             | Variant                    |
| -------------- | ----------------------- | -------------------------- |
| CVE-2017-5753  | Bounds check bypass     | Spectre variant 1          |
| CVE-2017-5715  | Branch target injection | Spectre variant 2          |
| CVE-2019-1125  | Spectre v1 swapgs       | Spectre variant 1 (swapgs) |

### A conceptual example

A minimal illustration of the Spectre v1 pattern is the following branch:

```c
if (secret == 0) {
    x = array[0];
} else {
    x = array[1];
}
```

Here the access to `array` depends on a secret value. The CPU can speculatively perform the access even when the branch should prevent it, and the resulting cache state leaks the secret. This snippet is conceptual only; it shows the vulnerable pattern and is not a working exploit.

## <u><b>Mitigations</b></u>

Most variants were addressed by kernel updates and microcode changes. As presented at LinuxCon, the majority of kernel patches for x86 systems were released by July 2018. The Linux kernel exposes the active mitigations under `/sys/devices/system/cpu/vulnerabilities/`.

On the test machine, the reported mitigations were:

- **Spectre v1** — usercopy and swapgs barriers, with `__user` pointer sanitisation.
- **Spectre v2** — IBRS and conditional IBPB, RSB filling; the CPU was not affected by post-barrier eIBRS.

{% include figure.liquid loading="eager" path="assets/img/spectre/spectre_mitigation.png" title="Reported mitigations for Spectre v1 and v2" class="img-fluid rounded z-depth-1" %}

## <u><b>Proof of Concept</b></u>

To reproduce the attack, I used a public proof-of-concept based on the paper by Kocher, Horn, et al. The demonstration targets an unpatched Linux kernel. The test machine was an i7-9750H laptop running Ubuntu 20.04.6 LTS with kernel 5.15.0-71.

{% include figure.liquid loading="eager" path="assets/img/spectre/spectre_kernel_gcc.png" title="Kernel and GCC versions on the test system" class="img-fluid rounded z-depth-1" %}

The proof-of-concept reads a secret `char[]` array from a separate memory region. The run is considered successful when the recovered bytes match the target secret string.

{% include figure.liquid loading="eager" path="assets/img/spectre/spectre_poc_test1.png" title="Proof-of-concept, test run 1" class="img-fluid rounded z-depth-1" %}

{% include figure.liquid loading="eager" path="assets/img/spectre/spectre_poc_test2.png" title="Proof-of-concept, test run 2" class="img-fluid rounded z-depth-1" %}

The recovered bytes match the expected secret string, which shows that a branch-predictor-based read still succeeds on this configuration.

## <u><b>Conclusion</b></u>

This project surveyed side-channel attacks and examined the Spectre class in detail. The key finding is that the effectiveness of an attack depends on the mitigations in place, and that new speculative-execution vulnerabilities continue to appear. As recent as February 2023, a new Spectre v2 issue was reported in kernel 6.2, with a working proof-of-concept, and it was patched by 10 March 2023.

## <u><b>References</b></u>

1. P. C. Kocher, "Timing Attacks on Implementations of Diffie-Hellman, RSA, DSS, and Other Systems," *CRYPTO '96*, LNCS 1109, pp. 104–113, Springer, 1996.
2. Y. Zhou and D. Feng, "Side-Channel Attacks: Ten Years After Its Publication and the Impacts on Cryptographic Module Security Testing," 2005.
3. B. Menezes, "Side Channel Attacks on AES and DSA," 2022.
4. C. Ashokkumar, R. P. Giri, and B. Menezes, "Highly Efficient Algorithms for AES Key Retrieval in Cache Access Attacks," *IEEE EuroS&P*, pp. 261–275, 2016.
5. G. Kroah-Hartman, "Presentation: Spectre," 2019.
6. CVE-2018-3665, CVE Record, May 2018.
7. P. Kocher, J. Horn, A. Fogh, D. Genkin, D. Gruss, W. Haas, et al., "Spectre Attacks: Exploiting Speculative Execution," *IEEE S&P*, pp. 1–19, 2019.
8. Semihalf, "Spectre-Based Meltdown Attack (proof-of-concept)," 2018.
9. Google, "Linux Kernel: Spectre v2 SMT Mitigations Problem," 2023.
