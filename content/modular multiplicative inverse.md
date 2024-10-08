---
id: 1725555801-modular-multiplicative-inverse
aliases:
  - modular multiplicative inverse
tags: []
---

# Modular multiplicative inverse
1. Definition:
   For two numbers 'a' and 'm', the modular multiplicative inverse of 'a' modulo 'm' is a number 'b' such that:
   
   (a * b) % m = 1

   In other words, 'b' is the number that when multiplied by 'a', gives a remainder of 1 when divided by 'm'.

2. Existence:
   The modular multiplicative inverse of 'a' modulo 'm' exists if and only if 'a' and 'm' are coprime (their greatest common divisor is 1).

3. Usage in [[RSA]]:
   In the context of RSA, we use the modular multiplicative inverse to calculate the private key 'd' given the public exponent 'e' and the totient φ(n). We need to find 'd' such that:
   
   (e * d) % φ(n) = 1

4. Calculation:
   The most common way to calculate the modular multiplicative inverse is using the [[extended Euclidean algorithm]].

## Rust Implementation 1
```rust
fn mod_inverse(a: &BigUint, m: &BigUint) -> Option<BigUint> {
    let a = a.to_bigint().unwrap();
    let m = m.to_bigint().unwrap();
    let (mut t, mut newt) = (BigInt::zero(), BigInt::one());
    let (mut r, mut newr) = (m.clone(), a);

    while !newr.is_zero() {
        let quotient = &r / &newr;
        (t, newt) = (newt.clone(), t - &quotient * &newt);
        (r, newr) = (newr.clone(), r - &quotient * &newr);
    }

    if r > BigInt::one() {
        None
    } else {
        while t < BigInt::zero() {
            t += &m;
        }
        Some((t % &m).to_biguint().unwrap())
    }
}
```
### Notes
This Rust function implements the [[Extended Euclidean Algorithm]] to find the modular multiplicative inverse of a number. Here's a step-by-step explanation:

1. The function takes two `BigUint` (big unsigned integer) references as input: `a` and `m`.
2. It converts both inputs to `BigInt` (big signed integer) for the algorithm.
3. It initializes variables for the algorithm:
   - `t` and `newt` for tracking coefficients
   - `r` and `newr` for tracking remainders
4. The main loop implements the Extended Euclidean Algorithm:
   - It calculates the quotient and updates `t`, `newt`, `r`, and `newr` in each iteration
   - The loop continues until `newr` becomes zero
5. After the loop, it checks if a modular inverse exists:
   - If `r > 1`, no inverse exists, so it returns `None`
   - Otherwise, it adjusts `t` to be positive and returns `t % m` as the inverse

This algorithm is used in various cryptographic applications, including [[RSA|RSA encryption]].

## Rust Implementation 2
```rust
/// Computes the modular multiplicative inverse of 'a' modulo 'm' using the extended Euclidean algorithm.
///
/// # Arguments
/// * `a` - The number to find the inverse for
/// * `m` - The modulus
///
/// # Returns
/// The modular multiplicative inverse of 'a' modulo 'm' as a BigInt
///
/// # Panics
/// Panics if 'a' is not invertible modulo 'm'
fn mod_inverse(a: &BigInt, m: &BigInt) -> BigInt {
    let mut t = BigInt::zero();
    let mut new_t = BigInt::one();
    let mut r = m.clone();
    let mut new_r = a.clone();

    while new_r != BigInt::zero() {
        let quotient = &r / &new_r;
        t -= &quotient * &new_t;
        std::mem::swap(&mut t, &mut new_t);
        r -= &quotient * &new_r;
        std::mem::swap(&mut r, &mut new_r);
    }

    if r > BigInt::one() {
        panic!("a is not invertible");
    }
    if t < BigInt::zero() {
        t += m;
    }
    t
}
```

### Notes
1. Function signature:
   ```rust
   fn mod_inverse(a: &BigInt, m: &BigInt) -> BigInt
   ```
   This function takes two references to `BigInt` values (`a` and `m`) and returns a `BigInt`.

2. Initialization:
   ```rust
   let mut t = BigInt::zero();
   let mut new_t = BigInt::one();
   let mut r = m.clone();
   let mut new_r = a.clone();
   ```
   - `t` and `new_t` are used to keep track of coefficients in the extended Euclidean algorithm.
   - `r` and `new_r` are used to store remainders.

3. Main loop:
   ```rust
   while new_r != BigInt::zero() {
       let quotient = &r / &new_r;
       t -= &quotient * &new_t;
       std::mem::swap(&mut t, &mut new_t);
       r -= &quotient * &new_r;
       std::mem::swap(&mut r, &mut new_r);
   }
   ```
   This loop implements the extended Euclidean algorithm:
   - Calculate the quotient of `r` divided by `new_r`.
   - Update `t` and `r` using the quotient.
   - Swap `t` with `new_t` and `r` with `new_r` to prepare for the next iteration.

4. Check for invertibility:
   ```rust
   if r > BigInt::one() {
       panic!("a is not invertible");
   }
   ```
   If the final remainder is greater than 1, it means `a` is not invertible modulo `m`.

5. Ensure positive result:
   ```rust
   if t < BigInt::zero() {
       t += m;
   }
   ```
   If `t` is negative, add `m` to make it positive while maintaining the same result modulo `m`.

6. Return the result:
   ```rust
   t
   ```
   The final value of `t` is the modular inverse of `a` modulo `m`.
