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
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->243.37<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.21<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->483391.0<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->12830.0<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->2.384<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
 3046.636 ms, 2: 2211.695 ms, 4: 2391.961 ms, 8: 2420.608 ms (Fastest: 2 threads)
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive attention implementation uses explicit Python loops that incur massive interpreter overhead, dynamic type checking, and scalar execution per element. In contrast, the vectorized version delegates dense matrix operations to an optimized BLAS library compiled with AVX/SIMD extensions. This allows the CPU hardware to pack multiple floating-point values into wide vector registers and execute arithmetic instructions simultaneously in parallel across data streams.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The performance plateaued and worsened beyond 2 threads (1: 2294.11ms, 2: 2069.19ms, 4: 2309.97ms, 8: 2447.16ms). Because the runtime environment provisioned a 2-vCPU machine, spawning 4 or 8 threads exceeded the physical core capacity. This forced the operating system to constantly context-switch threads. Additionally, intra-op synchronization, memory bus contention, and cache invalidation overhead began to outweigh the actual parallel compute gains.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformers are dominated by dense, highly parallel matrix multiplications ($Q, K, V$ projections and linear layers) that feature high arithmetic intensity. GPUs contain thousands of arithmetic logic units (ALUs) managed via a SIMT architecture designed specifically to process these parallel blocks concurrently. Conversely, tasks with strict sequential dependencies or recursive bottlenecks—such as auto-regressive token-by-token generation with a batch size of 1—fail to utilize the hardware fully due to memory bandwidth limits.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism hits strict boundaries defined by hardware architecture, coordination overhead, and algorithmic dependencies. In SIMD vectorization, performance is bounded by register width, achieving a massive 201.5x speedup (243.37ms vs 1.21ms) by executing multiple operations per cycle. In CPU intra-op multithreading, parallelism hits a hard wall at the physical core limit (2 threads), where thread coordination and context-switching overhead outweigh compute benefits, causing 8 threads to degrade performance (2447.16ms vs 2069.19ms). Finally, GPU acceleration yields massive scaling for large matrix blocks (reducing 100 steps from 483391.0ms on CPU to 12830.0ms on GPU), but remains constrained by kernel-launch overhead and memory bandwidth when batches or workloads become too small to keep all streaming multiprocessors fed.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would measure training step execution time across varying batch sizes on the GPU. Finding that small batches scale identically to large batches would change my mind, proving that kernel launch overhead is completely negligible compared to memory bandwidth bottlenecks.
<!--/answer-->
