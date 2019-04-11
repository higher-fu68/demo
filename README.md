# Homework 4: AES

- Author: [Yifan Cao](mailto:caoyf@shanghaitech.edu.cn)
- **Due: 23:59:59, April 18 (Thursday), 2019 CST**
- Last updated: March 30, 2019

# Get Started

To get started, please simply fork this GitLab repository and follow the structure and submissions guidelines below.

**Remember to change your repo to a private one before you commit to it.**

# Repository Structure

### README.md
Problem description and requirements.

### utils.py
Some constants and functions which are useful in this homework.

### aes.py
A basic template for this homework.

# Submission

**You should check in `aes.py` and `utils.py` to GitLab.**

First, make a commit and push your files. From within the root folder of this repo, run
```sh
git add aes.py utils.py
git commit -m '{your commit message}'
git push
```
Then add a tag to create a submission.
```sh
git tag {tagname} && git push origin {tagname}
```
You need to define your own submission tag by changing `{tagname}`, e.g.
```sh
git tag first_trial && git push origin first_trial
```
**Please use a new tag for every new submission.**

Every submission will create a new GitLab issue, where you can track the progress.

# Regulations

- No late submissions will be accepted.
- You have 30 chances of grading (i.e. `git tag`) in this homework. If you hand in more 30 times, each extra submission will lead to 10% deduction. In addition, you are able to require grading at most 10 times every 24 hours.
- We enforce academic integrity strictly. If you participate in any form of cheating, you will fail this course immediately. You can view the full edition on [Piazza](https://piazza.com/class/jrykv8wi15m5dx?cid=7).
- If you have any questions about this homework, please ask on Piazza first.

# Hints

- You may *not* use third-party libraries. Standard libraries (e.g. `random`, `copy`, `sys`, etc.) are allowed, but you may *not* use the AES module in the standard library if exists (actually, it doesn't). The reference program only uses `random` (for generating keys).
- You may *not* run external programs by taking advantage of modules like `os`, `subprocess`, etc.
- The main work will be quite a lot bitwise operations. You will need `&` (bitwise AND), `|` (bitwise OR), `^` (bitwise XOR), `<<` (left shift), `>>` (right shift).
- Feel free to define new functions, but you may *not* rename the names of required classes and functions.
- Cipher programs are carefully designed and are usually very sensitive to a small change somewhere. Test your program locally before submission.
- You may find the pre-defined constants and functions in `utils.py` useful. In addition, we will use your copy of `utils.py`, so you can add your own functions in it and use them in `aes.py`.
- You may need some background on cryptography. There is a [video](https://coolshell.cn/wp-content/uploads/2010/10/rijndael_ingles2004.swf) (also on [Youtube](https://www.youtube.com/watch?v=mlzxpkdXP58)) for you to understand the model better.

### `test` Functions

The `test` functions is given to make the description more clear and help you checking your code with several [`assert` statements](https://docs.python.org/3/reference/simple_stmts.html#assert). If you pass the test, nothing happens, otherwise it will raise an `AssertionError`. If you can't understand this function, don't worry, just focus on which statement raise the `AssertionError` then check the corresponding function called in that statement.

Feel free to add some test code and `print` statements to this function, or even define some new test functions. This will not affect your score since `test` functions will not be called in the judger, as long as you don't run them, or just run them under `if __name__ == '__main__'`.

# Description

## Introduction to Private-Key Ciphers

It is a common issue that two parties need to communicate secretly on an insecure channel, which may be monitored by an eavesdropper. A solution is to use a *cipher*, which is generally an encryption scheme. Security of many classical ciphers relies on a secret *key*, which is shared by the communicating parties in advance, and unknown to the eavesdropper. This scenario is known as the *private-key* setting.

Formally, a private-key encryption scheme is defined by specifying three spaces:
- the *key space* $`\mathcal{K}`$,
- the *message space* $`\mathcal{M}`$, and
- the *ciphertext space* $`\mathcal{C}`$,

along with three algorithms:
- The *key-generation algorithm* $`\mathsf{Gen}`$ is a probabilistic algorithm that outputs a key $`k \in \mathcal{K}`$ chosen according to some distribution.
- The *encryption algorithm* $`\mathsf{Enc}`$ takes as input a key $`k \in \mathcal{K}`$ and a message $`m \in \mathcal{M}`$ and outputs a ciphertext $`c \in \mathcal{C}`$. We denote this by $`c = \mathsf{Enc}_k(m)`$.
- The *decryption algorithm* $`\mathsf{Dec}`$ takes as input a key $`k \in \mathcal{K}`$ and a ciphertext $`c \in \mathcal{C}`$ and outputs a plaintext $`m \in \mathcal{m}`$. We denote this by $`m = \mathsf{Dec}_k(c)`$.

An encryption scheme must satisfy the *correctness* requirement - encrypting a message and then decrypting the resulting ciphertext (using the same key) yields the original message. In other words, for every key $`k`$ output by $`\mathsf{Gen}`$ and every message $`m \in \mathcal{M}`$, it holds that $`\mathsf{Dec}_k(\mathsf{Enc}_k(m)) = m`$.

Such an encryption scheme can be used by two parties, say Alice and Bob, who wish to communicate securely as follows.
- First, $`\mathsf{Gen}`$ is run to obtain a key $`k`$ that Alice and Bob share in advance.
- Later, Alice wants to send a plaintext $`m`$ to Bob. She computes $`c := \mathsf{Enc}_k(m)`$ and sends the resulting ciphertext $`c`$ over the public channel to Bob.
- Upon receiving $`c`$, Bob computes $`m := \mathsf{Dec}_k(c)`$ to recover the original plaintext.

## Task 1: AES

### Requirements

A *block cipher* is a kind of private-key cipher. Given the private key and input, the encryption and decryption algorithm is deterministic - encrypting the same plaintext always yields the same ciphertext, and vice versa. In addition, the input is a fixed-length group of bits called a *block*, and the algorithm does an unvarying transformation specified by the private key to get the output.

The *Advanced Encryption Standard* (AES) is a kind of block cipher, adopted and standardized by the U.S. government. To form the standard, three members are selected from the Rijndael block cipher, each with a same block size of 128 bits, but three different key lengths: 128, 192 and 256 bits. These are also known as AES-128, AES-192 and AES-256.

Implement AES block cipher in Python, which supports all the three key lengths.

### I/O

In the constructor, the input is the key length (128, 192 or 256 bits), and optionally the cipher key with that length, encoded in [big endian](https://en.wikipedia.org/wiki/Endianness#Big-endian). If the cipher key is not specified, it is randomly generated.

The input is the 128-bit plaintext (for encryption) or the ciphertext (for decryption), encoded in big endian.

The output is the 128-bit ciphertext (for encryption) or the plaintext (for decryption), encoded in big endian.

Read the docstring in the code for more details.

### Exceptions

Raise `AESInputError` on any invalid inputs in the arguments.

Read the docstring in the code for more details.

### API

Your program must provide the following API.

```python
class AES:
    def __init__(self, keylen, key=None):
        '''
        Input:
        - `keylen`: int type, the key length, with value 128, 192 or 256.
        - `key`: int type, the `keylen`-bit cipher key, encoded in big endian;
          or None, generate the key randomly (None by default).

        Store the `key`, which is a cipher key provided by the user, as an
        attribute. If `key` is None, generate the cipher key randomly according
        to the length `keylen` with `random.randrange()`.

        Raise `AESInputError` if and only if either
        - `keylen` is not an int, or is a value other than 128, 192 and 256, or
        - `key` is not an int, is negative, or cannot fit in 128 bits.
        '''

    def enc(self, msg):
        '''
        Input: `msg`: int type, the 128-bit plaintext to be encrypted,
        encoded in big endian.
        Output: int type, the 128-bit ciphertext encoded in big endian.

        Encrypt the plaintext `msg` with AES algorithm, given the key provided
        or generated before. Return the ciphertext.

        Raise `AESInputError` if and only if `msg` is not an int, is negative,
        or cannot fit in 128 bits.
        '''

    def dec(self, ciph):
        '''
        Input: `msg`: int type, the 128-bit ciphertext to be decrypted,
        encoded in big endian.
        Output: int type, the 128-bit plaintext encoded in big endian.

        Decrypt the plaintext `ciph` with AES algorithm, given the key provided
        or generated before. Return the plaintext.

        Raise `AESInputError` if and only if `ciph` is not an int, is negative,
        or cannot fit in 128 bits.
        '''

    def getkey(self):
        ''' Output: int type, the cipher key encoded in big endian. '''
```

### Hints

- You will need to read about 15 pages of the standard document and understand how AES works. Be patient.
- Check the appendices of the standard document. Those are examples of the algorithm with the details of every step, which could be helpful for debugging.
- Use `hex()` to convert an integer to the corresponding hexadecimal string.

### Implementation Details

**You will need to read the standard document [FIPS 197](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf).** Since it is quite long, a guide is organized below.

#### The State (Sec. 3.4, 3.5)

Internally, the AES algorithm's operations are performed on a two-dimensional array ($`4 \times Nb`$) of bytes called the *State*. The State consists of 4 rows of bytes, each containing $`Nb`$ bytes, where $`Nb`$ is the block length (in bits) divided by 32. **In AES, the block length is 128 bits and $`Nb = 4`$, so the State is a $`4 \times 4`$ array.**

In the State array denoted by the symbol $`s`$, each individual byte has two indices, with its row number $`r`$ in the range $`0 \le r < 4`$ and its column number $`c`$ in the range $`0 \le c < Nb`$ (remember that $`Nb = 4`$). This allows an individual byte of the State to be referred to as either $`s_{r,c}`$ or $`s[r,c]`$.

At the start of the cipher, the input (128 bits, 16 bytes, 1 dimension) is copied into the State array in *column-major order*. In other words, $`s[r,c] = in[r+4c]`$ for any $`r`$ and $`c`$. The algorithm then applies transformations on this State array for several times (this can be called state transitions).

In the end, the State array can look very different from the original one. The State is then copied to the output array (128 bits, 16 bytes, 1 dimension) in column major order. In other words, $`out[r+4c] = s[r,c]`$ for any $`r`$ and $`c`$.

If the input is a plaintext and the algorithm is for encryption, the output is then the ciphertext, and vice versa.

There are 4 bytes in each column of a State array. They form a 32-bit *word*, and we could use a word to represent a column. The $`c`$'th column, or the $`c`$'th word, is represented as $`w_c = s_{0,c}s_{1,c}s_{2,c}s_{3,c}`$.

#### Finite Field Arithmetic (Sec. 4)

Unlike the four arithmetic operation we know, the AES algorithm is operating on elements in the so-called *finite fields*. An element is a byte which has only 8 bits. Without furthur ado, we use *byte* to represent an element in a finite field below.

We could represent a byte with 8 bits (0 or 1). The rightmost bit is called the *least significant bit* (LSB), represented by $`b_0`$. The leftmost one is called the *most significant bit* (MSB), represented by $`b_7`$. The order is $`\{b_7,b_6,b_5,b_4,b_3,b_2,b_1,b_0\}`$.

The byte can also be interpreted by a polynomial $`\sum\limits_{i=0}^7 b_ix^i`$. For example, $`\{01100011\}`$ identifies $`x^6 + x^5 + x + 1`$.

It is also convenient to denote byte values using hexadecimal notation with each of 2 groups of 4 bits being denoted by a single character. For example, $`\{01100011\}`$ can be represented as $`\{63\}`$, because $`0110_{16} = 6_{2}`$ and $`0011_{16} = 3_{2}`$.

##### Addition (Sec. 4.1)

The addition of two bytes is achieved by "adding" the corresponding bits in the byte. The addition is performed with the *XOR operation* (denoted by $`\oplus`$) - i.e., modulo 2 addition - so that $`1 \oplus 1 = 0`$, $`1 \oplus 0 = 1`$, and $`0 \oplus 0 = 0`$. Consequently, subtraction of polynomials is identical to addition of polynomials.

For example, $`\{01010111\} \oplus \{10000011\} = \{11010100\}`$ in binary, and $`\{57\} \oplus \{83\} = \{d4\}`$ in hexadecimal.

Note: In Python, **`^` is for XORing, e.g. `a ^ b`**.

##### Multiplication (Sec. 4.2)

**Note: This part is quite hard to understand. We have implemented `byte_mul(a, b)` for you in `utils.py`, and feel free to use it directly. You may continue reading the rest of this part if interested or simply skip it.**

The multiplication (denoted by $`\bullet`$) of two bytes is not as simple at byte level as addition. It is done by the multiplication of polynomials modulo an irreducible polynomial of degree 8. For the AES algorithm, this polynomial is $`m(x) = x^8 + x^4 + x^3 + x + 1`$, or $`\{01\}\{1b\}`$ in hexadecimal.

For example, $`\{57\} \bullet \{83\} = \{c1\}`$, because
```math
(x^6 + x^4 + x^2 + x + 1)(x^7 + x + 1) = x^{13} + x^{11} + x^9 + x^8 + x^6 + x^5 + x^4 + x^3 + 1,
```
and
```math
(x^{13} + x^{11} + x^9 + x^8 + x^6 + x^5 + x^4 + x^3 + 1) \bmod (x^8 + x^4 + x^3 + x + 1) = x^7 + x^6 + 1.
```
Note that all additions and subtractions in the operations above are done by XOR. The modular reduction by $`m(x)`$ ensures that the result will be a binary polynomial of degree less than 8, and thus can be represented by a byte.

The procedure above is not easy to implement in a small piece of code. However, there is a much simpler solution that the reference program also uses.

Consider multiplying a polynomial $`b_7x^7 + \dots + b_0`$ with $`x`$. The result then becomes $`b_7x^8 + b_6x^7 + \dots + b_0x`$. If $`b_7 = 0`$, we are done, since the degree is less than 8 and reduction does not take any effects. Otherwise $`b_7 = 1`$, and the reduction is accomplished by subtracting (i.e. XORing) the polynomial $`m(x)`$.

To sum up, multiplication by $`x`$ can be implemented at the byte level as a left shift and a subsequent conditional bitwise XOR with $`\{1b\}`$ (note that $`m(x) = \{01\}\{1b\}`$).
```math
x \bullet b(x) =
\begin{cases}
b(x) \ll 1, & b_7 = 0, \\
(b(x) \ll 1) \oplus \{1b\}, & b_7 = 1.
\end{cases}
```
Note: Python supports arbitrary length of integers, so left shifting does not automatically drop the MSB. **XOR with `0x11b` to drop that manually.**

The multiplication holds that $`a(x) \bullet (b(x) + c(x)) = a(x) \bullet b(x) + a(x) \bullet c(x)`$. Thus, to implement the multiplication $`a(x) \bullet b(x)`$, simply split $`b(x)`$ to exponents of $`x`$.

For example, $`\{57\} \bullet \{13\} = \{fe\}`$ because:
- $`\{57\} \bullet \{01\} = \{57\}`$,
- $`\{57\} \bullet \{02\} = \{ae\}`$ (multiply by $`x`$),
- $`\{57\} \bullet \{04\} = \{47\}`$ (multiply again),
- $`\{57\} \bullet \{08\} = \{8e\}`$ (multiply again),
- $`\{57\} \bullet \{10\} = \{07\}`$ (multiply again),
- Thus $`\{57\} \bullet \{13\} = \{57\} \bullet (\{01\} \oplus \{02\} \oplus \{10\}) = \{57\} \oplus \{ae\} \oplus \{07\} = \{fe\}`$.

#### Algorithm Specification (Sec. 5)

For the AES algorithm, **the length of the input block, the output block and the State is 128 bits**. This is represented by $`Nb = 4`$, which reflects the number of 32-bit words (number of columns) in the State.

For the AES algorithm, **the length of the Cipher Key, K, is 128, 192, or 256 bits**. The key length is represented by $`Nk = 4, 6, 8`$, which reflects the number of 32-bit words (number of columns) in the Cipher Key.

For the AES algorithm, the number of rounds to be performed (on the State array) during the execution of the algorithm is dependent on the key size. The number of rounds is represented by $`Nr`$, where $`Nr = 10`$ when $`Nk = 4`$, $`Nr = 12`$ when $`Nk = 6`$, and $`Nr = 14`$ when $`Nk = 8`$.

**The only Key-Block-Round combinations that conform to this standard are given in Figure 4 (Page 14).**

#### Key Expansion (Key Schedule) (Sec. 5.2)

**Note: Follow the pseudo code in Figure 11, Page 20. Run this procedure in the end of the constructor of `AES`.**

The AES algorithm does not apply the Cipher Key to the State array directly. Instead, it takes the Cipher Key $`K`$ and performs a Key Expansion. The key is then expanded to a total of $`Nb(Nr + 1)`$ words, which will then be applied to the State array. In Figure 5 (Page 15), the algorithm requires an initial set of $`Nb`$ words, and each of the $`Nr`$ rounds requires $`Nb`$ words of key data.

The resulting key schedule consists of a linear array of 4-byte words, denoted $`[w_i]`$, with $`i`$ in the range $`0 \le i < Nb(Nr + 1)`$.
```math
w[i] =
\begin{cases}
K[4i, \dots, 4i+3], & i < Nk, \\
w[i-Nk] \oplus \mathtt{SubWord}(\mathtt{RotWord}(w[i-1])) \oplus \mathtt{Rcon}[i/Nk], & i \ge Nk \text{ and } i \equiv 0 \pmod{Nk}, \\
w[i-Nk] \oplus \mathtt{SubWord}(w[i-1]), & i \ge Nk > 6 \text{ and } i \equiv 4 \pmod{Nk}, \\
w[i-Nk] \oplus w[i-1], & \text{otherwise}.
\end{cases}
```

The following concepts are involved:
- `SubWord()`: Takes a 4-byte input word, applies S-box (Figure 7, Page 16) to each of the 4 bytes to produce an output word.
- `RotWord()`: Takes a 4-byte word $`[a_0,a_1,a_2,a_3]`$ as input, performs a cyclic permutation, and returns the word $`[a_1,a_2,a_3,a_0]`$.
- `Rcon[i]`: The round constant word array, which contains the values given by $`[x^{i-1}, \{00\}, \{00\}, \{00\}]`$. Note that this array starts at 1, not 0. **In `utils.py`, we have already evaluated all the possibly useful exponent values for you. Check the tuple `RCON_VAL`.**

It can be seen that the first $`Nk`$ words of the expanded key are filled with the
Cipher Key. Every following word, $`w[i]`$, is equal to the XOR of the previous word, $`w[i-1]`$, and the word $`Nk`$ positions earlier, $`w[i-Nk]`$. For words in positions that are a multiple of $`Nk`$, a transformation is applied to $`w[i-1]`$ prior to the XOR, followed by an XOR with a round constant, $`Rcon[i]`$. This transformation consists of a cyclic shift of the bytes in a word (`RotWord()`), followed by the application of a table lookup to all four bytes of the word (`SubWord()`).

It is important to note that the Key Expansion routine for 256-bit Cipher Keys ($`Nk = 8`$) is slightly different than for 128- and 192-bit Cipher Keys. If $`Nk = 8`$ and $`i-4`$ is a multiple of $`Nk`$, then `SubWord()` is applied to $`w[i-1]`$ prior to the XOR.

Appendix A (Page 27) presents examples of the Key Expansion. **Those could be helpful for debugging.**

#### Encryption Cipher (Sec. 5.1)

**Note: Follow the pseudo code in Figure 5, Page 15. Run this procedure in the `enc()` method.**

The cipher is roughly done in three steps:
1. At the start of the Cipher, the input is copied to the State array.
2. After an initial Round Key addition, the State array is transformed by implementing a round function 10, 12, or 14 times (depending on the key length), with the final round differing slightly from the first $`Nr-1`$ rounds.
3. The final State is then copied to the output.

The individual transformations - `SubBytes()`, `ShiftRows()`, `MixColumns()`, and `AddRoundKey()` - process the State, and are described in the following subsections.

As shown in the pseudo code, all $`Nr`$ rounds are identical with the exception of the final round, which does not include the $`MixColumns()`$ transformation.

Appendix B (Page 33) presents an example of the Cipher, showing values for the State array at the
beginning of each round and after the application of each of the four transformations described in the following sections. **Those could be helpful for debugging.**

##### `SubBytes()` Transformation (Sec. 5.1.1)

The `SubBytes()` transformation is a non-linear byte substitution that operates independently on each byte of the State using a substitution table (S-box, Figure 7, Page 16). **We have already copied the S-box to the `utils.py` for you. Check the tuple `S_BOX`.**

**You do not need to use the following techniques in your code. Simply apply the S-box.**

This S-box, which is invertible, is constructed by composing two transformations:
1. Take the multiplicative inverse in the finite field; the element $`{00}`$ is mapped to itself.
2. Apply the following affine transformation:
```math
b_i' = b_i \oplus b_{(i+4) \bmod 8} \oplus b_{(i+5) \bmod 8} \oplus b_{(i+6) \bmod 8} \oplus b_{(i+7) \bmod 8} \oplus c_i,
```
for $`0 \le i < 8`$, where $`b_i`$ is the $`i`$'th bit of the byte, and $`c_i`$ is the $`i`$'th bit of a byte $`c`$ with the value $`\{63\}`$ or $`\{01100011\}`$.

Figure 6 (Page 16) illustrates the effect of the `SubBytes()` transformation on the State.

##### `ShiftRows()` Transformation (Sec. 5.1.2)

In the `ShiftRows()` transformation, the bytes in the last three rows of the State are cyclically shifted over different numbers of bytes (offsets). The first row, `r = 0`, is not shifted. Specifically, the `ShiftRows()` transformation proceeds as follows:
```math
s_{r,c}' = s_{r,(c+\text{shift}(r,Nb)) \bmod Nb} \text{ for } 0 \le r < 4 \text{ and } 0 \le c < Nb,
```
where the shift value $`\text{shift}(r,Nb)`$ depends on the row number, $`r`$, as follows (recall that $`Nb = 4`$):
```math
\text{shift}(1,4) = 1; \quad \text{shift}(2,4) = 2; \quad \text{shift}(3,4) = 3.
```
You can assume $`\text{shift}(r,Nb) = r`$ in your code.

This has the effect of moving bytes to "lower" positions in the row (i.e., lower values of $`c`$ in a given row), while the "lowest" bytes wrap around into the "top" of the row (i.e., higher values of $`c`$ in a given row).

Figure 8 (Page 17) illustrates the `ShiftRows()` transformation.

##### `MixColumns()` Transformation (Sec. 5.1.3)

The `MixColumns()` transformation operates on the State column-by-column, treating each
column as a 4-term polynomial. The columns are considered as polynomials and multiplied modulo $`x^4 + 1`$ with a fixed polynomial $`a(x)`$, given by
```math
a(x) = \{03\}x^3 + \{01\}x^2 + \{01\}x + \{02\}.
```
As a result of this multiplication, the 4 bytes in a column are replaced by the following:
- $`s_{0,c}' = (\{02\} \bullet s_{0,c}) \oplus (\{03\} \bullet s_{1,c}) \oplus s_{2,c} \oplus s_{3,c}`$,
- $`s_{1,c}' = s_{0,c} \oplus (\{02\} \bullet s_{1,c}) \oplus (\{03\} \bullet s_{2,c}) \oplus s_{3,c}`$,
- $`s_{2,c}' = s_{0,c} \oplus s_{1,c} \oplus (\{02\} \bullet s_{2,c}) \oplus (\{03\} \bullet s_{3,c})`$,
- $`s_{3,c}' = (\{03\} \bullet s_{0,c}) \oplus s_{1,c} \oplus s_{2,c} \oplus (\{02\} \bullet s_{3,c})`$.

Figure 9 (Page 18) illustrates the `MixColumns()` transformation.

##### `AddRoundKey()` Transformation (Sec. 5.1.4)

In the `AddRoundKey()` transformation, a Round Key is added to the State by a simple bitwise XOR operation. Each Round Key consists of $`Nb`$ words from the key schedule. Those $`Nb`$ words are each added into the columns of the State, such that
```math
[s_{0,c}',s_{1,c}',s_{2,c}',s_{3,c}'] = [s_{0,c},s_{1,c},s_{2,c},s_{3,c}] \oplus [w_{round \cdot Nb + c}] \text{ for } 0 \le c < Nb,
```
where $`[w_i]`$ are the key schedule words, and round is a value in the range $`0 \le round \le Nr`$. In the Cipher, the initial Round Key addition occurs when $`round = 0`$, prior to the first application of the round function (see Figure 5, Page 15). The application of the `AddRoundKey()` transformation to the $`Nr`$ rounds of the Cipher occurs when $`1 \le round \le Nr`$.

The action of this transformation is illustrated in Figure 10 (Page 19), where $`l = round \cdot Nb`$.

#### Decryption Cipher (Inverse Cipher) (Sec. 5.3)

**Note: Follow the pseudo code in Figure 12, Page 21. Run this procedure in the `dec()` method.**

The Cipher transformations for encryption can be inverted and then implemented in reverse order to produce a straightforward inverse cipher for the AES algorithm. The individual transformations used in the Inverse Cipher - `InvShiftRows()`, `InvSubBytes()`, `InvMixColumns()`, and `AddRoundKey()` - process the State, and are described in the following subsections.

##### `InvShiftRows()` Transformation (Sec. 5.3.1)

`InvShiftRows()` is the inverse of the `ShiftRows()` transformation. The bytes in the last 3 rows of the State are cyclically shifted over different numbers of bytes (offsets). The first row, $`r = 0`$, is not shifted. The bottom three rows are cyclically shifted by $`Nb - \text{shift}(r, Nb)`$ bytes, where the shift value $`\text{shift}(r,Nb)`$ depends on the row number, and is given by
```math
\text{shift}(1,4) = 1; \quad \text{shift}(2,4) = 2; \quad \text{shift}(3,4) = 3.
```
You can assume $`\text{shift}(r,Nb) = r`$ in your code.

Specifically, the `InvShiftRows()` transformation proceeds as follows:
```math
s_{r,(c+\text{shift}(r,Nb)) \bmod Nb}' = s_{r,c} \text{ for } 0 \le r < 4 \text{ and } 0 \le c < Nb.
```
**You may have noticed that you only need to change a plus sign to minus in `ShiftRows()`:**
```math
s_{r,c}' = s_{r,(c-\text{shift}(r,Nb)) \bmod Nb} \text{ for } 0 \le r < 4 \text{ and } 0 \le c < Nb.
```
Figure 13 (Page 22) illustrates the `InvShiftRows()` transformation.

##### `InvSubBytes()` Transformation (Sec. 5.3.2)

`InvSubBytes()` is the inverse of the byte substitution transformation, in which the inverse S-box (Figure 14, Page 22) is applied to each byte of the State. This is obtained by applying the inverse of the affine transformation followed by taking the multiplicative inverse. **We have already copied the inverse S-box to the `utils.py` for you. Check the tuple `INV_S_BOX`.**

##### `InvMixColumns()` Transformation (Sec. 5.3.3)

`InvMixColumns()` is the inverse of the `MixColumns()` transformation. `InvMixColumns()` operates on the State column-by-column, treating each column as a 4-term polynomial. The columns are considered as polynomials and multiplied modulo $`x^4 + 1`$ with a fixed polynomial $`a^{-1}(x)`$, given by
```math
a^{-1}(x) = \{0b\}x^3 + \{0d\}x^2 + \{09\}x + \{0e\}.
```
As a result of this multiplication, the 4 bytes in a column are replaced by the following:
- $`s_{0,c}' = (\{0e\} \bullet s_{0,c}) \oplus (\{0b\} \bullet s_{1,c}) \oplus (\{0d\} \bullet s_{2,c}) \oplus (\{09\} \bullet s_{3,c})`$,
- $`s_{1,c}' = (\{09\} \bullet s_{0,c}) \oplus (\{0e\} \bullet s_{1,c}) \oplus (\{0b\} \bullet s_{2,c}) \oplus (\{0d\} \bullet s_{3,c})`$,
- $`s_{2,c}' = (\{0d\} \bullet s_{0,c}) \oplus (\{09\} \bullet s_{1,c}) \oplus (\{0e\} \bullet s_{2,c}) \oplus (\{0b\} \bullet s_{3,c})`$,
- $`s_{3,c}' = (\{0b\} \bullet s_{0,c}) \oplus (\{0d\} \bullet s_{1,c}) \oplus (\{09\} \bullet s_{2,c}) \oplus (\{0e\} \bullet s_{3,c})`$.

##### Inverse of the `AddRoundKey()` Transformation (Sec. 5.3.4)

`AddRoundKey()`, which was described before, is its own inverse, since it only involves an application of the XOR operation.

## Task 2: AES-CBC

### Requirements

AES is only a block cipher, and can only handle inputs of 128 bits. To handle an input of arbitrary number of bits, a typical solution is to split the input into several 128 bit blocks and apply the cipher to each block. To ensure security, some more operations are done. This is called a *block cipher mode of operation*.

A common mode is *Cipher Block Chaining* (CBC). In CBC mode, each block of plaintext is XORed with the previous ciphertext block before being encrypted. This way, each ciphertext block depends on all plaintext blocks processed up to that point. To make each message unique, an *initialization vector* (IV, a specified value of 64 bits) must be used in the first block.

![CBC Encryption](https://upload.wikimedia.org/wikipedia/commons/8/80/CBC_encryption.svg)
![CBC Decryption](https://upload.wikimedia.org/wikipedia/commons/2/2a/CBC_decryption.svg)

Let the blocks of plaintexts be $`P_1, \dots, P_n`$, and those of ciphertexts be $`C_1, \dots, C_n`$. The mathematical formula for CBC encryption is
```math
C_i = \mathsf{Enc}_K(P_i \oplus C_{i-1}), \quad C_0 = IV,
```
while the mathematical formula for CBC decryption is
```math
P_i = \mathsf{Dec}_K(C_i) \oplus C_{i-1}, \quad C_0 = IV.
```
Note that $`C_0`$ is not a ciphertext but an IV.

You may notice that the input may not be a multiple of 128 bits (16 bytes). The CBC mode does not allow a block less than 128 bits, so padding techniques are used.

Here we use PKCS\#7 padding.
- If the last block has less than 16 bytes, say $`16-N`$ bytes, pad the rest $`N`$ bytes with the value $`N`$. For example, if the last block is 10 bytes, pad the rest bytes with the value 6, or `06` in hexadecimal.
- If the last block has exactly 16 bytes, we add a new block and pad all the 16 bytes with the value 16, or `10` in hexadecimal. This is required to ensure that the last block always has padded values (otherwise, we cannot tell if the last block is padded or not).
After the padding, the number of bytes becomes a multiple of 128 bits (16 bytes).

Implement CBC mode on AES block cipher (i.e. AES-128-CBC, AES-192-CBC and AES-256-CBC), and use PKCS\#7 as padding method.

### I/O

The input is, roughly, the plaintext (for encryption) or ciphertext (for decryption) file, together with a 128-bit initialization vector (IV).

The output is, roughly, the ciphertext (for encryption) or plaintext (for decryption) file.

Read the docstring in the code for more details.

### Exceptions

- Raise `AESInputError` on any invalid inputs in the arguments.
- Raise `AESDataError` if the data is corrupted.
- Raise `AESPaddingError` if the padding doesn't seem correct.

Read the docstring in the code for more details.

### API

Your program must provide the following API.

```python
class AESCBC(AES):
    def encfile(self, infile, outfile, iv=0, pad=True):
        '''
        Input:
        - `infile`: str type, the input file (plaintext) path.
        - `outfile`: str type, the output file (ciphertext) path.
        - `iv`: int type, the 128-bit initialization vector (IV) encoded in
          big endian (0 by default).
        - `pad`: bool type, pad the file using PKCS#7 if True (True by
          default).

        Read the plaintext from file `infile`. Encrypt with AES-128/192/256-CBC
        and use `iv` as the initialization vector.
        If `pad` is True, pad the file using PKCS#7.
        If `pad` is False,
        - If the input is a multiple of 128 bits (16 bytes), don't pad.
        - Otherwise, raise `AESPaddingError`.
        Write the ciphertext to file `outfile`.
        DON'T CHECK the existence of `infile` or I/O errors.

        Raise `AESInputError` if and only if either
        - `infile` is not a str, or
        - `outfile` is not a str, or
        - `iv` is not an int, is negative, or cannot fit in 128 bits, or
        - `pad` is not a bool.

        Raise `AESPaddingError` if and only if `pad` is False, but the input is
        not a multiple of 128 bits (16 bytes).
        '''

    def decfile(self, infile, outfile, iv=0, pad=True):
        '''
        - `infile`: str type, the input file (ciphertext) path.
        - `outfile`: str type, the output file (plaintext) path.
        - `iv`: int type, the 128-bit initialization vector (IV) encoded in
          big endian (0 by default).
        - `pad`: bool type, remove the padding according to PKCS#7 if True
          (True by default).

        Read the ciphertext from file `infile`. Decrypt with AES-128/192/256-CBC
        and use `iv` as the initialization vector.
        If `pad` is True, remove the padding according to PKCS#7 before writing.
        Write the plaintext to file `outfile`.
        DON'T CHECK the existence of `infile` or I/O errors.

        Raise `AESInputError` if and only if either
        - `infile` is not a str, or
        - `outfile` is not a str, or
        - `iv` is not an int, is negative, or cannot fit in 128 bits, or
        - `pad` is not a bool.

        Raise `AESDataError` if and only if the input is not a multiple of
        128 bits.

        Raise `AESPaddingError` if and only if `pad` is True but the padding
        doesn't seem correct.
        '''
```

### Hints

- You can assume that the file size (for input or output) does not exceed 2048 bytes.
- You may find `openssl` useful. It provides command lines for many ciphers, including AES-128/192/256-CBC. It also uses PKCS\#7 padding. For example, encrypting a file `file.in` with AES-128-CBC, with `2b7e151628aed2a6abf7158809cf4f3c` as the cipher key and `0` as the initialization vector, without padding:
  ```sh
  openssl enc -aes-128-cbc -K 2b7e151628aed2a6abf7158809cf4f3c -e -in file.in -out file.out -iv 0 -nopad
  ```
  The output file `file.out` should exactly match the output of your program.
- To check the bytes of a file in hexadecimal, you can use `hexdump`. To avoid endianness issues, the option `-C` is suggested. For example, to check the file `file.txt`, run
  ```sh
  hexdump -C file.txt
  ```

# References

1. [FIPS 197, Advanced Encryption Standard (AES)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf)
2. [Advanced Encryption Standard - Wikipedia](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
3. [Rijndael key schedule - Wikipedia](https://en.wikipedia.org/wiki/Rijndael_key_schedule)
4. [Block cipher mode of operation - Wikipedia](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)
