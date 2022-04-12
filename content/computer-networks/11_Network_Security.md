---
layout: default
usemathjax: true
permalink: /network/ch11
---

# Network Security

**Confidentiality**: only sender and intended receiver can read message contents
- Cannot be viewed in transit or by wrong receiver
- Sender encrypt
- Receiver decrypt

**Authentication**: sender and receiver confirm each other's identity

**Message integrity**: Readonly; message sholdn't be altered without detection

**Access and availability**: Services must be accessible to all users.

**Threat model**: what are the capabilities of the attackers/threat?
- Eavesdropping (intercepting messages)
- Insertion of messages
- Impersonation via spoofing source address
  Hijacking: "taking over" ongoing connection
- Denial of service (prevent serivce from being used by other authorized users)
  - DDos: many different attackers attacking one server.

## Terms

- $m$: plaintext message
- $K_A$: Alice's encryption key
- $K_B$: Bob's decryption key
- $K_A(m)$: ciphertext, encrypted with key $K_A$
- $m = K_B(K_A(m))$

## How to break an encryption scheme

1. Cipher-text only attack (attacker only has ciphertext)
   1. Brute force (go through all possibilities in the search space)
   2. Statistical analysis (attempts to shrink the search space)
2. Known-plaintext attack (attacker has some plaintexts and their ciphertexts)
3. Chosen-plaintext (attakcer can get ciphertext for chosen plaintext)

## Cryptographies

### Symmetric key cryptography

Bob and Aliceshare same symmetric key $K_A = K_B$.

#### Substitution cipher

e.g. Monoalphabetic cipher. Substitution of one letter for another.

Drawbacks: 
- static mapping
- long key (all letters in alphabet)
- can be cracked by frequency analysis (matching common frequencies of letters)

#### Caesar's cipher

Rotates the alphabet (with wraparound) by a certain amount $k$.
- Left rotation: ciphertext key needs to be rotated left by $k$ letters
- Right rotation: ciphertext key needs to be rotated right by $k$ letters
- Key: single number $k$

Drawbacks: 
- static mapping
- can be cracked by frequency analysis (matching common frequencies of letters)

#### n-cipher approach

Start with $n$ substitution ciphers $M_1, M_2, \dots, M_n$. Establish a cyclic permutation of ciphers e.g. $M_4, M_1, M_3, M_6, M_5, M_2, M_4, M_1, \dots$

For each next plaintext symbol, use the next substitution pattern.

Key: n substitution ciphers, cyclic pattern

#### Encryption standards

DES: Data Encryption Standard

AES: Advanced Encryption Standard

### Public Key Cryptography