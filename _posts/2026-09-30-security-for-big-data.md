---
title: "Security for Big Data"
date: 2026-09-30 18:15:00 +0200
categories: [Security, BigData]
tags: [Cryptography, Symmetric, Asymmetric, Encryption]
toc: true
math: true
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
> <img width="434" height="76" alt="image" src="https://github.com/user-attachments/assets/e01c25fa-4ab0-4692-bfae-d4467ed2285b" />
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
> <img width="376" height="116" alt="image 1" src="https://github.com/user-attachments/assets/60059c9d-ce22-483a-99c2-312aa1087cb7" />
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

> For example, the RSA algorithm can generate private and public key which are mathematically linked together. Only the matching key can be used to decrypt the data encrypted. To exchange a message, the sender can use the public key of the receiver to encrypt the data and send it to the receiver. Then the receiver decrypts the message with its private key, who is the only person who have access to it. Nobody can decrypt the message without stealing the private key of the receiver.
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
> <img width="407" height="273" alt="image 2" src="https://github.com/user-attachments/assets/bcce12bc-4d2c-40a4-8a17-6e275a8c7d55" />
> 

## Technologies for Big Data

Cloud technologies and IOT technologies contribute massively for Big Data. 

**What is Cloud?** https://van-tran-19.github.io/posts/cloud/

“Cloud computing is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction” according to NIST. 

**What is IOT?** 

IoT creates massive data streams, and Big Data systems are what allow us to store, process, analyze, and act on that flood of information. Big Data is the **engine** that makes the data useful. 

IoT devices (sensors, cameras, meters, wearables, industrial machines…) continuously generate:

- **High‑volume** data (petabytes → exabytes)
- **High‑velocity** streams (millisecond latency)
- **High‑variety** formats (time series, logs, images, video)
- **Low‑veracity** issues (noise, missing data, errors)

These characteristics are known as the **4 Vs of IoT Big Data**.

A single machine can produce **thousands of readings per hour**, and globally IoT devices generate **tens of zettabytes per year**.

Bid Data provides the ability to handle the flood of information. 

## Data management lifecycle

| Phase | Description |
| --- | --- |
| Data Acquisition/Collection | gathering, filtering, cleaning data before storage
data sources: sensors, wearable devices, social networks, public registers, etc |
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

GDPR stands for general data protection regulations. It gives individuals greater control, new obligations for company since may 25th 2018. It is a protection of individuals with regards to the processing of personal data, it applies to EU companies & to non-EU companies that process personal data of individuals in the EU and it finally fines up to 4% of annual global revenue or 20 million euros. 

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

## Anonymisation for Big Data

The goal is to collect and release data such that the data remains practically useful while
the identities of the subjects of the data are not revealed. User needs to find the perfect balance between privacy and utility: finds a way to protect privacy with the minimum loss of accuracy. 

| step | what |
| --- | --- |
| Pseudonymization | Replace identifiers with pseudonyms using very secure techniques.  |
| Attribute suppresion | Remove identifying attributes from the data |

> 
> 
> 
> <img width="276" height="192" alt="image 3" src="https://github.com/user-attachments/assets/3540eecc-ff1a-450b-82f4-62eae0f6b842" />
> 
> In many cases, a certain number of people can uniquely be identified with these attributes: ZIP code, birth date and sex. 
> 
> It’s the reason why researchers must find a way to categorize correctly data by groups: 
> 
> - **Identifiers** such as attributes that unambiguously identify respondents (PII) (name, passport number, SS number, etc)
> - **Quasi-identifiers** like **a**ttributes that, in combination with others can lead to
> unambiguous identification (Age, address, gender, etc)
> - **Sensitive attributes** such as attributes that contain sensitive information (disease, salary, etc)
> - **Non-sensitive attributes** like attributes that do not belong to any of the previous categories

### What are different methods to anonymize data?

#### Non-perturbative methods

They are operations that do not modify data, where original data are not denatured but are still true. 

| Step | what |
| --- | --- |
| Sampling | Only release a sample of the original data set |
| Generalisation (AKA coarsening) | Combine several categories, values |
| Local suppression | Suppress certain individual attributes and can be combined with generalization |

#### Perturbative methods

This method consists on modifying data while some properties are retained. Original data is denatured. 

| methods | what |
| --- | --- |
| Noise addition | Add ε to each data record such that $N(0,\alpha\sum)$ where $\sum$corresponds to the covariance matrix of the original data; Choose $\alpha$ such that means and correlations are preserved; Applicable to numerical data only  |
| Micro-aggregation | It’s a privacy technique that groups similar data records together into clusters of at least $k$ individuals and replaces their true values with the group's average or most common value, hiding unique identities while preserving overall statistical accuracy. |
| Data swapping | It involves taking specific attribute values (columns) from one record and swapping them with the matching attribute values of another record in the dataset. |
| Post-randomization | Post-Randomization (often called **PRAM**) is a probabilistic technique where the values of categorical attributes in a dataset are intentionally changed (perturbed) according to a predefined statistical model known as a **Markov transition matrix**. |

**What is the covariance matrix of the original data and what does it mean?** 

$\Sigma =
\begin{pmatrix}
\mathrm{Cov}(X_1, X_1) & \mathrm{Cov}(X_1, X_2) & \cdots & \mathrm{Cov}(X_1, X_n) \\
\mathrm{Cov}(X_2, X_1) & \mathrm{Cov}(X_2, X_2) & \cdots & \mathrm{Cov}(X_2, X_n) \\
\vdots & \vdots & \ddots & \vdots \\
\mathrm{Cov}(X_n, X_1) & \mathrm{Cov}(X_n, X_2) & \cdots & \mathrm{Cov}(X_n, X_n)
\end{pmatrix}$

Mathematically, for a dataset containing p numerical variables, the **covariance matrix** $Σ$ is a p×p square and symmetric matrix that describes how each variable varies with every other variable.

- **Diagonal elements** $Σii=Var(Xi)$
These represent the **variance** of variable $X_i$. They measure how much each feature spreads around its mean.
- **Off‑diagonal elements** $Σij=Cov(Xi,Xj) for i≠j$
These represent the **covariance** between variables $X_i$ and $X_j$. They indicate whether the two variables tend to increase together (positive covariance), move in opposite directions (negative covariance), or are linearly independent (covariance near zero).

In this context, the covariance matrix $\Sigma$ acts as the mathematical shape that dictates how random noise must be shaped to protect privacy without destroying the dataset's analytical value.

#### Synthetic methods

Randomly generate data in such a way that some statistics or relationships of the original data are preserved. They have two different approaches: 

- Partially synthetic data where only the most sensitive data are replaced by synthetic version before released and the rest is preserved.
- Hybrid data where original data and synthetic data are combined

## Privacy Notions

### K-anonymity

According to Samarati & Sweeney, “a data set is said to satisfy k-anonymity if each combination of values of the quasi-identifier attributes in it is shared by at least k records (k-anonymous class).”

**Drawback**

Sensitive attributes lack diversity in values. They could suffer homogeneity attack. They have demographic or additional background knowledge which means that for example, female do not have prostate cancer, which can give supplementary information for attackers. 

#### $l$ **diversity**

According to Ninghui, an equivalence class is said to have $l$-diversity if there are at least $l$ “well-represented” values for the sensitive attribute. A table is said to have $l$-diversity if every equivalence class of the table has $l$-diversity.

**Drawback**

It doesn’t prevent from semantical closeness of sensitive attributes. Moreover, if the relative frequency of a value within a cluster differs widly from the overall one, a possibly more sensitive value can be strongly predicted for a target. 

#### $t$-closeness

According to Li, “An equivalence class is said to have t-closeness if the distance between the distribution of a sensitive attribute in this class and the distribution of the attribute in the whole table is no more than a threshold $t$. A table is said to have $t$-closeness if all equivalence classes have $t$-closeness”

### Differential Privacy

Let A be a randomized algorithm. For any two neighbouring
databases $𝐷_1$, $𝐷_2$ any subset $𝑆$ ⊆ 𝑟𝑎𝑛𝑔𝑒(𝐴) , A is $\epsilon$-differentially private iff
$Pr[𝐴(𝐷_1)∈ 𝑆 ]$ ≤ $𝑒^𝜀 Pr[𝐴(𝐷_2) ∈ 𝑆 + \delta]$

In the differential privacy, database will be a set of rows corresponding to different persons. The neighboring dataset will differing by one element only. Algorithm A maps randomized outcome. 

If $\delta$ = 0, it will be a differential privacy. 

**Example**

**Randomized Response [Warner65]**

It uses for survey on sensitive information. Some subjects are allowed to lie (in a controlled manner). 

Algorithm

- Flip a coin
- If heads: say the truth
- If tails: Flip a coin
If heads: “Yes”
If tails: “No

**Laplace mechanism**

It adds noise with particular shape. It’s suitable to numerical outcomes. 

Algorithm

- Let $f$ be the actual function
- Compute $f(D)$
- Add noise and output: $f(𝐷) + 𝐿𝑎𝑝(Δ𝑓/\epsilon)$

### Sensitivity of a function

Let $𝐷_1$ and $𝐷_2$ be two different datasets
$Δ𝑓 = 𝑚𝑎𝑥𝐷_1,𝐷_2{|𝑓(𝐷_1) − 𝑓(𝐷_2)|}_1$ 
It measures how much one person can affect output. 

## Searchable encryption

Searchable Encryption (SE) is a cryptographic primitive that addresses a fundamental dilemma in cloud computing: how to search through encrypted data without decrypting it on an untrusted server. When outsourcing sensitive data to a cloud provider (e.g., AWS, Google Cloud), standard encryption prevents the server from reading the data, but it also prevents the server from performing queries (like keyword lookups or range queries). Searchable Encryption bridges this gap by enabling servers to execute queries directly on ciphertext while keeping both the data and the query terms confidential.

### Symmetric Searchable Encryption

In Symmetric Searchable Encryption, a single secret key (or a set of symmetric keys derived from a master key) is used for both encryption and search token generation. Typically, this means the data owner is also the data consumer (or part of a tightly controlled closed group sharing the same keys).

It is composed of: 

- Key generation: Keygen($k$) = $K$, it takes a security parameter $k$ as input and outputs a secret symmetric key $K$
- Encryption of word $W_i$: Encrypt$(K, W_i) = C_i$
- Trapdoor generation for word $W$:

Trapdoor$(K, W)=T$, it takes a secret key $K$ and a target search word $W$ to generate a secure search token, or trapdoor T. This trapdoor is sent to the untrusted server so it can execute the query without learning the underliyng keyword $W$. 

- Search if $C_i$ corresponds to $W$

$Test(T, C_i) = b, b ∈ {0, 1}$

The security requirements are that 

- Cloud cannot discover content of records
- Cloud cannot distinguish between records
- Cloud cannot discover queried keyword

### SWP00

SWP stands for song, wagner and perrig. It’s a searchable encryption scheme. The goal is to transform a plaintext word $W_i$ into a secure ciphertext $C_i$ using a combination of deterministic encryption, pseudo-random functions (PRFs), and a stream cipher. 

For each word $W_i$ in the dataset, the client performs these steps: 

- Deterministic encryption ($DE$), while the word is encrypted deterministically using a key $K_{DET}$
- The resulting deterministic cyphertext $DE(W_i)$ is split into two halves: $L_i$ and $R_i$.
- A stream cipher using key $K_{PRG}$ generates a sequence of random bits denoted as $S_i$
- $L_i$ is passed through a pseudo-random function using $K_{PRF}$ to generate a subkey $K_i$.
- Another PRF evaluation computes a pseudo-random value $V_i = F(K_i, S_i)$ using the subkey $K_i$ and the random bits $S_i$.
- Finally, the combined blocks $S_i || V_i$ and XORed with $L_i || R_i$ to produce the final ciphertext $C_i$.

**Why use a stream cipher?** 

Even though the initial step involves deterministic encryption ($\text{DE}$), the integration of the stream cipher introduces fresh randomness ($S_i$) for each encryption operation. This ensures randomized encryption, meaning identical words appearing in different places will yield different ciphertexts, preventing an observer from tracking keyword frequencies directly from the ciphertext structure. 

#### SWP-trapdoor

When the client wants to search for a specific word $W$ without revealing it to the untrusted server, they execute the following steps to construct the search token:

- The target search word $W$ is encrypted deterministically using the secret key $K_{DET}$
- The resulting deterministic ciphertext DE($W$) is divided into two halves, $L$ and $R$.
- The left part is passed through a PRF using the secret key $K_{PRF}$ to generate the subkey $K$.

The resulting component form the trapdoor. 

### Public key encrypted keyword search

While Symmetric Searchable Encryption (SSE) requires both the data sender and receiver to share the same secret key, Public-Key Encryption with Keyword Search (PEKS) pioneered by Boneh, Di Crescenzo, Ostrovsky, and Persiano solves a different problem: allowing anyone to encrypt data and keywords using a receiver's public key, while only the receiver (holding the private key) can generate search trapdoors.

### Bilinear pairings

#### Diffie Hellman protocol

Alice and Bob agree on two parameters: $p$ a large prime number and a generator $g$. These parameters can be known to everyone. Then, Alice will choose a random private integer $a$ which is her private key and Bob will choose a random private integer $b$ which is his private key. Then, Alice computes her public value $A = g^a$ and Bob computes his public key value $B = g^b$. Alice receives $B$ and computes $s = B^a$ and Bob receives $A$ and computes $s = A^b$. 

#### Bilinear pairings

They are a powerful mathematical tool that revolutionized public-key cryptography. They provide a bridge between two different algebraic groups, enabling advanced cryptographic primitives that were previously considered impossible, such as Identity-Based Encryption (IBE), short signature schemes (e.g., Boneh-Lynn-Shacham / BLS), and advanced searchable encryption primitives like Public-Key Encrypted Keyword Search (PEKS).

**Core properties of the pairings** 

$e : G$  x $G → G_T$

For any elements $g^a$ and $g^b$ in group $G$, the pairing satisfies $e(g^a, g^b) = e(g^b, g^a) = e(g, g)^{ab}$

#### PEKS

PEKS stands for public key encrypted keyword search.

During the key generation which is the first step, the receiver picks a generator $g ∈ G_1$ and a random scalar $a ∈ Z^*_{p}$. It outputs the public key $PK =$  {${g, g^a}$} and the secret key $SK$ = a. 

Then, to encrypt a keyword $w_i$, the sender picks a random value $r \in \mathbb{Z}_p^*$. The sender hashes the keyword using a cryptographic hash function $H_1$, and computes the ciphertext components: $g^r$ and $t = e(H_1(w_i), g^{ar})$. It outputs the keyword ciphertext tuple $c = (c_{i1}, c_{i2}) = (g^r, t)$, which is stored on the untrusted server. 

When the receiver wants to search for a query word $w$, they compute the search token (trapdoor) using their secret key $a$:

$T = H_1(w)^a$. 

The receiver sends $T$ to the server. Because the server does not know $a$, it cannot reverse $T$ to find the original keyword $w$. 

The server checks whether a stored ciphertext component $c_{i2}$ matches the trapdoor $T$ by evaluating the pairing equality: $e(T, c_{i1}) == c_{i2}$. If the encrypted keyword matches the queried word ($w_i = w$), the pairing expansion yields:

$e(T, c_{i1}) = e(H_1(w)^a, g^r) = e(H_1(w), g)^{ar} = e(H_1(w), g^{ar}) = c_{i2}$

### Index based searchable encryption

Index-Based Searchable Encryption represents a major evolutionary leap toward sub-linear search performance ($O(R)$ time, where $R$ is the number of matching documents). Instead of encrypting documents word-by-word and forcing the server to evaluate every record, index-based schemes decouple the document storage from the search mechanism by introducing an encrypted search index.

The client processes the document collection to create an inverted index, which maps each distinct keyword $W$ to the identifiers of the documents containing it. Then, the client encrypts the entries of this index using cryptographic primitives (such as pseudorandom permutations, keyed hash functions, or encrypted arrays/lists) so that the server can look up entries securely.

Both the encrypted documents and the encrypted index are uploaded to the untrusted cloud server.

## **Homomorphic encryption 28/73**

## **Privacy and security for Machine Learning**

It’s possible to classify vulnerabilities by colour: 

- White-box: attacked ML is completely known
- Black-box: No knwledge of the attacked ML system
- Gray-box: partial knowledge of the attacked ML system

Attack could be during training, testing and inference. Attack can be targeted or untargeted. 

#### Poisoning attack

A poisoning attack is a training-time security threat where an adversary deliberately injects, modifies, or crafts malicious samples into the training dataset or update pipeline to compromise the integrity or availability of a machine learning model. Poisoning attacks are generally classified across three primary dimensions: Attack Intent, Attacker Knowledge, and Manipulation Strategy. Maliciously modifying training data to perturb training. Model looses accuracy in such type of data

#### Evasion attack

It is an inference-time security threat where an adversary subtly manipulates input data to deceive a deployed machine learning model, causing it to output an incorrect prediction.

Unlike poisoning attacks (which target the training phase), evasion attacks leave the model's learned parameters untouched. Instead, they exploit vulnerabilities in the model's decision boundaries by crafting inputs that sit close to the boundary line.

#### **Model extraction**

Steal the intellectual property, proprietary parameters, or decision logic of a trained machine learning model exposed via an API. The adversary treats the target model as a black box, submitting carefully crafted query vectors and recording the corresponding outputs (such as predicted labels, logits, or probability scores). They then use this collected input-output dataset to train a surrogate model that replicates the original model's behavior and decision boundaries.

#### **inversion attacks**

Reconstruct private or sensitive features of the training data (such as facial images, medical records, or personal attributes) associated with a specific output class. Given access to the model's output confidence scores for a target class (e.g., a specific person's identity), the attacker uses optimization techniques (such as gradient descent) to iteratively generate an input image that maximizes the model's prediction confidence for that class.

#### **Membership Inference Attacks (MIA)**

Determine whether a specific individual data record was included in the target model's training dataset. Membership inference attacks exploit the fact that machine learning models tend to behave differently on data they have seen during training versus unseen data. Because models naturally exhibit slightly higher prediction confidence, lower loss, and smaller entropy on training samples (due to minor overfitting), an attacker can train shadow classifiers or analyze output confidence vectors to distinguish "members" from "non-members.”

#### **Property Inference Attacks**

Infer global statistical properties, distribution characteristics, or sensitive subgroup attributes of the training dataset that are not directly disclosed by the model's primary outputs. Rather than targeting individual records, the adversary wants to learn broader traits (e.g., "What percentage of the training population was female?" or "What was the ratio of smokers to non-smokers in a medical model's training set?"). Attackers typically train a set of shadow models on datasets with varying property ratios and build a meta-classifier to map model parameters or output variations to the target property.

#### Reconstruction Attacks

Recover entire, original training samples (such as images, text sequences, or audio clips) from intermediate model representations, shared gradients, or latent features. Most prominently studied in **Federated Learning (FL)** through techniques like *Deep Leakage from Gradients (DLG)* or *Improved DLG (iDLG)*. When a participant shares their local gradient updates with a central server, an adversary can formulate an optimization problem to invert the gradients, matching them against dummy data until the exact input images or tokens used in training are mathematically reconstructed.

#### Prompt Injection Attacks

Exploit vulnerabilities in large language models (LLMs) by injecting malicious instructions into input text, tricking the model into bypassing safety guardrails, ignoring system prompts, or executing unauthorized actions.

## Privacy Enhancing Technologies

Technologies that enable the collection, analysis and sharing of information while protecting data
confidentiality and privacy. They complement frameworks on the protection of privacy and confidentiality but need clear guidance to tackle regulatory and technical challenges

**What is federated learning?** 

It is a decentralized machine learning approach that enables multiple parties (such as edge devices, smartphones, hospitals, or organizations) to collaboratively train a shared global model **without sharing their raw local data**.

The central server initializes a baseline global model (e.g., with random weights or a pre-trained base) and sends this model architecture and current parameters to a subset of participating clients. Each client trains the global model locally using its own private, localized dataset for a few epochs. Instead of uploading raw data, each client computes the changes made to the model, either as model parameter weights or gradient updates ($\Delta w$). The central server collects the updates from all participating clients and aggregates them (commonly using algorithms like FedAvg or Federated Averaging, which computes a weighted average based on client dataset sizes) to update the global model.Iteration: The updated global model is sent back out to the clients, and the cycle repeats until convergence. 

#### Differential privacy

It is a mathematical framework that guarantees that the output of a data analysis, query, or machine learning model reveals virtually nothing about whether any specific individual's data was included in the underlying dataset.

#### Multi-parti computation

Secure Multi-Party Computation (MPC) is a subfield of cryptography that enables multiple distinct parties to jointly compute a function over their private, distributed inputs without any party ever revealing their input to the others.

#### Fully Homomorphic Encryption

Fully Homomorphic Encryption (FHE) is a cryptographic scheme that permits arbitrary computations (both additions and multiplications an unlimited number of times) on encrypted data without requiring decryption. When the resulting ciphertext is eventually decrypted by the keyholder, the output matches the exact result as if the operations had been performed directly on the plaintext.

#### Trusted Execution Environment

A Trusted Execution Environment (TEE) is a secure and isolated space within a system's processor and main memory. It ensures that the code and data executed within it benefit from a high level of confidentiality and integrity, even in the face of a compromised operating system or hypervisor. 

#### **Robust Access Control**

Robust access control in modern cryptographic and distributed systems moves beyond traditional server-enforced access control lists (ACLs) to cryptographically enforced access control . Even if an underlying cloud storage provider or server is fully compromised, data remains protected by mathematical policies tied directly to the ciphertext. They have different authentication for different accessing needs. 

#### Adversarial training

Adversarial Training is widely regarded as the most effective empirical defense mechanism against evasion (adversarial) attacks in machine learning. Rather than treating input data as static and trustworthy, adversarial training proactively hardens a model during its training phase by forcing it to learn correct classifications over worst-case, intentionally perturbed data.

### **ML based mitigation**

They are strategies designed to defend models against various security and privacy threats. 

1. **Data sanitization** mitigates poisoning attacks and evasion attacks by increasing database size overhead and requiring significant computational time for data cleaning. 
2. **Adding more data** results in increased training time and introduces logistical challenges regarding data acquisition. It mitigates poisoning attacks and evasion attacks 
3. **Model hardening** fights model extraction and model inversion attacks by negatively impacting overall system efficiency. 
4. **Regularization** fights membership inference attacks, property inference attacks, and reconstruction attacks by reducing overfitting and smoothing generalization gaps. It may introduce a slight penalty on model accuracy on clean validation and test data. 

## Fairness in big data

Fairness can be described as a treatment, a behaviour or an outcome that respects established facts, beliefs and norms and is not determined by favouritism or unjust
discrimination, while bias is the ”systematic difference in treatment of certain objects, people, or groups in
comparison to others.

Classification and clustering algorithms cannot function without bias. However, the impact of this bias could be positive, neutral or negative according to the system goals and objectives. 

- Positive effect: AI developers can introduce bias to ensure a fair result
- Neutral effect: bias does not impact the outcome of the system
- Negative effect: bias can have unintended consequences of limiting the opportunities of those affected.

One challenge with determining the relevance of bias is that what constitutes negative effect can depend on the specific use case or application domain.

Within the context of AI, fairness can happen in different context like: 

- unfair allocation which occurs when an AI system unfairly extends or withholds opportunities or resources for some groups
- unfair quality of service happens when AI system performs less well for some groups
- stereotyping reinforces existing social stereotypes
- denigration is when the AI system behaves in ways that are derogatory or demeaning
- over or under representation and erasure is when the AI system over or under represents some groups.

Achieving fairness in systems often means making trade-offs. They exist many of unwanted bias such as human cognitive bias, data bias, bias introduced by engineering decisions, and data characteristics. 

For example, human cognitive bias could happen through automation bias. It occurs when a human favours recommendations made by an automated system over information made without automation, even when the automation makes errors.

Data bias could happen through statistical bias, which occurs when data records are not collected randomly from the intended population. 

**Missing features and labels**

Real world data are rarely complete. In particular, features are often missing from individual training samples. If the frequency of missing features is higher for one group
than another then this presents another vector for bias.

**Simpson’s Paradox**

Simpson's paradox is a statistical phenomenon where a trend observed in several data groups disappears or reverses when those groups are combined.

Let's imagine a study comparing the effectiveness of two medical treatments (A and B) on 1,000 patients:

- **Treatment A:** Administered mainly to severe cases. It cures 78% of severe cases and 93% of mild cases, for an overall cure rate of **83%** .
- **Treatment B:** Administered mainly to mild cases. It cures 73% of severe cases and 90% of mild cases, for an overall cure rate of **87%** .

Overall, treatment B appears more effective (87% vs. 83%). However, if we look at it by patient category:

- In **severe cases** , treatment A is more effective (78% vs 73%).
- In **mild cases** , treatment A is also more effective (93% vs 90%).

The paradox arises in the presence of a confounding variable (here, the initial severity of the disease). Treatment B appears better overall only because it was administered predominantly to patients with mild cases, who recover more easily anyway.

In data science and statistics, this underlines the crucial importance of controlling for hidden variables before drawing hasty conclusions from an aggregated dataset.

**How MLE tends to increase bias?** 

Given that ML often uses functions like a maximum likelihood estimator (MLE) to determine parameters, if there is under-representation present in the data, the MLE tends to amplify any underlying bias in the distribution. That is, the ML model learns patterns proportional to what it sees most often in the training data. As a result, minority patterns get underrepresented because they contribute less to the overall likelihood function.

**How to assess fairness?** 

Metrics of fairness seek to evaluate differences between average observed values and true values. Protected groups should be treated similarly. However, there are different fairness notions used in machine learning:

- Demographic Parity;
- Equality of Opportunity;
- Equalized Odds;
- Predictive equality.

We can introduce a confusion matrix among positive and negative impact: 

> 
> 
> 
> <img width="293" height="170" alt="image 4" src="https://github.com/user-attachments/assets/7a5cf565-9589-4130-a3f4-d1edb0897828" />
> 

#### Demographic parity

A binary classification or decision-making system satisfies demographic parity if the probability of receiving a positive outcome (e. g., being hired, getting a loan, or being admitted) is independent of sensitive protected attributes (such as race, gender, age, or socioeconomic status).

Mathematically, let $Y∈{0,1}, Y∈{ 0 ,1 }$ be the binary decision (where 1 is the favorable outcome), and let A be the sensitive attribute (where A = 1 represents a protected group and A=0 represents an unprivileged or baseline group). Demographic parity requires:

$$
\mathbb{P}(\hat{Y} = 1 \mid A = 1) = \mathbb{P}(\hat{Y} = 1 \mid A = 0)
$$

**The selection rate (or acceptance rate) must be equal across all demographic groups.** If 40% of applicants from the general population receive a positive outcome, approximately 40% of applicants from the protected group must also receive a positive outcome.

#### Equalized odds

$$
\mathbb{P}(\hat{Y} = 1 \mid A = 1, Y = y) = \mathbb{P}(\hat{Y} = 1 \ A = 0, Y = y)
$$

Equalized **Odds** is a machine learning fairness metric that ensures a classification model treats different demographic groups with the same level of accuracy, *controlling* for both false positives and false negatives.

**Who is wrongly flagged?** 

A false positive occurs when a student who did not cheat is wrongly flagged. The false-positive rate is calculated among students who did not cheat:

$FPR = FP / FP + TN$

**Who is correctly flagged?** 

A true positive occurs when a student who cheated is correctly flagged. The true-positive rate is calculated among students who cheated:

$TPR = TP / TP + FN$

#### Equality of opportunity

Unlike demographic parity, which imposes identical overall selection rates, equality of opportunity recognizes and respects the differences in actual qualifications between groups.

$$
\mathbb{P}(\hat{Y} = 1 \mid A = 0, Y = 1) = \mathbb{P}(\hat{Y} = 1 \mid A = 1, Y = 1)
$$

## Sources

1. https://www.geeksforgeeks.org/computer-networks/difference-between-symmetric-and-asymmetric-key-encryption/
2. https://www.youtube.com/watch?v=AQDCe585Lnc
3. https://www.youtube.com/watch?v=o_g-M7UBqI8
4. https://www.youtube.com/watch?v=kPPFNrlN3zo
5. https://www.youtube.com/watch?v=j6wwBqfSk-o&t=75s
6. https://www.youtube.com/watch?v=gI0wk1CXlsQ
7. https://www.youtube.com/watch?v=NmM9HA2MQGI
8. https://www.youtube.com/watch?v=ebEkn-BiW5k
