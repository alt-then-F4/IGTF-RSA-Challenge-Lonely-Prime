# **🐍 Deep Dive: Line-by-Line Code Analysis**

This document provides a granular explanation of the `solve.py` script. It breaks down every significant block of code to explain the mathematical logic behind the exploit.

## 1. Configuration Block

```python
n = 162130...  # Prime Modulus
e = 32         # Public Exponent
ct = 32275...  # Ciphertext
```
- `n`: Normally in RSA, n is the product of two primes ($p \times q$). Here, n is the prime itself. This allows us to perform calculations directly in the field $\mathbb{Z}_n$ without factoring.
- `e = 32`: This is $2^5$. In RSA, encryption is $C \equiv M^e \pmod n$. Here, it means the message $M$ was squared 5 times: $((((M^2)^2)^2)^2)^2$
- `ct:` The encrypted integer we need to reverse.
## 2. The `tonelli_shanks` Function
This function calculates $x$ such that $x^2 \equiv n\_val \pmod p$.
Function Signature
```python
def tonelli_shanks(n_val, p):
```
- **Input**: `n_val` (the number we want the square root of) and `p` (the prime modulus n).
- **Output**: A list containing the roots `[root1, root2]` or empty `[]`.
### Step A: The Legendre Symbol 
```python
if pow(n_val, (p - 1) // 2, p) != 1:
      return []
```
- **Logic**: Before calculating, we check if a square root exists.
- **Math**: Euler's criterion states that $a$ is a quadratic residue modulo $p$ only if $a^{\frac{p-1}{2}} \equiv 1 \pmod p$.
- **Result:** If the result is not `1`, it's impossible to find a root. We return an empty list, effectively pruning this branch of the solution tree.
### Step B: The "Fast Path" ($p \equiv 3 \pmod 4$)
```python
if p % 4 == 3:
    x = pow(n_val, (p + 1) // 4, p)
    return [x, p - x]
```
- **Logic**: If the prime number `p` leaves a remainder of 3 when divided by 4, there is a shortcut.
- **Math**: The square root is simply $n\_val^{\frac{p+1}{4}} \pmod p$.
- **Return**: We return `x` (the positive root) and `p - x` (the negative root).
### Step C: The Full Algorithm ($p \equiv 1 \pmod 4$)
If the prime is of the form $4k+1$, we must use the full iterative algorithm.  
**1. Factorizing $p-1$**
```python
s = p - 1
    r = 0
    while s % 2 == 0:
        s //= 2
        r += 1
```
We write $p - 1 = Q \times 2^S$ (here code uses `s` for $Q$ and `r` for $S$). We keep dividing by 2 until `s` is odd.  
**2. Finding a Non-Residue (z)**  
```python
    z = 2
    while pow(z, (p - 1) // 2, p) != p - 1:
        z += 1
```
We search for a number `z` which does not have a square root. This is needed to "balance" the equation later.  
**3. Initializing Variables** 
```python
c = pow(z, s, p)
    x = pow(n_val, (s + 1) // 2, p)
    t = pow(n_val, s, p)
    m = r
```
`x`: The current guess for the square root.  
`t`: The correction factor. We want t to become 1.  
`c`: A variable derived from our non-residue `z`.  
**4. The Correction Loop** 
```python
while t != 1:
    # ... inner loop to find 'i' ...
    # ... updating b, c, t, x ...
```
This loop repeatedly updates `x` (our root guess) and `t` (our error) until `t` becomes 1. When `t == 1`, `x` is the correct square root.  
**3. The Main Execution Block**  
**Initialization**
```python
candidates = [ct]
```
We start with a list containing only the ciphertext.  
**The "Un-squaring" Loop**  
```python
for i in range(5):
```
Since $e = 32 = 2^5$, we must perform the square root operation 5 times.  
**Generating the Next Level of the Tree**
```python
next_gen = []
    for val in candidates:
        roots = tonelli_shanks(val, n)
        next_gen.extend(roots)
```
 - We iterate through every number in our current `candidates list`.
 - For each number, we call `tonelli_shanks`.
 - We collect all returned roots (0, 1, or 2) into `next_gen`.
**Pruning Duplicates**
```python
candidates = list(set(next_gen))
```
Using `set()` removes any duplicate numbers, keeping the search efficient.
**4. Candidate Filtering**
At this stage, `candidates` contains up to 32 potential plaintexts. We need to find the one that is actually text.
```python
for m in candidates:
```
We assume `m` is the potential plaintext message as an integer.  
**Integer to Bytes Conversion**
```python
length = (m.bit_length() + 7) // 8
    msg = m.to_bytes(length, 'big')
```
- `m.bit_length()`: Counts how many bits represent the number.
- `+ 7 // 8`: Standard formula to convert bits to bytes (rounding up).
- `to_bytes`: Transforms the Python integer into a raw byte string (e.g., `b'IGTF{...}'`).
**The Flag Check**
```python
if b"IGTF{" in msg:
        print(f"Result: {msg.decode()}")
        break
```
- We check if the byte string contains the known flag prefix.
- If yes, we decode it to a string, print it, and stop the script.

## **📸 Proof of Concept**  
Here is the screenshot of the script execution showing the successful recovery of the flag:
![Script Execution](Final_Result.png)
