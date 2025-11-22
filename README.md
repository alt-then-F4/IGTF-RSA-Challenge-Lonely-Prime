# 🔐 RSA Single Prime Challenge

## 📋 Information

| Category | Difficulty | Flag Format | Vulnerability |
| :--- | :--- | :--- | :--- |
| **Cryptography** | 🟡 Medium | `IGTF{...}` | Single Prime & Even Exponent |

## ☎️ Problem Statement

My crypto professor said that RSA requires two prime numbers. I decided to save energy and use just one, but a very large one, so it's super tough! I also doubled the security of the classic exponent by bumping it up to 32.

Can you recover my secret?

**Provided files:** `challenge_data.txt` (containing $N$, $e$, and $ct$).

## 📝 The Challenge

The author of this challenge wanted to "optimize" RSA encryption by committing two critical implementation errors:
1.  Using a **single prime number** $N$ instead of two ($N = p$ instead of $N = p \times q$).
2.  Using an **even exponent** $e=32$ (which is $2^5$).

**Goal:** Recover the original message (the flag) without knowing the private key (which mathematically doesn't exist in this configuration).

### 📂 Provided File

* `challenge_data.txt`: The raw values of $N$, $e$, and $ct$.

---

## 🛠️ Analysis and Theoretical Solution

To understand how to break this encryption, we must analyze why standard RSA rules do not apply.

### 1. The "Playground": Why Prime $N$ Changes Everything

> **The Hidden Room Analogy** 🏠
> * **Normal RSA ($N = p \times q$):** The "room" is a complex labyrinth built by mixing two architectural blueprints ($p$ and $q$). If you don't have the blueprints (factorization), you are lost.
> * **This Challenge ($N = p$):** The author forgot to mix them. The "room" is a large open space (a finite field $\mathbb{Z}_p$).

**Mathematical Consequence:**
In this open space, the rules are simple. Computing square roots (which is very difficult in normal RSA without the key) becomes possible and efficient thanks to algorithms like **Tonelli-Shanks**.

### 2. The "Revolving Door" Problem ($e=32$)

Why can't we simply calculate the inverse key ($d$) as usual?

> **The Door Analogy** 🚪
> * **Encrypt ($e$):** You push a revolving door $X$ times.
> * **Decrypt ($d$):** You push it more to get back exactly to the starting point.
> * **The Bug:** Here, the rotation size is even (since $N-1$ is even) and your push ($e=32$) is even. It's as if the door made full rotations. Looking at the closed door, it's impossible to know if it made 2, 4, or 32 turns. The information has overlapped.

**Consequence:** There is no unique "back button". The function is not bijective.

### 3. The Resolution: Climbing the Tree

Since we cannot reverse the calculation all at once, we must do it step by step.
The encryption equation is:
$$C \equiv M^{32} \pmod N$$

Since $32 = 2^5$, this means the message was squared **5 times in a row**.
To recover the message, we must extract the square root 5 times in a row.



[Image of binary tree diagram]


**The Root Trap:**
In modular arithmetic, a square root often has **two solutions** ($x$ and $-x$). This creates a decision tree:

* **Start:** 1 ciphertext.
* **Step 1 ($\sqrt{}$):** 2 possibilities.
* **Step 2 ($\sqrt{}$):** 4 possibilities.
* ...
* **Step 5 ($\sqrt{}$):** Up to 32 potential messages.

One of these 32 messages is the flag `IGTF{...}`. The others are just mathematical noise.

---

## 💻 Script Usage

The `solve.py` script automates this attack using the Tonelli-Shanks algorithm.

### How the script works
1.  It loads the ciphertext $ct$.
2.  It loops 5 times to extract successive square roots.
3.  It manages the decision tree (multiple candidates generated at each step).
4.  It converts the results to text and searches for the flag format `IGTF{`.

More details can be found in the `solve` directory on GitHub.

### Prerequisites
No complex installation is required; the script uses native Python (3.x).

### Command
Open a terminal in the folder and run:

```bash
python solve.py
