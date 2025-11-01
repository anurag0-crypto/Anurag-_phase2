# RSA Oracle

> Can you abuse the oracle? An attacker was able to intercept communications between a bank and a fintech company. They managed to get the [message](https://artifacts.picoctf.net/c_titan/32/secret.enc) (ciphertext) and the [password](https://artifacts.picoctf.net/c_titan/32/password.enc) that was used to encrypt the message.

## Solution:

The challenge involves exploiting a Chosen Ciphertext Attack vulnerability in an RSA implementation. The attack leverages the homomorphic property of RSA encryption to recover an unknown plaintext through strategic manipulation.

The mathematical foundation of the attack works as follows:
- We have ciphertext `C = t^e mod n` that we want to decrypt
- We choose a multiplier value `r` and compute `Ca = r^e mod n`  
- We submit `Cb = C × Ca mod n` to the decryption oracle
- The oracle returns `m' = (Cb)^d mod n = (t × r)^ed mod n = t × r mod n`
- We then compute `t = m' ÷ r` to recover the original message

In this implementation, I used `r = 1` (ASCII value 49, hex 0x31) since the oracle only accepts character input. The complete interaction with the oracle demonstrates the attack:

```
*****************************************
****************THE ORACLE***************
*****************************************
what should we do for you?
E --> encrypt D --> decrypt.
E
enter text to encrypt (encoded length must be less than keysize): 1
1

encoded cleartext as Hex m: 31

ciphertext (m ^ e mod n) 4374671741411819653095065203638363839705760144524191633605358134684143978321095859047126585649272872908765432040943055399247499744070371810470682366100689
```

The encrypted multiplier is then used to create the modified ciphertext for decryption:

```
what should we do for you?
E --> encrypt D --> decrypt.
D
Enter text to decrypt: 7721457704147667243014822210342015623441070646730334329369170960482434424280036173642970043315437595085392083961667520112650054794430310042790658505492250245959249047521482534760405619663668251566587006095670290533332380625235238831595584837302490911416483426164534051847266716546285422298258161162428036900
decrypted ciphertext as hex (c ^ d mod n): ac2c1742ee9
decrypted ciphertext:
ÂÁt.é
```

The returned hex value `ac2c1742ee9` represents `t × r`. To recover the original password:

```
t = 0xac2c1742ee9 ÷ 0x31 = 0x3838316439
```

Converting this hex to ASCII gives `881d9`, which is the password needed to decrypt the secret message using OpenSSL:

```
> openssl enc -aes-256-cbc -d -in secret.enc -k  881d9
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
picoCTF{su((3ss_(r@ck1ng_r3@_881d93b6}
```

## Flag:

```
picoCTF{su((3ss_(r@ck1ng_r3@_881d93b6}
```

## Concepts learnt:

- Chosen Ciphertext Attack on RSA
- RSA multiplicative homomorphic property
- Cryptographic oracle exploitation
- Combining asymmetric and symmetric encryption systems

## Notes:

- The attack works due to RSA's property: `enc(m1 × m2) = enc(m1) × enc(m2) mod n`
- Any multiplier value can be used as long as we can compute its encryption
- The oracle's character input restriction influenced the choice of multiplier
- Proper padding schemes like OAEP prevent this type of attack

## Resources:

https://crypto.stackexchange.com/questions/2323/how-does-a-chosen-plaintext-attack-on-rsa-work

---

# Custom Encryption

> Decrypt the flag encrypted with a custom algorithm

## Solution:

The challenge involves reversing a custom encryption algorithm that combines Diffie-Hellman key exchange, XOR encryption, and multiplication-based transformation.

The complete encryption algorithm is implemented in `custom_encryption.py`:

```python
from random import randint
import sys

def generator(g, x, p):
    return pow(g, x) % p

def encrypt(plaintext, key):
    cipher = []
    for char in plaintext:
        cipher.append(((ord(char) * key*311)))
    return cipher

def is_prime(p):
    v = 0
    for i in range(2, p + 1):
        if p % i == 0:
            v = v + 1
    if v > 1:
        return False
    else:
        return True

def dynamic_xor_encrypt(plaintext, text_key):
    cipher_text = ""
    key_length = len(text_key)
    for i, char in enumerate(plaintext[::-1]):
        key_char = text_key[i % key_length]
        encrypted_char = chr(ord(char) ^ ord(key_char))
        cipher_text += encrypted_char
    return cipher_text

def test(plain_text, text_key):
    p = 97
    g = 31
    if not is_prime(p) and not is_prime(g):
        print("Enter prime numbers")
        return
    a = randint(p-10, p)
    b = randint(g-10, g)
    print(f"a = {a}")
    print(f"b = {b}")
    u = generator(g, a, p)
    v = generator(g, b, p)
    key = generator(v, a, p)
    b_key = generator(u, b, p)
    shared_key = None
    if key == b_key:
        shared_key = key
    else:
        print("Invalid key")
        return
    semi_cipher = dynamic_xor_encrypt(plain_text, text_key)
    cipher = encrypt(semi_cipher, shared_key)
    print(f'cipher is: {cipher}')

if __name__ == "__main__":
    message = sys.argv[1]
    test(message, "trudeau")
```

The encryption process involves:
1. **Key Generation**: Diffie-Hellman with p=97, g=31, a=95, b=21
2. **XOR Encryption**: Using key "trudeau" on reversed plaintext
3. **Final Encryption**: Multiplying each character by shared_key × 311

Given the encrypted data:
```
a = 95
b = 21
cipher is: [237915, 1850450, 1850450, 158610, 2458455, 2273410, 1744710, 1744710, 1797580, 1110270, 0, 2194105, 555135, 132175, 1797580, 0, 581570, 2273410, 26435, 1638970, 634440, 713745, 158610, 158610, 449395, 158610, 687310, 1348185, 845920, 1295315, 687310, 185045, 317220, 449395]
```

The decryption process reverses each step. First, calculate the shared key:
```
u = g^a mod p = 31^95 mod 97 = 72
v = g^b mod p = 31^21 mod 97 = 8  
key = v^a mod p = 8^95 mod 97 = 85
b_key = u^b mod p = 72^21 mod 97 = 85
shared_key = 85
```

The decryption multiplier is `85 × 311 = 26435`. The Python solution reverses the encryption:

```python
# shared key = 85 
# text key = trudeau 

x = [237915, 1850450, 1850450, 158610, 2458455, 2273410, 1744710, 1744710, 1797580, 1110270, 0, 2194105, 555135, 132175, 1797580, 0, 581570, 2273410, 26435, 1638970, 634440, 713745, 158610, 158610, 449395, 158610, 687310, 1348185, 845920, 1295315, 687310, 185045, 317220, 449395]

xor_text = []
xor_ascii_number = []
xor_ascii_bin = []

div = 26435 # 85 * 311
for i in range(len(x)):
    y = chr(int(x[i]/div))
    xor_ascii_number.append(int(x[i]/div))
    xor_text.append(y)

print(xor_text)
print(xor_ascii_number)

semi_ciphertext = "".join(xor_text)

plaintext = ""
text_key = "trudeau"
key_length = len(text_key)
for i, char in enumerate(semi_ciphertext):
    key_char = text_key[i % key_length]
    decrypted_char = chr(ord(char) ^ ord(key_char))
    plaintext += decrypted_char
print(plaintext[::-1])
```

## Flag:

```
picoCTF{custom_d2cr0pt6d_66778b34}
```

## Concepts learnt:

- Custom cryptographic algorithm analysis and reversal
- Diffie-Hellman key exchange implementation
- XOR cipher cryptanalysis
- Multi-step encryption process reversal
- Mathematical operations in cryptography

## Notes:

- The algorithm reverses the plaintext before XOR encryption
- The shared key remains constant given fixed a and b values
- Division works perfectly since multiplication doesn't involve modular arithmetic
- XOR operations are reversible using the same key

## Resources:

- Python documentation for cryptographic operations

---

# miniRSA

> Let's decrypt this: https://jupiter.challenges.picoctf.org/static/ee7e2388b45f521b285334abb5a63771/ciphertext Something seems a bit small.

## Solution:

The challenge presents an RSA encryption with a very small public exponent (e=3), making it vulnerable to a cube root attack when the plaintext is small enough that `m^3 < n`.

Given the RSA parameters:
```text
N: 29331922499794985782735976045591164936683059380558950386560160105740343201513369939006307531165922708949619162698623675349030430859547825708994708321803705309459438099340427770580064400911431856656901982789948285309956111848686906152664473350940486507451771223435835260168971210087470894448460745593956840586530527915802541450092946574694809584880896601317519794442862977471129319781313161842056501715040555964011899589002863730868679527184420789010551475067862907739054966183120621407246398518098981106431219207697870293412176440482900183550467375190239898455201170831410460483829448603477361305838743852756938687673
e: 3

ciphertext (c): 2205316413931134031074603746928247799030155221252519872649649212867614751848436763801274360463406171277838056821437115883619169702963504606017565783537203207707757768473109845162808575425972525116337319108047893250549462147185741761825125
```

The encryption formula is:
```
c = m^3 mod n
```

When `m^3 < n`, the modulo operation doesn't affect the result, allowing recovery of the plaintext by simply computing the cube root of the ciphertext:
```
m = c^(1/3)
```

Using an online RSA tool (dcode.fr), the cube root of the ciphertext was computed and converted to ASCII, revealing the flag:

![](assets/minirsa.png)

## Flag:

```
picoCTF{n33d_a_lArg3r_e_606ce004}
```

## Concepts learnt:

- RSA with small public exponent vulnerability
- Cube root attack on textbook RSA
- Importance of proper padding in RSA
- Textbook RSA weaknesses and mitigations
- Mathematical attacks on cryptographic implementations

## Notes:

- This attack works specifically when `m^e < n`
- In practice, proper padding schemes like OAEP prevent this attack
- The flag itself suggests the solution - a larger exponent is needed
- Small exponents improve performance but reduce security without proper padding

## Resources:

https://en.wikipedia.org/wiki/RSA_cryptosystem
https://crypto.stackexchange.com/questions/33561/cube-root-attack-rsa-with-low-exponent
https://www.dcode.fr/rsa-cipher
