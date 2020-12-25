---
layout: post
title: Montgomary reduce overflow
date: 2020-12-25 10:05
tags: []
category: 
---

## Montgomary reducing

Montgomary reducing is always used in modular multiplier. A final subtract is needed to reduce output from 2P to P. There are some other tricks to avoid this subtract which are not effective in software implement.
$$
MM: R=A*B*2^{-n}\\
\begin{align}
1:& Sum(2n\ bits)=A*B \\
2:& T=\text{Montgomary_reduce}(Sum) < 2P \\
3:& R=T-P\ if\ T>P
\end{align}
$$

## Lazy Reduction

Lazy reduction is an effective method to speedup complex modular operation. 
$$
MM: R=(A*B + C*D)*2^{-n} \\
\begin{align}
1:& Sum1(2n\ bits)=A*B; Sum2(2n\ bits)=A*B;\\
2:& Sum = (Sum1 + Sum2) \mod P*2^n ;\\
3:& T=\text{Montgomary_reduce}(Sum) < 2P \\
4:& R=T-P\ if\ T>P
\end{align}
$$

In Step 2: just using higher n bit of $(Sum1+Sum2)$ to subtract P can make $Sum<P*2^n$.

_Step 3 will never overflow_
$$
Sum<P*2^n \\
\text{Montgomary_reduce}: S_{n+1} = \frac{S_n+k_i*P}{2}<\frac{S_n+P}{2}=S_n*2^{-1}+P(\frac12) \\
T = Sum*2^{-n}+P(\frac12+\frac14+\dots)<Sum*2^{-n}+P<P*2^n*2^{-n}+P=2P
$$
