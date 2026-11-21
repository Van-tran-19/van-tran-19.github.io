---
title: "Training a Tiny Transformer: SIMD, CPU Threading, and GPU Parallelism"
date: 2026-11-08 18:00:00 +0200
categories: [MachineLearning, Systems]
tags: [Transformers, GPT, CUDA, SIMD, PyTorch, Parallelism, GPU, NumPy, Performance]
toc: true
math: true
---
# Phase 2 — Intra-node parallelism: train your own tiny transformer

The goal is to train a small GPT while exercising the three layers of parallelism inside a single node CPU vectorization (SIMD), CPU threads, and GPU acceleration and measure where each stops helping.

## Learning

### What is Vectorization?

Vectorization consists of replacing explicit Python loops with operations on entire arrays/matrices at once. NumPy performs the whole operation internally in highly optimised C code. 

**Why is it faster?**

```sql
a = [1,2,3,4]
b = [5,6,7,8]
#A python loop does one operation after another.
1+5
2+6
3+7
4+8
#Vectorised NumPy does a + b and internally uses CPU instructions that process many numbers simultaneously.
a + b
#The result is the same but it's much faster

```

### What is SIMD?

SIMD means Single Instruction Multiple Data. One CPU instruction operates on many data values simultaneously. 

It’s the reason why `Q @ Kᵀ` on NumPy is ~100× a Python loop. 

```sql
scores = Q @ K.T
```

This performs matrix multiplication. For every pair of rows, it computes:

$score_{ij} = \sum_{k} Q_{ik} K_{jk}$

### **What is `numpy.linalg`?**

`numpy.linalg` is NumPy's **Linear Algebra** module.

It provides fast operations for:

```sql
A @ B          # matrix multiplication
np.dot(A,B)    # dot product
np.linalg.inv(A)
np.linalg.eig(A)
np.linalg.svd(A)
```

Numpy calls BLAS, OpenBLAS, MKL which are highly optimized numerical libraries.

Those libraries use:

- SIMD (AVX instructions)
- Multithreading
- Cache optimization
- Native machine code (C/Fortran)

### What are Threads / intra-op parallelism: PyTorch CPU threading?

A thread is a stream of execution. Imagine the CPU has 8 cores. With threading, cores work simultaneously. 

Intra-op means multiple threads cooperate on one operation. Because pytorch can split the work, all cores can contribute to the same operation. 

### What is the Amdahl's Law?

Some parts of a program cannot be parallelized. Even with infinite cores, if you have operations that can be computed only sequentially, this will create a speed limit. 

What are **GPU / CUDA / SIMT?** 

A GPU is designed for massive parallel computation. It has thousands of simpler cores.

CUDA is NVIDIA's platform for using the GPU for computation. 

SIMT stands for single instruction multiple threads. 

### What are safetensors?

`safetensors` is the file format that will store the weights of the GPT. During training, GPT learns millions of numbers.The trained model is essentially those weights.

## Practise

### **Task 1 — Run the unit tests**

### **Task 2 — Open the notebook in Colab with a GPU runtime**

https://colab.research.google.com/github/Van-tran-19/cloud/blob/main/phase-2-tiny-gpt/notebook.ipynb#scrollTo=cell2

```sql
import numpy as np, torch
from attention_numpy import attention_naive, attention_vectorized
from config import CONFIG
from model import GPT
import train, experiments

# 'cuda' only if Colab actually gave us a GPU runtime; everything else falls back to CPU.
device = 'cuda' if torch.cuda.is_available() else 'cpu'
print('device:', device, '| torch', torch.__version__)
#output
device: cuda | torch 2.14.1+cu130
```

#### Attention: naive loops vs vectorized (SIMD)

Both functions compute identical attention output the only difference is how they get there. `attention_naive` walks the sequence with Python loops, one dot product at a time. `attention_vectorized` hands whole matrices to NumPy, which dispatches to a BLAS library compiled with AVX/SIMD instructions that process several floats per CPU cycle.

`T=256` is the sequence length and `d=64` the head dimension, the same shapes the real model uses, so the timing is representative.

```sql
# Same maths, timed two ways. T = sequence length, d = head dimension.
t_naive = experiments.time_attention(attention_naive, T=256, d=64)
t_vec   = experiments.time_attention(attention_vectorized, T=256, d=64)
print(f'naive      : {t_naive:.2f} ms')
print(f'vectorized : {t_vec:.2f} ms   ({t_naive / t_vec:.1f}x faster)')

# Which BLAS is NumPy using? This is what makes the vectorized version fast.
np.show_config()
#output
naive      : 242.23 ms
vectorized : 0.98 ms   (247.9x faster)
Build Dependencies:
  blas:
    detection method: pkgconfig
    found: true
    include directory: /opt/_internal/cpython-3.13.8/lib/python3.13/site-packages/scipy_openblas64/include
    lib directory: /opt/_internal/cpython-3.13.8/lib/python3.13/site-packages/scipy_openblas64/lib
    name: scipy-openblas
    openblas configuration: OpenBLAS 0.3.31.188.0  USE64BITINT DYNAMIC_ARCH NO_AFFINITY
      Haswell MAX_THREADS=64
    pc file directory: /project/.openblas
    version: 0.3.31.188.0
  lapack:
    detection method: pkgconfig
    found: true
    include directory: /opt/_internal/cpython-3.13.8/lib/python3.13/site-packages/scipy_openblas64/include
    lib directory: /opt/_internal/cpython-3.13.8/lib/python3.13/site-packages/scipy_openblas64/lib
    name: scipy-openblas
    openblas configuration: OpenBLAS 0.3.31.188.0  USE64BITINT DYNAMIC_ARCH NO_AFFINITY
      Haswell MAX_THREADS=64
    pc file directory: /project/.openblas
    version: 0.3.31.188.0
Compilers:
  c:
    commands: cc
    linker: ld.bfd
    name: gcc
    version: 14.2.1
  c++:
    commands: c++
    linker: ld.bfd
    name: gcc
    version: 14.2.1
  cython:
    commands: cython
    linker: cython
    name: cython
    version: 3.2.4
Machine Information:
  build:
    cpu: x86_64
    endian: little
    family: x86_64
    system: linux
  host:
    cpu: x86_64
    endian: little
    family: x86_64
    system: linux
Python Information:
  path: /tmp/build-env-v_sw_7zo/bin/python
  version: '3.13'
SIMD Extensions:
  baseline:
  - X86_V2
  found:
  - X86_V3
  - X86_V4
  not found:
  - AVX512_ICL
  - AVX512_SPR

```

The vectorised NumPy implementation was approximately 248× faster than the naive Python implementation because matrix multiplication (`Q @ K.T`) was executed by OpenBLAS using highly optimised native code and SIMD instructions, avoiding the overhead of millions of interpreted Python loop iterations. 

#### Load the training corpus

Everything from here on needs training data, so load it once into `data` and reuse it.

The text is TinyShakespeare (~1 MB). If your repo already contains `input.txt` it is used directly; otherwise it is downloaded. `train.encode` turns the characters into integer token ids, and the result becomes one long 1-D tensor that later cells slice batches out of.

```sql
import os, urllib.request

# Prefer a local copy if the repo has one; otherwise fetch TinyShakespeare (~1 MB).
if os.path.exists('phase-2-tiny-gpt/input.txt'):
    text = open('phase-2-tiny-gpt/input.txt', encoding='utf-8', errors='replace').read()
else:
    url = 'https://raw.githubusercontent.com/karpathy/char-rnn/master/data/tinyshakespeare/input.txt'
    text = urllib.request.urlopen(url).read().decode('utf-8', 'replace')

# Encode characters -> token ids, as one long 1-D tensor to slice batches from.
data = torch.tensor(train.encode(text), dtype=torch.long)
print('corpus tokens:', len(data))
#output
corpus tokens: 1115394
```

### **Multi-threading: sweep CPU threads (intra-op parallelism)**

PyTorch splits a single CPU operation a matrix multiply, say across `torch.set_num_threads(n)` worker threads. This is *intra-op* parallelism: one operation, many cores, as opposed to running several operations at once.

The cell below builds a fresh model, defines one training step (fetch a batch, forward, backward), and times that step at 1, 2, 4 and 8 threads.

```sql
model = GPT(CONFIG)
print('parameters:', model.num_params())

# One training step: sample a batch, forward pass, backward pass. This is what gets timed.
def one_step():
    x, y = train.get_batch(data, CONFIG.block_size, 16, 'cpu')
    _, loss = model(x, y); loss.backward()

# Time that step at each thread count. Returns {'1': ms, '2': ms, ...}.
# threading_sweep restores the default thread count on the way out: the setting is
# global, and leaving it at 8 on a 2-vCPU machine would slow the CPU baseline in
# section 4 and overstate the GPU speed-up.
sweep = experiments.threading_sweep(one_step, threads=(1, 2, 4, 8))
print('threads -> ms:', sweep)
print('torch threads restored to:', torch.get_num_threads())
#output
parameters: 4935680
threads -> ms: {'1': 2229.977, '2': 2008.293, '4': 2125.72, '8': 2100.541}
torch threads restored to: 1
```

Increasing CPU threads produced only a small speedup. Moving from one thread to two threads improved performance, but using four or eight threads provided little additional benefit. This illustrates the limits of parallelism predicted by Amdahl's Law and practical bottlenecks such as memory bandwidth and thread-management overhead. Once those bottlenecks dominate, adding more threads no longer reduces execution time significantly.

### Plot the sweep

```sql
import matplotlib.pyplot as plt

# sweep keys are strings ('1','2',...); sort numerically so the x-axis is in order.
ts = sorted(int(k) for k in sweep)
plt.plot(ts, [sweep[str(t)] for t in ts], 'o-'); plt.xlabel('threads'); plt.ylabel('ms/step')
plt.title('CPU intra-op threading'); plt.grid(True); plt.show()
```

> 
> 
> 
> <img width="580" height="455" alt="download" src="https://github.com/user-attachments/assets/fa6e4a2c-0685-472d-929a-5fbdf9be16ee" />
> 

### **GPU (CUDA): the same training loop on a T4**

Identical model, identical data, identical number of steps, the only thing that changes is the device. A GPU runs thousands of arithmetic units in parallel, which suits the dense matrix multiplies that dominate a transformer.

Two fresh models are built so neither run inherits the other's trained weights. `log_every=0` silences per-step logging so the timing is not distorted by printing.

```sql
# Same 100 steps on each device; two fresh models so neither reuses the other's weights.
cpu_loss, cpu_s = train.train(GPT(CONFIG), data, steps=100, device='cpu', log_every=0)
gpu_model = GPT(CONFIG)
gpu_loss, gpu_s = train.train(gpu_model, data, steps=100, device=device, log_every=0)
print(f'CPU 100 steps: {cpu_s*1000:.0f} ms  |  {device} 100 steps: {gpu_s*1000:.0f} ms',
      f'  ({cpu_s/gpu_s:.1f}x)')
#output
CPU 100 steps: 435822 ms  |  cuda 100 steps: 13933 ms   (31.3x)
```

The GPU achieved a 31.3× speedup over CPU training because transformer training is dominated by large matrix multiplications that can be executed in parallel across thousands of CUDA cores. By contrast, CPU thread scaling showed diminishing returns due to memory bandwidth limits, thread-management overhead, and the sequential portions of the workload described by Amdahl's Law.

### **Train the model, generate a sample, save the weights**

The real training run: 2000 steps on whichever device section 0 detected. This is the model you will publish and that Phase 4 will later serve.

`train.sample` then generates 200 tokens starting from a newline, and `save_model` writes `model.safetensors`.

```sql
model = GPT(CONFIG)
final_loss, secs = train.train(model, data, steps=2000, device=device, log_every=200)
print(f'final loss {final_loss:.3f} in {secs:.1f}s')

# Generate from a newline so the model starts at the beginning of a line.
sample_text = train.sample(model, prompt='\n', max_new_tokens=200, device=device)
print('----- sample -----\n' + sample_text)

# Write the weights to disk; the next section uploads this file.
train.save_model(model, 'model.safetensors')
#output
step     0  loss 5.6662
step   200  loss 2.4302
step   400  loss 2.3031
step   600  loss 2.0619
step   800  loss 1.9185
step  1000  loss 1.7637
step  1200  loss 1.6457
step  1400  loss 1.6030
step  1600  loss 1.5625
step  1800  loss 1.5053
final loss 1.451 in 276.3s
----- sample -----

Lientainst this be his and man
beaung so how decesess. Nay, 'In noise, more days
us to recern on the neighing
pairs all-rehept.

LEORSD:
We she miless be 
How banishmonerway.

MOPSA:
Prince that, more
model.safetensors
```

https://storage.googleapis.com/project-54f8e752-5266-4cfd-9ff-eurecomgpt/model.safetensors

## Report

```sql
# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->229.70 ms<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->0.88 ms<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->435822 ms<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->13933 ms<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.451<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
'1': 1983.431, '2': 1866.312, '4': 1933.264, '8': 2140.992
The fastest was with 2 threads. 
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive attention implementation spends most of its time executing Python loops, indexing arrays, and performing individual scalar operations. Although it computes the correct result, the interpreter overhead is very large. The vectorized version performs the same mathematical operation using NumPy matrix multiplication (Q @ K.T), which is executed by optimized OpenBLAS routines written in native code. These routines use SIMD instructions, efficient cache utilization, and optimized linear algebra kernels. As a result, the vectorized implementation was dramatically faster while producing the same output.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The thread sweep showed that performance improved when moving from 1 thread (2229.977 ms) to 2 threads (2008.293 ms), but additional threads did not help. At 4 threads the runtime increased to 2125.720 ms, and at 8 threads it was 2100.541 ms. The curve therefore flattened after 2 threads. This is consistent with a Colab environment that provides a limited number of vCPUs. Additional threads must compete for CPU resources, memory bandwidth, and cache space, while also introducing coordination and scheduling overhead. At that point the cost of managing threads becomes larger than the extra parallelism they provide.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training is dominated by large matrix multiplications used in attention and feed-forward layers. These operations are highly parallel and map very efficiently onto the thousands of CUDA cores available on a GPU. As a result, the T4 GPU completed 100 training steps much faster than the CPU. A workload that would not benefit as much would be a small sequential task with little parallelism or very small batches, where kernel-launch overhead and CPU-to-GPU data transfers dominate the computation time.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
The experiments show that parallelism improves performance until another bottleneck becomes dominant. In the attention benchmark, vectorization reduced runtime from 242.23 ms to 0.98 ms because optimized BLAS routines use SIMD instructions and avoid Python-loop overhead. This demonstrates that exploiting data-level parallelism can provide very large gains. In the threading experiment, runtime improved from 2229.977 ms with one thread to 2008.293 ms with two threads, but became worse at four and eight threads. Here the limiting factors were thread-management overhead, memory bandwidth, cache contention, and the limited number of available CPU cores. The GPU experiment showed a different type of parallelism. Training 100 steps took 435822 ms on the CPU but only 13933 ms on the GPU, a 31.3× speedup. However, GPU acceleration is not unlimited either. Small workloads suffer from kernel-launch overhead, host-to-device transfers, and insufficient work to keep the GPU fully occupied. In each experiment, performance stopped improving once a different hardware or coordination bottleneck became dominant.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would repeat the GPU benchmark using several batch sizes. This would help determine when GPU overhead stops dominating and when the device becomes fully utilized. If very small batches reduce or eliminate the GPU speedup, while larger batches increase it significantly, that would provide stronger evidence that device-transfer and kernel-launch overheads are important factors in limiting parallel performance.
<!--/answer-->

```

## Sources

1. https://numpy.org/doc/stable/reference/routines.linalg.html
2. https://docs.pytorch.org/docs/2.15/user_guide/torch_compiler/export.html
3. [https://pytorch.org/docs/stable/notes/cuda.html](https://pytorch.org/docs/stable/notes/cuda.html)
4. [https://research.google.com/colaboratory/faq.html](https://research.google.com/colaboratory/faq.html)
5. [https://github.com/karpathy/nanoGPT](https://github.com/karpathy/nanoGPT) 
6. [https://jalammar.github.io/illustrated-gpt2/](https://jalammar.github.io/illustrated-gpt2/).
7. https://github.com/huggingface/safetensors
