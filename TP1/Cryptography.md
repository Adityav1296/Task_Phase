## Hill Cipher
Hill cipher is a classical polygraphic substitution cipher that uses linear algebra. It encrypts blocks of n letters at a time by treating each block as an n×1 vector of numbers (A=0, B=1, …, Z=25) and multiplying it by an n×n key matrix modulo 26.
Encryption: C = K · P (mod 26)
Decryption: P = K⁻¹ · C (mod 26), where K⁻¹ is the inverse of K modulo 26.

Key points:

1. Key must be an invertible matrix modulo 26. That requires gcd(det(K), 26) = 1.
2. Block size n is the dimension of the key matrix (common choices: 2 or 3).
3. Plaintext is split into length-n blocks; pad the last block if needed (e.g., with 'X').

### Python Code

```python
import numpy as np
import itertools

# --- Helper Functions ---
def text_to_nums(text):
    return [ord(c) - ord('A') for c in text.upper() if c.isalpha()]

def nums_to_text(nums):
    return ''.join(chr(n + ord('A')) for n in nums)

def mod_inverse(a, m):
    for x in range(1, m):
        if (a * x) % m == 1:
            return x
    return None

def matrix_inverse_mod26(matrix):
    det = int(round(np.linalg.det(matrix))) % 26
    det_inv = mod_inverse(det, 26)
    if det_inv is None:   # no inverse exists
        return None
    adj = np.round(det * np.linalg.inv(matrix)).astype(int) % 26
    return (det_inv * adj) % 26

# --- Encryption ---
def encrypt(plaintext, key):
    nums = text_to_nums(plaintext)
    while len(nums) % 2 != 0:  # pad
        nums.append(ord('X') - ord('A'))
    cipher = []
    for i in range(0, len(nums), 2):
        block = np.array(nums[i:i+2])
        enc = np.dot(key, block) % 26
        cipher.extend(enc)
    return nums_to_text(cipher)

# --- Decryption ---
def decrypt(ciphertext, key):
    nums = text_to_nums(ciphertext)
    key_inv = matrix_inverse_mod26(key)
    if key_inv is None:
        return None
    plain = []
    for i in range(0, len(nums), 2):
        block = np.array(nums[i:i+2])
        dec = np.dot(key_inv, block) % 26
        plain.extend(dec)
    return nums_to_text(plain)

# --- Brute Force (try all invertible 2x2 matrices) ---
def brute_force(ciphertext):
    for a, b, c, d in itertools.product(range(26), repeat=4):
        key = np.array([[a, b], [c, d]])
        key_inv = matrix_inverse_mod26(key)
        if key_inv is not None:  # only valid keys
            pt = decrypt(ciphertext, key)
            if pt is not None: 
                print(f"Key={key.tolist()} => Plaintext={pt}")

# --- Example ---
key = np.array([[3, 3], [2, 5]])
plaintext = "HELLO"
cipher = encrypt(plaintext, key)
decrypted = decrypt(cipher, key)

print("Plaintext :", plaintext)
print("Ciphertext:", cipher)
print("Decrypted :", decrypted)

print("\nBrute Force Guess")
brute_force(cipher)
```

### Working 
1. Encryption:
   1. Break plaintext into blocks of 2 letters.
   2. Multiply with key matrix mod 26.
   3. Convert numbers → ciphertext.

2. Decryption:
   1. Compute inverse of key matrix mod 26.
   2. Multiply ciphertext blocks with inverse.
   3. Convert → plaintext.

3. Brute Force:
   1. Try all 26⁴ = 456,976 possible 2×2 matrices.
   2. Skip those without inverse mod 26.
   3. Decrypt and print possible results.



## Spiral Cipher
Spiral cipher (a type of route cipher) places characters into a rectangular grid and reads them out by following a spiral path. It’s not substitution — it’s a reordering (permutation) of characters.

Two common variants (you must choose one and be consistent):
  1. Write row-wise, read spiral.
     Fill the grid left→right, top→bottom with the plaintext, then read characters by walking a spiral (e.g., clockwise from top-left inward).
  2. Write spiral, read row-wise.
     Fill the grid following a spiral path with the plaintext, then produce ciphertext by reading the grid row-by-row.

### Parameters to follow:
1. Grid size: number of rows (R) and columns (C). Must satisfy R * C >= len(plaintext).
2. Padding character (e.g., 'X') to fill unused cells.
3. Spiral direction: clockwise or counterclockwise.
4. Spiral start corner: top-left, top-right, bottom-left, bottom-right.
5. Spiral orientation: inward (usual: start outer layer, go inward) or outward (less common).

### Challenge Quest:
Cipher: taskphaWL_PL4sOingpYefdngaP{_diddL40ap}y5rn_s1m37

### Solutions:
1. Assuming that the text was encrypted using the first method where we write the text row-wise and read spiral. Backtracking through the process and writing the cipher text as spiral and then reading it row wise the plaintext comes out to be:
taskphaWL_PL4sOingpYefdngaP{_diddL40ap}y5rn_s1m37

2. Now trying the other method where we write the plaintext in spiral and then read it row-wise when encrypting. Backtracking through this method, plaintext obtained:
taskphase{4r73m1s_n0_fOWL_PL4YPL5y}paddingpadding



## Frequecy Analysis
Frequency analysis is a cryptanalysis technique based on the observation that letters (or symbols) in a language appear with characteristic frequencies.

For example, in English:
   - ‘E’ is the most common letter,
   - followed by ‘T’, ‘A’, ‘O’, ‘I’, ‘N’, etc.

If a cipher replaces each letter with another symbol (like in a Caesar cipher, monoalphabetic substitution, or even emojis), the frequency of those symbols will still roughly match the pattern of the original letters.

### Working:
1. Count symbol occurrences: 
   You calculate how often each symbol appears in the ciphertext.
2. Compare with known letter frequencies:
   You compare those frequencies with the typical English letter frequencies (like E ≈ 13%, T ≈ 9%, etc.).
3. Guess replacements:
   The most common symbol probably stands for ‘E’.
   The second most common might stand for ‘T’ and so on.
   You test these hypotheses to gradually recover the plaintext.
4. Refine guesses using context:
   Words, letter patterns, and grammar clues help fix wrong guesses — e.g., a one-letter word must be ‘A’ or ‘I’.

### Challenge Quest:
cipher: book.txt - can you help my find the name of this book and it’s writer?

### Solution:
**Book**: Ulysses
**Author**: James Joyce
