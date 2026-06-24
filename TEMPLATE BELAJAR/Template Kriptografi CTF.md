# 🔐 Template Kriptografi CTF — LKS Cybersecurity 2026

  

> **Alur:** Identifikasi jenis cipher → Decode → Verifikasi → Submit

  

---

  

## IDENTIFIKASI CEPAT

  

| Ciri-ciri | Kemungkinan Cipher |

|-----------|-------------------|

| Hanya huruf kapital | Caesar / Vigenere / Atbash |

| `=` atau `==` di akhir | Base64 |

| Hanya 0 dan 1 | Binary |

| Panjang 32 char hex | MD5 |

| Panjang 40 char hex | SHA1 |

| Panjang 64 char hex | SHA256 |

| `$2b$` atau `$2y$` | BCrypt |

| `$1$` | MD5-crypt |

| `-----BEGIN...-----` | PEM (RSA/cert) |

| Tanda titik `.` banyak | Morse |

| Angka besar tidak biasa | RSA |

| Spasi dan titik | Morse Code |

  

---

  

## 1. ENCODING (BUKAN ENKRIPSI)

  

### Base64

  

```bash

# Decode:

echo "SGVsbG8gV29ybGQ=" | base64 -d

  

# Encode:

echo -n "Hello World" | base64

  

# Python:

python3 -c "import base64; print(base64.b64decode('SGVsbG8gV29ybGQ=').decode())"

  

# Base64 URL-safe (- dan _ ganti + dan /):

python3 -c "import base64; print(base64.urlsafe_b64decode('SGVsbG8-V29ybGQ=').decode())"

```

  

### Base32

  

```bash

echo "JBSWY3DPEBLW64TMMQQQ====" | base32 -d

python3 -c "import base64; print(base64.b32decode('JBSWY3DPEBLW64TMMQQQ====').decode())"

```

  

### Base85 / ASCII85

  

```python

import base64

print(base64.b85decode(b'Xk~0{Xk~0{').decode())

```

  

### Hex / Base16

  

```bash

# Decode hex:

echo "48656c6c6f" | xxd -r -p

python3 -c "print(bytes.fromhex('48656c6c6f').decode())"

  

# Encode ke hex:

echo -n "Hello" | xxd -p

python3 -c "print('Hello'.encode().hex())"

```

  

### Binary

  

```python

# Decode binary:

binary = "01001000 01100101 01101100 01101100 01101111"

result = ''.join(chr(int(b, 2)) for b in binary.split())

print(result)

```

  

### Morse Code

  

```python

# Morse decoder:

MORSE = {

    '.-': 'A', '-...': 'B', '-.-.': 'C', '-..': 'D', '.': 'E',

    '..-.': 'F', '--.': 'G', '....': 'H', '..': 'I', '.---': 'J',

    '-.-': 'K', '.-..': 'L', '--': 'M', '-.': 'N', '---': 'O',

    '.--.': 'P', '--.-': 'Q', '.-.': 'R', '...': 'S', '-': 'T',

    '..-': 'U', '...-': 'V', '.--': 'W', '-..-': 'X', '-.--': 'Y',

    '--..': 'Z', '-----': '0', '.----': '1', '..---': '2',

    '...--': '3', '....-': '4', '.....': '5', '-....': '6',

    '--...': '7', '---..': '8', '----.': '9'

}

  

morse_input = ".... . .-.. .-.. ---"   # <-- GANTI INI

words = morse_input.strip().split('  ')  # 2 spasi = pemisah kata

result = ' '.join(''.join(MORSE.get(char, '?') for char in word.split()) for word in words)

print(result)

```

  

---

  

## 2. CIPHER KLASIK

  

### Caesar / ROT

  

```python

# Brute force semua ROT (0-25):

def caesar_bruteforce(ciphertext):

    ciphertext = ciphertext.upper()

    for shift in range(26):

        result = ''

        for char in ciphertext:

            if char.isalpha():

                result += chr((ord(char) - 65 + shift) % 26 + 65)

            else:

                result += char

        print(f"ROT{shift:2d}: {result}")

  

caesar_bruteforce("CIPHER_TEKS_DISINI")  # <-- GANTI

  

# ROT13 khusus:

import codecs

print(codecs.encode("hello", 'rot_13'))

```

  

### Vigenere

  

```python

def vigenere_decrypt(ciphertext, key):

    result = []

    key = key.upper()

    key_idx = 0

    for char in ciphertext.upper():

        if char.isalpha():

            shift = ord(key[key_idx % len(key)]) - 65

            decrypted = chr((ord(char) - 65 - shift) % 26 + 65)

            result.append(decrypted)

            key_idx += 1

        else:

            result.append(char)

    return ''.join(result)

  

ciphertext = "CIPHERTEXT"  # <-- GANTI

key = "KEY"                 # <-- GANTI

print(vigenere_decrypt(ciphertext, key))

```

  

### Atbash

  

```python

def atbash(text):

    result = ''

    for char in text.upper():

        if char.isalpha():

            result += chr(90 - (ord(char) - 65))

        else:

            result += char

    return result

  

print(atbash("CIPHERTEXT"))  # <-- GANTI

```

  

### XOR

  

```python

# XOR single byte (brute force):

ciphertext = bytes.fromhex("1a2b3c4d5e")  # <-- GANTI ke hex ciphertext

for key in range(256):

    result = bytes(b ^ key for b in ciphertext)

    if all(32 <= c <= 126 for c in result):  # printable ASCII

        print(f"Key {key} (0x{key:02x}): {result.decode()}")

  

# XOR dengan key string:

def xor_decrypt(ciphertext_hex, key_str):

    ciphertext = bytes.fromhex(ciphertext_hex)

    key = key_str.encode()

    result = bytes(ciphertext[i] ^ key[i % len(key)] for i in range(len(ciphertext)))

    return result

  

print(xor_decrypt("HEXSTRING", "KEY"))  # <-- GANTI

```

  

---

  

## 3. RSA

  

### RSA Basics

  

```

Public key: (n, e)

Private key: (n, d)

Enkripsi: c = m^e mod n

Dekripsi: m = c^d mod n

```

  

### RSA Solver Template

  

```python

from sympy import factorint, mod_inverse

  

# ===== DATA DARI SOAL =====

n = 3233    # <-- GANTI

e = 17      # <-- GANTI

c = 2790    # <-- GANTI ciphertext

# p, q biasanya diberikan atau harus difaktor

  

# Jika p dan q diberikan:

p = 61      # <-- GANTI

q = 53      # <-- GANTI

  

# Hitung private key:

phi_n = (p - 1) * (q - 1)

d = mod_inverse(e, phi_n)

  

# Dekripsi:

m = pow(c, d, n)

print(f"Pesan (angka): {m}")

print(f"Pesan (teks): {m.to_bytes((m.bit_length()+7)//8, 'big').decode(errors='replace')}")

```

  

### RSA — Faktorisasi (jika n kecil)

  

```python

from sympy import factorint

  

n = 3233  # <-- GANTI

factors = factorint(n)

print(f"Faktor dari n: {factors}")

# Output: {61: 1, 53: 1} → p=61, q=53

```

  

### RsaCtfTool (otomatis)

  

```bash

# Install:

git clone https://github.com/RsaCtfTool/RsaCtfTool.git

cd RsaCtfTool && pip3 install -r "optional-requirements.txt" --break-system-packages

  

# Gunakan:

python3 RsaCtfTool.py --publickey public.pem --uncipherfile ciphertext.txt

python3 RsaCtfTool.py -n <N> -e <E> --uncipher <C>

```

  

---

  

## 4. HASH CRACKING

  

```bash

# Identifikasi hash:

hash-identifier <HASH>

hashid <HASH>

  

# John the Ripper:

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

john --format=raw-md5 hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

john --format=raw-sha1 hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

john --show hash.txt   # lihat hasil

  

# Hashcat:

# Mode: -m 0=MD5, -m 100=SHA1, -m 1400=SHA256, -m 1000=NTLM, -m 1800=sha512crypt

hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt --show

  

# Online (jika diizinkan):

# https://crackstation.net/

# https://hashes.com/en/decrypt/hash

  

# Generate wordlist kustom:

crunch 8 8 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt

```

  

---

  

## 5. OPENSSL — ENKRIPSI MODERN

  

```bash

# Dekripsi AES-256-CBC:

openssl enc -aes-256-cbc -d -in encrypted.enc -out decrypted.txt -k "password"

# dengan salt:

openssl enc -aes-256-cbc -d -in encrypted.enc -out decrypted.txt -k "password" -md sha256

  

# Dekripsi RSA:

openssl rsautl -decrypt -inkey private.pem -in cipher.bin -out plain.txt

# atau:

openssl pkeyutl -decrypt -inkey private.pem -in cipher.bin -out plain.txt

  

# Baca sertifikat:

openssl x509 -in cert.pem -text -noout

openssl req -in req.csr -text -noout

  

# Baca private key info:

openssl rsa -in private.pem -text -noout

```

  

---

  

## 6. PYTHON ONE-LINERS SERBA GUNA

  

```python

# Decode berbagai format sekaligus (coba berurutan):

import base64, binascii, codecs

  

data = "MASUKKAN_DATA_DI_SINI"

  

# Coba base64:

try: print("Base64:", base64.b64decode(data).decode())

except: pass

  

# Coba hex:

try: print("Hex:", bytes.fromhex(data).decode())

except: pass

  

# Coba ROT13:

print("ROT13:", codecs.encode(data, 'rot_13'))

  

# Coba binary:

try:

    groups = data.split()

    print("Binary:", ''.join(chr(int(g, 2)) for g in groups))

except: pass

  

# Coba base32:

try: print("Base32:", base64.b32decode(data).decode())

except: pass

```

  

---

  

> 💡 **Tips Crypto CTF:**

> - Selalu coba decode base64 terlebih dahulu — paling sering muncul

> - Cek panjang hash untuk identifikasi: MD5=32, SHA1=40, SHA256=64

> - Untuk RSA, cek nilai `e` yang tidak biasa (misal e=3 bisa diserang dengan Hastad)

> - Gunakan [CyberChef](https://gchq.github.io/CyberChef/) untuk chaining decode secara visual