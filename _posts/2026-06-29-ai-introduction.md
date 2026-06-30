---
title: "An Introduction to AI"
date: 2026-06-29 17:30:00 +0200
categories: [AI]
math: true
tags: [AI, Engineering, LLM, Transformers, Inference, AI generative, AI discriminative]
---
# AI engineer

## What is AI engineering?

AI models have gotten dramatically better at solving real problems, and the barrier to building them has gotten much lower. Today, AI engineering is about building applications on top of foundation models rather than training models from scratch like a traditional ML engineer. Because foundation models use self-supervised learning, they bypass the massive bottleneck of manual data labeling, leading to the explosion of LLMs and large multimodal models. Nowadays, models power coding assistance (GitHub Copilot), writing aids, and customer support bots. [1]

## Foundation Models and architectures

One of the key concepts in foundation models is that if a model hasn’t seen, for example, a specific language or concept during training, it simply doesn’t have that knowledge. Furthermore, models are trained on what they see, which leads them to learn from biased data, making the models biased as well. The principal datasets used to train models are business and industry, technology, news and media, and art. Most models use transformer architectures based on the attention mechanism. [1]

### What are Transformers?

A Transformer is a deep learning architecture built around a mechanism called self-attention, which allows a model to understand relationships between words (or tokens) *no matter how far apart they are.* The transformer architecture is built from stacked blocks, each containing [4]:

- Multi-head self-attention (MHSA): the model attends to different types of relationships in parallel.
- Feed-forward network (FFN): a small MLP applied to each token independently.
- LayerNorm + residual connections: stabilize training and allow deep stacking.
- Positional embeddings: because attention alone has no sense of order.

Nowadays, transformers are used in LLMs, vision transformers, audio models, multimodal models, reinforcement learning, and protein folding. [4]

During inference, transformers work with a prefill concept, which processes all input tokens in parallel to create the intermediate state, and a decode process that generates one output token at a time. [4]

### What is self-attention?

Self attention answers one question: for a given token, which other in the sequence token import the most? It computes **weights** between all pairs of tokens so the model can focus on the right relationships. It uses three types of vectors[4]: 

- Query vectors $Q$: represent what information the model is looking for
- Key vectors $K$ that like indexes to previous tokens
- Value vectors $V$ that is the actual content of the previous state

Attention mechanism computes: 

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

Meaning:

- $QK^T$ = similarity between tokens
- softmax = turns similarities into weights

$$\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}$$

- multiply by $V$ = weighted sum of information

This gives each token a **context‑aware representation**.

### What is a RNN?

An **RNN (Recurrent Neural Network)** is a neural network designed to process **sequences** (text, audio, time series) by keeping a **hidden state** that carries information from previous steps.
An RNN processes a sequence **one token at a time**, and at each step it updates a **hidden state**:

$$h_t = f(h_{t-1}, x_t)$$

Where:

- $x_t$ = input at time $t$
- $h_t$ = hidden state (memory)
- $f$ = a small neural network

The hidden state acts like a **memory** of everything seen so far.
RNNs transport a weak memory; each step overwrites part of the previous knowledge.

**RNN is the ancestor of transformers and is no longer efficient. Why?**

They have major limitations that make them obsolete. They cannot handle long-range dependencies. Gradients must flow through hundreds of time steps, and RNNs easily forget information from far earlier in the sequence.

## The two great tasks of AI

AI can be discriminative or generative depending on its role.

### Discriminative AI

It learns to separate data into categories. It makes a decision or judgement. They are designed to learn a direct mapping from inputs to outputs, or more formally, to model the conditional probability $p(x|y)$. Few examples include digit recognition, image classification, object segmentation (autonomous driving), object detection, 3D reconstruction, etc.

A mathematical view: 

We consider a joint distribution over inputs and outputs $(X,Y) \sim p(x,y)$. Discriminative modeling learns the conditional probability distribution $p(y|x)$ or a joint predictor $f(x)$ directly. The distribution captures the likelihood of observing an output y given an input x. This is achieved by defining a family of parameterized probability distributions $q_\theta(y|x)$ where the model $f_\theta(x)$ determines the key parameters of the distribution. One of the main examples are 

- regression (which can predict patient recovery time, stock price forecasting, house price estimation…)
- classification (which can be used to spam detection, medical imaging, image recognition)

In discriminative modeling, the goal is to train a model whose $q_\theta(y|x)$ is as close as possible to the true distribution $p_\text{true}(y|x)$ and the standard measure to measure for the distance between 2 probability distributions is Kullback-Leibler Divergence (KL div). 

$$\mathrm{KL}\!\left(p_{\text{true}} \,\|\, q_{\theta}\right) = \underbrace{\mathbb {E}_{y \sim p_{\text{true}}}\!\left[-\log p_{\text{true}}(y \mid x)\right]}_{\text{Entropy of true data}} + \underbrace{\mathbb{E}_{y \sim p_{\text{true}}}\!\left[-\log q_{\theta}(y \mid x)\right]}_{\text{Cross-Entropy}}$$

Then, objective of learning is to find the model parameters $\theta$ that minimize the KL div, averaged over all the possible inputs: 

$$\theta^* = \arg\min_{\theta} \, \mathbb{E}_{x \sim p_{\text{true}}(x)} \left[ \mathrm{KL}\big(p_{\text{true}}(y \mid x) \,\|\, q_{\theta}(y \mid x)\big) \right]$$

This shows that minimizing the KL div is equivalent to minimizing the cross entropy. This is the principle of MLE (maximum likelihood estimation)

$$\theta^* = \arg\min_{\theta} \, \mathbb{E}_{(x,y)\sim p_{\text{true}}}\!\left[-\log q_{\theta}(y \mid x)\right]$$

We approximate the true risk using our finite training dataset of n samples. This approximate is called the Empirical Risk. 

$$\hat{R}(f) = \frac{1}{n} \sum_{i=1}^{n} \ell\!\left(y_i,\, f(x_i)\right)$$

The best parameters are find by minimizing the empirical average: the ERM (Empirical Risk Minimization) procedure: 

$$\theta^* \approx \arg\min_{\theta} \, \frac{1}{n} \sum_{i=1}^{n} \left[-\log q_{\theta}(y_i \mid x_i)\right]$$

the term $\ell(y \mid \hat{y}) = -\log q_{\theta}(y_i \mid x_i)$ is the loss function for a singe data point. 

Ultimately, the true goal is to minimize the Population Risk: 

$$\mathcal{R}(f) = \mathbb{E}_{(X,Y)\sim p_{\text{true}}}\!\left[\ell\!\left(Y,\, f(X)\right)\right]$$

An good indactor to know if the model generalizes well is the gap, as small as gap is: 

$$\text{Gap}(f) = \mathcal{R}(f) - \widehat{\mathcal{R}}(f)$$

By the law of large numbers: with enough representative data, training performance approaches true performance. 

#### Advanced Non-Linear models

Neural networks are powerful function approximators composed of interconnected layers of nodes.  They are foundations of modern deep learning. The simplest one is the MLP: multi layer perceptron. [4]

#### Specialized architectures

- CNNs (Convolutional neural networks) are designed for data such as Images
- Sequence models (RNNs, Transformers) are used for sequential data like text or time series.
- Structured outputs with CNN [4]

### Generative AI

It learns the underlying structure of the data itself in order to create new, original data. They are designed to learn directly the joint probability $p(y,x)$. In a sence, there is no conceptual distinction between x’s and y’s. The core idea is the following:

Given a dataset of observations $\mathcal{D} = \{\, \mathbf{x}_i \,\}_{i=1}^{N}$ sampled from an unkown distribution $p_d(x)$, learn a parametric model $p_\theta(x) \approx p_d(x)$. First, the method requires density estimation: evaluate $p_\theta(\tilde{x})$ for a new $\tilde{x}$ and generation: draw new samples $x_\text{new} \sim p_\theta(x)$. Generative AI requires manifold hypothesis, high-dimensional data often concentrate near a lower-dimensional manifold; generative models aim to learn its structure. 

#### List of alternatives

Generative adversarial networks (GANs)

<img width="522" height="253" alt="image" src="https://github.com/user-attachments/assets/2a4c2920-1d78-4ee9-916c-1a1dbc537b5c" />

Variational autoencoders (VAEs)

<img width="321" height="368" alt="image" src="https://github.com/user-attachments/assets/a059de31-c67b-4fd9-8e53-f50b81cca8bf" />

## Model Selection

## Prompt Engineering

## RAG (Retrieval-Augmented Generation)

## Agents & Tool Use

## Fine-Tuning & Data Engineering

### What is backpropagation (gradient descent)?

This is where the data and the parameters come together. The principle is the following: 

$$w \leftarrow w - \eta \nabla L$$

- $w$ is the weight of the model
- $L$ is the loss function (how wrong the model is)
- $\nabla L$ is the gradient of the loss telling how much and in which direction the weight increases the error
- $\eta$ is the learning rate controlling how big the update step is [4]

### The danger of overfitting

Overfitting means **memorizing** the training data instead of **understanding** the underlying pattern. When a model becomes too flexible, it starts fitting noise instead of structure.

Training loss:

$$L_{\text{train}}(\theta) = \frac{1}{N} \sum_{i=1}^{N} \ell\!\left(f_{\theta}(x_i),\, y_i\right)$$

Testing loss:

$$L_{\text{test}}(\theta) = \mathbb{E}_{(x,y)\sim \mathcal{D}_{\text{test}}}\!\left[\ell\!\left(f_{\theta}(x),\, y\right)\right]$$

A large gap indicates overfitting: 

$$L_{\text{test}}(\theta^*) \gg L_{\text{train}}(\theta^*)$$

This mismatch can be caused by a distribution shift between the training data $D_\text{train}$ and the testing data $D_\text{test}$. [4]

## Inference Optimizations

### What is inference?

Inference is the phase where a trained AI model is actually put to work to make predictions, generate text, or solve problems based on new, unseen data. It must be dissociated from training. Inference is like taking an exam: the model applies what it learned to generate a useful result in real time.

### How inference is optimized?

Inference is limited by memory bandwidth: how fast the system can move data from memory to processors. Because LLMs generate text sequentially, the server has to reload the model’s parameters for every token.

To optimize inference, engineers use techniques such as:

#### Quantization

AI models are essentially giant mathematical matrices. By default, these numbers are stored as highly precise 32-bit decimals (FP32: floating points 32 bits). Quantization rounds these numbers down to 16-bit, 8-bit, or even 4-bit integers. The fact is that during training, values are stored using a range of $2*10^{38}-1$ numbers. The conversion at each step makes the values round up to the closest number, make it losing precision but in the same time, loading file much faster. [5]

In mathematical terms, quantization is the problem of representing a continuous random variable $X$ using a finite number of bits according to the Shannon’s theory (How do we represent a continous random variable by a finite number of bits?). If you are given $R$ bits to represent $X$, the representation $\hat{X}$can take on at most $2^R$ discrete values.

The mathematical process of quantization involves two main steps:

1. **Defining Regions:** Dividing the continuous space of the random variable into distinct regions.

If you have only 1 bits to reprensent some values. 1 bit corresponding to 0 or 1. It creates our 2 regions where we can classify those values. 

1. **Finding Code Points:** Finding the optimal set of discrete values (code points) for $\hat{X}$ to represent each region in order to minimize a specific distortion measure $D(R)$ which is the cost or error of representing $X$ by $\hat{X}$. 

**A Mathematical Example: 1-Bit Quantization** by applying a 1-bit quantization ($R=1$) to a Gaussian random variable $X \sim \mathcal{N}(0,\sigma^2)$:

- With 1 bit, you can represent $2^1=2$ values, so the real line is divided into a positive region and a negative region.
- The optimal code point for the positive real line is its centroid a

which is calculated using the conditional expectation be

$$E[X \mid X > 0] = \int_{0}^{\infty} x \frac{1}{\sqrt{2\pi\sigma^2}} e^{-\frac{x^2}{2\sigma^2}} \, dx = \sigma \sqrt{\frac{2}{\pi}}$$

- This means the quantized value becomes

$$\hat{X} = \sigma \sqrt{\frac{2}{\pi}} \quad \text{if } X \ge 0, \qquad \hat{X} = -\sigma \sqrt{\frac{2}{\pi}} \quad \text{if } X < 0$$

Where $\sigma = 1$: 

<img width="527" height="340" alt="image" src="https://github.com/user-attachments/assets/14cfc6f4-90fb-4b24-bf39-3985e63c00d8" />

It means that at $\hat{X} = \pm 0.79$, instead of using the entire continous values between $[-4; 4]$, all values are reshaped into two different areas: $+\hat{X}$ or $-\hat{X}$

- The resulting quantization error is calculated as

$$D(R) = E[(X - \hat{X})^2] = \left( \frac{\pi - 2}{\pi} \right) \sigma^2 \approx 0.36 \sigma^2$$

Distortion error is like the trade off

By replacing every value with either +0.79 or -0.79, the model saved a massive amount of memory, but it also made an error. Some value actually scored a +2.15, and some scored a +0.10, but it recorded them all as +0.79.

$D(R)$ is just calculating the "damage." It tells us that by forcing an entire infinite bell curve into just two numbers, our recorded numbers will be off from the real numbers by an average of 36%. [5]

## Source

1. [https://www.youtube.com/watch?v=JV3pL1_mn2M&t=2s](https://www.youtube.com/watch?v=JV3pL1_mn2M&t=2s)
2. [https://www.youtube.com/watch?v=dbUIjFXIpis](https://www.youtube.com/watch?v=dbUIjFXIpis)
3. AI engineering by Chip Huyen
4. Eurecom - Introduction to AI courses [https://eurecom-ds.github.io/intro-ai/](https://eurecom-ds.github.io/intro-ai/)
5. Eurecom - Information Theory courses
