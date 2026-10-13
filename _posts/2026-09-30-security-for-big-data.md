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

This encryption uses a pair of keys: public key and private key to secure data. 

The public key is used to encrypt data and the private key for decryption. It eliminates the needed to share a secret key, which is improved security. It still slower than symmetric key it requires much more resources. Common algorithms include RSA, ECC, Diffie-Hellman, ElGamal, and DSA and its commonly used in SSL/TLS, digital signatures, secure emails, and authentication systems.

> For example, the RSA algorithm can generate private and public key which are mathematically link together. Only the matching key can be used to decrypt the data encrypted. To exchange a message, the sender can use the public key of the receiver to encrypt the data and send it to the receiver. Then the receiver decrypt the message with its private key, who is the only person who have access to it. Nobody can decrypt the message without stealing the private key of the receiver.
> 

### Symmetric vs Asymmetric

| **Symmetric Key Encryption** | **Asymmetric Key Encryption** |
| --- | --- |
| It only requires a single key for both encryption and decryption. | It requires two keys, a public key and a private key, one to encrypt and the other to decrypt. |
| The size of cipher text is the same or smaller than the original plain text. | The size of cipher text is the same or larger than the original plain text. |
| The encryption process is very fast. | The encryption process is slow. |
| It is used when a large amount of data needs to be transferred. | It is used to transfer small amount of data. |
| It only provides confidentiality. | It provides confidentiality, authenticity, and non-repudiation. |
| The length of key used is 128 or 256 bits | The  length of key used is 2048 or higher |
| In symmetric key encryption, resource utilization is low compared to asymmetric key encryption. | In asymmetric key encryption, resource utilization is high. |
| It is efficient as it is used for handling large amount of data. | It is comparatively less efficient as it can handle a small amount of data. |
| Security is lower as only one key is used for both encryption and decryption purposes. | Security is higher as two keys are used, one for encryption and the other for decryption. |
| **Mathematical representation:**        $P = D (K, E(K, P))$
where
$K$**:** encryption and decryption key
$P$**:** plain text
$D$**:** Decryption 
$E(K, P)$**:** Encryption of plain text using $K$ | **Mathematical representation:**        $P = D(K_d, E (K_e,P))$
where
$K_e$**:** encryption key
$K_d$**:** decryption key
$D$**:** Decryption
$E(K_e, P)$**:** Encryption of plain text using encryption key $K_e$. $P$ --> plain text |
| **Examples:** 3DES, AES, DES and RC4 | **Examples:** Diffie-Hellman, ECC, El Gamal, DSA and RSA |

## Big Data

Originally, big data is defined as “To me it means working with data at a large scale and velocity” according to Kyle Rush, Head of Optimization, Optimizely. Gartner introduced 3Vs to define big data: 

- Volume: Huge amount of generated data
- Velocity: Real time stream
- Variety: Vast range of data sources, data formats and data types

> Nowadays, big data becomes more complex, a simple definition is hard to find.
> 
> 
> <img width="407" height="273" alt="image 2" src="https://github.com/user-attachments/assets/17f45e21-755e-49ef-aa9b-6e6ac105e6de" />
> 

## Technologies for Big Data

Cloud technologies and IOT technologies contribute massively for Big Data. 

**What is Cloud?** https://van-tran-19.github.io/posts/cloud/

“Cloud computing is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction” according to NIST. 

**What is IOT?** 

IoT *creates* massive data streams, and Big Data systems are what allow us to *store, process, analyze, and act* on that flood of information. Big Data is the **engine** that makes the data useful. 

IoT devices (sensors, cameras, meters, wearables, industrial machines…) continuously generate:

- **High‑volume** data (petabytes → exabytes)
- **High‑velocity** streams (millisecond latency)
- **High‑variety** formats (time series, logs, images, video)
- **Low‑veracity** issues (noise, missing data, errors)

These characteristics are known as the **4 Vs of IoT Big Data**.

A single machine can produce **thousands of readings per hour**, and globally IoT devices generate **tens of zettabytes per year**.

Bid Data provides the ability to handle the flood of information. 

## Data management lifecycle

| Data Acquisition/Collection | gathering, filtering, cleaning data before storage
data sources: sensors, wearable devices, social networks, public registers, etc |
| --- | --- |
| Data Analysis | making “raw” collected data “meaningful” for decision making
structured or unstructured data
analysis techniques: descriptive, predictive, etc |
| Data Curation | Making analysed data “qualitative”
content creation, selection, classification, transformation, validation, preservation. |
| Data Storage | storing and managing data while enabling access
storage solutions: cloud storage, distributed storage |
| Data Usage | use of data by interested parties
public or restricted access
potential users: general public, single organization, or multiple users such as banks, retailers, etc. |

## Data privacy and security requirements

### CIA triad

The CIA triad is composed by confidentiality, integrity and availability. The CIA triad is implemented if a user wants to create something that requires security, this thing needs to check all of the requirements of the CIA triad. 

**Confidentiality** ensures that information is hiding from authorized parties. 

**Integrity** ensures that data remains unchanged and accurate. 

**Availability** ensures that data is accessible. 

### GDPR

GDPR tends for general data protection regulations. It gives individuals greater control, new obligations for company since may 25th 2018. It is a protection of individuals with regards to the processing of personal data, it applies to EU companies & to non-EU companies that process personal data of individuals in the EU and it finally fines up to 4% of annual global revenue or 20 million euros. 

**Key Concepts & Roles**

- **Personal Data:** Any information relating to an identified or identifiable natural person.
- **Data Controller:** The entity that determines the *purposes and means* of data processing.
- **Data Processor:** The entity that processes personal data *on behalf* of the controller.
- **Data Subject Rights:** Individuals maintain broad rights, including the right to be informed, access, rectification, erasure ("right to be forgotten"), data portability, and restriction of automated decision-making

With AI incidents multiplying across industries, the **EU AI Act** establishes a comprehensive legal framework to ensure AI systems are safe, transparent, traceable, non-discriminatory, and respectful of fundamental rights. It utilizes a strict **risk-based pyramid**:   

|  **Unacceptable Risk (Banned)** | Applications that threaten safety and fundamental rights such as social scoring, cognitive behavioral manipulation, and real-time remote biometric identification in public spaces are strictly prohibited.    |
| --- | --- |
| **High Risk (Heavily Regulated)** | systems used in critical infrastructure, medical devices, biometric categorization, employment screening, and law enforcement require rigorous data governance, transparency, human oversight, and a Fundamental Rights Impact Assessment (FRIA).    |
| **Transparency Risk** | Chatbots and deepfakes carry clear disclosure and content-labelling obligations to ensure users know when they are interacting with synthetic media |
|  **Minimum Risk** | Standard AI systems (e.g., AI-enabled video games or spam filters) face minimal or no mandatory restrictions |

## Sources

1. https://www.geeksforgeeks.org/computer-networks/difference-between-symmetric-and-asymmetric-key-encryption/
2. https://www.youtube.com/watch?v=AQDCe585Lnc
3. https://www.youtube.com/watch?v=o_g-M7UBqI8
4. https://www.youtube.com/watch?v=kPPFNrlN3zo
5. https://www.youtube.com/watch?v=j6wwBqfSk-o&t=75s
