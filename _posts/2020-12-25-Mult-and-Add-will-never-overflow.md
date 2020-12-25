---
layout: post
title: Mult and Add will never overflow
date: 2020-12-25 08:47
tags: []
category:
---

When I implemented SM9, I used `mac64` to do modular operation.

```rust
pub const fn mac64(a: u64, b: u64, c: u64, carry: u64) -> (u64, u64) {
    let ret = (a as u128) + ((b as u128) * (c as u128)) + (carry as u128);
    (ret as u64, (ret >> 64) as u64)
}
```

_This function will never overflow_

$$
\text{Assume the maximum of input} \\
\begin{align}
& a+(b*c)+carry \\
\le & 2^n-1 + (2^n-1)*(2^n-1) + 2^n-1 \\
=& 2^{2n} - 2* 2^n + 1 + 2^n-1 + 2^n-1 \\
=& 2^{2n}-1
\end{align}
$$
