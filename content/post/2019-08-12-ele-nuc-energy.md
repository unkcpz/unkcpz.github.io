---
title: "Step by step: electron-nuclear Energy in terms of density $n(r)$"
date: '2019-08-12'
categories:
    - note
tags:
    - physics 
    - dft
---

As for many-body system, the `E` generally is not a functional of n alone, only the `$E_{e-n}[n]$` is:

`$$E_{e-n}[n] = \int v(r)n(r) \mathrm{d}r^3 .$$`

Here we indicate how this functional derived.

### The definition of density

The single-particle operator of density is:

`$$\hat{n}(r) = ∑_{i=1}^{N} δ(r-r_i)$$`

For many-body system, the electron density `$n(r)$` is found
by calculating the expectation value of the single-particle density operator for the many-body wavefunction:

`$$\begin{align}
  n(r) &= ⟨Ψ|\hat{n}(r)|Ψ⟩ = ∑_{i=1}^{N} ∫ δ(r-r_i) |Ψ(r_1, r_2, …, r_N)|^2 d^3 r_1 d^3 r_2 … d^3 r_N \\
    &= ∫ |Ψ(r,r_2,…,r_N)|^2 d^3 r_2 d^3 r_3 … d^3 r_N + ∫ |Ψ(r_1,r,r_3,…,r_N)|^2 d^3 r_1 d^3 r_3 … d^3 r_N + … \\
    &= N ∫ |Ψ(r,r_2,…,r_N)|^2 d^3 r_2 d^3 r_3 … d^3 r_N
\end{align}$$`

Assuming wavefunction is nomalized to unity:

`$$∫ n(r)\mathrm{d}^3 r = N$$`

### Derivation of electron energy in ion potential

The operator of electron-nuclear interactions can be write as:

`$$\hat{V}_{n-e} = -\sum_{i}^{N_e} \sum_{I}^{N_n} \frac{Z_I}{|r_i-R_I|}$$`

then energy of electron-nuclear interactions is:

`$$\begin{align}
E_{n-e} &= ⟨Ψ(r_1,r_2,…,r_N)|\hat{V}_{n-e}|Ψ(r_1,r_2,…,r_N)⟩ \\
  &= -\sum_{i}^{N_e} \sum_{I}^{N_n} ∫ Ψ^{*}(r_1,r_2,…,r_N) \frac{Z_I}{|r_i-R_I|} Ψ(r_1,r_2,…,r_N) d^3 r_1 d^3 r_2 … d^3 r_N
\end{align}$$`

Since operator does not contain any derivatives:

`$$E_{n-e} = -\sum_{i}^{N_e} \sum_{I}^{N_n} ∫ \frac{Z_I}{|r_i-R_I|} |Ψ(r_1,r_2,…,r_N)|^2 d^3 r_1 d^3 r_2 … d^3 r_N$$`

Expend ∑ of electrons in a way that is much similar to the one we followed in the calculation of the density:

`$$\begin{align}
E_{e-n} &= -\sum_{I}^{N_n}\left[ ∫ \frac{Z_I}{|r_1-R_I|} |Ψ|^2 d^3 r_1 d^3 r_2 … d^3 r_N + … + ∫ \frac{Z_I}{|r_N-R_I|} |Ψ|^2 d^3 r_1 d^3 r_2 … d^3 r_N \right] \\
  &= -\sum_{I}^{N_n} \left[ ∫ \frac{Z_I}{|r_1-R_I|} d^3 r_1 |Ψ(r_1,r_2,…,r_N)|^2 d^3 r_2 … d^3 r_N + … \right]
\end{align}$$`

For each of electron terms in the sum, the second integral is the definition of density `$n(r)$`, therefore:

`$$\begin{align}
  E_{n-e} &= -\frac{1}{N_e} \sum_{I}^{N_n} \left[ ∫ \frac{Z_I}{|r_1-R_I|}n(r_1) + ∫ \frac{Z_I}{|r_1-R_I|}n(r_1) + … \right] \\
    &= -\sum_{I}^{N_n} ∫\frac{Z_I}{|r-R_I|}n(r) d^3 r = ∫ n(r)v_{n-e}(r) \mathrm{d}^3 r
\end{align}$$`
