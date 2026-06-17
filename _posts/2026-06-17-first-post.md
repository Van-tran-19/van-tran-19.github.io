---
title: "First Post"
date: 2026-06-16 10:00:00 +0200
categories: [Blog]
tags: [AI, ML]
---
My Journey Toward Becoming an LLM Engineer (2026–2028)
Over the next two years, I’m committing myself to a clear and ambitious goal: becoming a Large Language Model Engineer capable of contributing to cutting‑edge AI systems — and ultimately joining a company like Mistral AI.
This blog will be my public record of that journey. I’ll share the skills I’m learning, the projects I’m building, the papers I’m studying, and the lessons I pick up along the way.

Why I’m Pursuing This Path
What fascinates me about LLM engineering is the intersection of deep learning, systems design, and large‑scale optimization. Modern language models are not just neural networks; they are complex software systems. They involve distributed training pipelines, inference engines, retrieval architectures, and a deep understanding of both mathematics and engineering.

My goal is to become the kind of engineer who can not only train models, but also optimize them, deploy them, evaluate them, and understand them at a fundamental level.
This blog is my way of staying accountable — and hopefully helping others who are on a similar path.

Building the Foundations (2026–2027)
Before diving into advanced LLM systems, I need a solid engineering base. That starts with Python at an engineer‑level: type‑safe code, clean logging, modular design, robust error handling, asynchronous programming, multiprocessing, and proper unit testing with pytest. These are the habits that make large projects maintainable.

On the machine learning side, I’m reinforcing the fundamentals: linear and logistic regression, SVMs, Random Forests, XGBoost, cross‑validation, overfitting and underfitting, and evaluation metrics such as F1, ROC curves, and perplexity.
These concepts are essential because they teach how models learn, fail, and generalize.

Deep learning with PyTorch is the next layer. I’m focusing on tensors, autograd, optimizers, training loops written from scratch, classical architectures (CNNs, RNNs, MLPs), and regularization techniques.
This foundation will support everything I do with LLMs later.

Mastering LLM‑Specific Skills
The core of my roadmap is dedicated to understanding and building systems around large language models.

I’m studying the Transformer architecture in depth: attention mechanisms, multi‑head attention, rotary embeddings (RoPE), LayerNorm, residual connections, KV caching, and the ideas behind FlashAttention.
These are the building blocks of every modern LLM.

Fine‑tuning is another major area. I’ll work with SFT, LoRA, QLoRA, and PEFT, and learn how to structure datasets and evaluate models before and after fine‑tuning.
This is essential for adapting open‑source models to real‑world tasks.

Retrieval‑Augmented Generation (RAG) is also a priority. I want to understand chunking strategies, embeddings, vector databases like FAISS and Milvus, reranking, multi‑source retrieval, and evaluation methods.
RAG is becoming a core component of production LLM systems.

Inference optimization is equally important. Tools like vLLM, quantization methods (GPTQ, AWQ), dynamic batching, optimized KV caching, and streaming responses all play a role in making LLMs fast and affordable.

Finally, I’ll dive into distributed training: DDP, FSDP, DeepSpeed, sharding, mixed precision (FP16/BF16), and GPU profiling.
These skills are required to train and scale models efficiently.

Learning Systems & DevOps for Production‑Grade AI
LLM engineering isn’t just about models — it’s about systems.

I’ll learn Docker, FastAPI, Kubernetes basics, monitoring with Prometheus and Grafana, and cloud platforms like AWS or GCP.
These tools are essential for deploying and maintaining real AI applications.

On the software architecture side, I’ll focus on writing modular code, using clean design patterns, managing configurations with Hydra or YAML, and building reproducible pipelines.
Good engineering practices matter as much as good models.

The Projects I Plan to Build
To turn theory into practice, I’ve mapped out a series of projects across the next two years.

In 2026, I’ll build a full RAG pipeline, experiment with LoRA fine‑tuning on a custom dataset, create a simple LLM agent, and write an optimized inference script.

In 2027, I’ll tackle more advanced challenges: multi‑source RAG with reranking, a multi‑model benchmark (Mistral vs LLaMA vs Qwen), a vLLM inference API, and a mini‑Transformer implemented from scratch.

In 2028, I aim to reach “Mistral level”: training a small LLM (125M–350M), building a complete pipeline from data to inference, and designing a full LLM system with RAG, agents, monitoring, and a UI — along with advanced GPU optimizations.

These projects will form the backbone of my portfolio.

Contributing to Open‑Source
Open‑source is one of the best ways to get noticed in the AI world.
I plan to contribute to projects like the Mistral Cookbook, HuggingFace Transformers, LlamaIndex, LangChain, vLLM, and xFormers.

My contributions will range from documentation and notebooks to inference scripts, fine‑tuning examples, bug fixes, and small optimizations.

My goal is to make 3–5 PRs in 2026, and 10+ PRs in 2027–2028.

My Internship Goal (2027)
In 2027, I want to secure an internship that aligns with my LLM ambitions.
My target organizations include INRIA, HuggingFace, Meta FAIR Paris, AI startups in Paris or Sophia, and scale‑ups like Alan, Doctolib, or Qonto.

The internship must involve real LLM work: RAG, fine‑tuning, inference optimization, PyTorch, and distributed systems.

Building a Public Presence
While optional, I believe sharing my work publicly can be powerful.
I plan to post technical threads on X, write Medium articles, explain papers, publish demos, and share benchmarks.

What impresses people most is not just using models, but understanding them deeply — reproducing papers, implementing Transformer modules, and comparing open‑source models.

GitHub as My Real CV
By 2028, I want my GitHub to reflect the engineer I’ve become:
a collection of serious LLM projects, open‑source contributions, clean and typed code, detailed READMEs, and reproducible demos.

This is what recruiters will look at.

The Road Ahead
This blog marks the beginning of a long journey.
I’m not simply “learning AI” — I’m training to become an engineer capable of building, optimizing, and understanding large‑scale language models.

If you’re interested in LLMs, deep learning, or AI engineering, feel free to follow along.
I’ll share everything I learn — the successes, the failures, and the breakthroughs.

This is just the start.