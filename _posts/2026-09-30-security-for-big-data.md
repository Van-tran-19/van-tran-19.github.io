---
title: "Security for Big Data"
date: 2026-09-30 18:15:00 +0200
categories: [Security, BigData]
tags: [Cryptography, Symmetric, Asymmetric, Encryption]
toc: true
---
# Security for BigData

## Requirements

- Symmetric and asymmetric cryptography
- Different security definitions/models and cryptography assumptions

### Symmetric and asymmetric cryptography

**What is Encryption?** 

Encryption is a core cryptographic technique used to protect data by converting it into an unreadable form. It’s ensuring integrity, confidentiality and secure communication between system. Encryption techniques are broadly classified into symmetric and asymmetric key methods.  

> 
> 
> 
> <img width="434" height="76" alt="image" src="https://github.com/user-attachments/assets/4d1b104a-7971-45db-b955-f303966cd23f" />
> 

> Plain text is the data before encryption and after decryption. The cypher text is data after encrypted. To transform plain text into cypher text, algorithms are created by mathematician and cyber security expert. The recipient can interpret data because he gets a secret key that has been randomly generated.
> 

**Why use methods?** 

Each method serves different purposes based on performance, security needs and use cases. Modern systems combine both methods to achieve optimal security and efficiency. 

**Symmetric key encryption**

Symmetric key encryption is a method where the same secret key is used for both encrypting and decrypting data. Encrypt and decrypt use the same key. Using symmetric key encryption is much faster than asymmetric encryption, that provides a lower CPU cost. 

> 
> 
> 
> <img width="376" height="116" alt="image 1" src="https://github.com/user-attachments/assets/80f2edce-a4ea-482d-afa7-8d94671b958d" />
> 
> In this example, encryption and decryption both use the same shared secret key = 3. 
> 

It is generally used for large volumes of data, in file encryption, VPNs, databases and secure storage system. Symmetric key encryption requires a secure key-sharing mecanism, which can be a risk. The classic algorithm are: 

| Algorithm | key size (bit) |
| --- | --- |
| DES | 56 |
| RC4 | 128 |
| 3DES | 168 |
| AES | 128, 192, 256 |
| ChaCha20 | 128 or 256 |

**Asymmetric Key Encryption**

## Sources

1. https://www.geeksforgeeks.org/computer-networks/difference-between-symmetric-and-asymmetric-key-encryption/
2. https://www.youtube.com/watch?v=AQDCe585Lnc
3. https://www.youtube.com/watch?v=o_g-M7UBqI8
