
Prove that for any prime number $p \geq 5$, the expression $p^2 - 1$ is divisible by 24.
## Proof

We are given that $p$ is a prime number such that $p \geq 5$.  
We want to show that:

$$
p^2 - 1 \text{ is divisible by } 24
$$

### 🔹 Step 1: Factor the expression

$$
p^2 - 1 = (p - 1)(p + 1)
$$

So we need to prove that $(p - 1)(p + 1)$ is divisible by **24**.

### 🔹 Step 2: Divisibility by 2 and 4

Since $p$ is an **odd** prime ($\geq 5$), both $p - 1$ and $p + 1$ are **even**.

Thus, their product is divisible by at least:

$$
2 \times 2 = 4
$$

### 🔹 Step 3: Divisibility by 3

Among any three consecutive integers — $p - 1$, $p$, and $p + 1$ — one must be divisible by 3.

Since $p$ is prime and $p \ne 3$, $p$ is **not** divisible by 3.  
So either $p - 1$ or $p + 1$ is divisible by 3.

Therefore:

$$
(p - 1)(p + 1) \text{ is divisible by } 3
$$

### 🔹 Step 4: Divisibility by 8

We now check divisibility by 8 using modulo:

Since $p$ is odd, $p \equiv 1, 3, 5, \text{ or } 7 \mod 8$. Check each:

- If $p \equiv 1 \mod 8$: $(p - 1)(p + 1) \equiv 0 \cdot 2 = 0 \mod 8$
- If $p \equiv 3 \mod 8$: $(p - 1)(p + 1) \equiv 2 \cdot 4 = 8 \equiv 0 \mod 8$
- If $p \equiv 5 \mod 8$: $(p - 1)(p + 1) \equiv 4 \cdot 6 = 24 \equiv 0 \mod 8$
- If $p \equiv 7 \mod 8$: $(p - 1)(p + 1) \equiv 6 \cdot 0 = 0 \mod 8$

So in **all cases**, $(p - 1)(p + 1) \equiv 0 \mod 8$

### 🔹 Step 5: Combine Results

We’ve shown that:

- $(p - 1)(p + 1)$ is divisible by **8**
- $(p - 1)(p + 1)$ is divisible by **3**

Since $\gcd(3, 8) = 1$, we conclude:

$$
(p - 1)(p + 1) \text{ is divisible by } 8 \times 3 = 24
$$

## ✅ Conclusion

For any prime number $p \geq 5$:

$$
p^2 - 1 = (p - 1)(p + 1)
$$

is divisible by **24**. ∎
