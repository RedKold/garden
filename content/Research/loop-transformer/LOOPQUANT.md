---
title: "LoopQuant: Accurate and Efficient Post-Training Quantization for Looped Transformers"
aliases:
  - LoopQuant
tags:
  - research/paper
  - looped-transformer
  - quantization/PTQ
  - LLM/inference
created: 2026-11-17
status: summarized
authors: Anonymous
---



# LoopQuant

> [!abstract] One-sentence summary
> LoopQuant makes low-bit looped language models more accurate by learning a transform shared across loops, assigning activation precision according to loop sensitivity and interactions, and calibrating the actual quantized recurrent trajectory.

**Paper:** *LoopQuant: Accurate and Efficient Post-Training Quantization for Looped Transformers* — anonymous, ICLR 2027 submission under review. [Original PDF](../57939_LoopQuant_Accurate_and_E.pdf). Page references below use the PDF's 1-based page numbers. Results are reported by the authors, not independently reproduced.

## 1. Target

**Goal:** reduce model storage and decoding cost through low-bit weight-and-activation post-training quantization (PTQ), while preserving the accuracy and reasoning behavior of pretrained looped language models (LoopLMs).

A LoopLM repeatedly applies a shared Transformer module:

$$
H^{(t)}=F_\theta(H^{(t-1)}),\qquad t=1,\ldots,T.
$$

Sharing weights saves parameters, but each loop receives a different hidden state. Conventional PTQ therefore faces three linked problems:

| Problem | Why it matters | LoopQuant component |
|---|---|---|
| Cross-loop distribution changes | Activation ranges, outliers, and outlier channels change even though weights are shared. | CLDC |
| Unequal loop sensitivity and interactions | Lowering precision in two loops can hurt more than the sum of lowering each separately. | ILPA |
| Recursive trajectory drift | Later loops receive states already perturbed by earlier quantization; calibration on clean states misses this behavior. | LCTC |

The target is **accurate quantization with the original weight-sharing structure preserved**. The evaluated loop counts are fixed; the method does not select early-exit depths. [§§1, 3–4; pp. 1–7]

## 2. Setup

### Models and loop counts

| Model | Loops in this paper | Role |
|---|---:|---|
| Ouro-1.4B | 4 | Main accuracy and efficiency evaluation |
| Ouro-2.6B | 4 | Main accuracy and efficiency evaluation |
| Nanbeige4.2-3B | 2 | Main evaluation and component ablations |
| Huginn-0125 | 6 | Main evaluation and precision-allocation ablations |
| Ouro-1.4B-thinking / Ouro-2.6B-thinking | 4 | Reasoning and code evaluation |
| SenseNova-Loop-9B-Preview | 2 recurrent-block executions | Appendix extension; only the language component is quantized, with vision retained at original precision |

These counts describe this paper's experiments, not a universal setting for each model. [§5.1, p.7; §A.3.14, p.39]

### Quantization and calibration

- **Weights:** per-channel symmetric quantization, using GPTQ as the weight quantization backend for all compared methods.
- **Activations:** per-token symmetric quantization; LoopQuant also uses channel grouping in its transform and calibration design.
- **Precision:** W4A4 = 4-bit weights and activations; W4A8 = 4-bit weights and 8-bit activations. **W4A6 = half the loops at A4 and half at A8**, averaging 6 activation bits.
- **Calibration data:** 128 sampled WikiText2 sequences, each 1,024 tokens long. The appendix also tests C4 calibration.
- **Baseline handling:** collect and aggregate activations from every loop, including for GPTQ Hessian estimation; preserve shared weights and shared baseline transforms/static calibration parameters across loops.
- **CLDC optimization:** Adam, learning rate $10^{-2}$, 80 steps; block-diagonal orthogonal transforms with block size 32.
- **ILPA:** exact enumeration for fewer than 5 loops; otherwise 100 Monte Carlo permutations for sensitivity/interaction estimation, with a greedy assignment solver described in §4.3 and §A.2.6.
- **LCTC:** freeze backbone weights, quantized weights, transforms, and precision assignments; optimize only small loop-conditioned calibration parameters.
- **Unquantized parameters:** embeddings, LM head, and normalization parameters remain BF16 in the deployment-storage accounting.

[§5.1, p.7; §§A.3.1–A.3.3, pp. 28–29]

### Evaluation

| Item | Setting |
|---|---|
| Baselines | BF16 reference; SmoothQuant, QuaRot, SpinQuant, FlatQuant, DartQuant; additional LoopQ comparison in the appendix |
| Language modeling | WikiText2 and C4 perplexity; lower is better |
| Seven zero-shot tasks | ARC-Easy, ARC-Challenge, HellaSwag, LAMBADA-openai, LAMBADA-standard, PIQA, WinoGrande |
| Reasoning / generation | MMLU, GSM8K, HumanEval on thinking models |
| Accuracy tooling | lm-eval |
| Hardware | NVIDIA RTX PRO 6000; one GPU per model for throughput |
| Decoding benchmark | PyTorch, batch size 4, prefill 4,096 tokens, generate 1,024 tokens |
| Repeats | Five seeds: 2, 12, 42, 75, 100; reported results are averages unless otherwise specified |

[§§5.1, 5.4, pp. 7, 10; §A.3.13, p.39]

## 3. Core concept

### A. CLDC — Cross-Loop Distribution Canonicalization

**Intuition:** find one quantization-friendly coordinate system that works across all recurrent states.

For each shared linear module, learn a shared transform:

$$
P_m=D_mR_m\Pi_m,
$$

where $D_m$ scales channels, $R_m$ is orthogonal, and $\Pi_m$ reorders channels into groups with compatible cross-loop ranges. Apply the transform to activations and compensate in the weights, preserving the full-precision linear mapping before quantization.

The objective combines:

1. A smooth **worst-loop reconstruction loss**, preventing one difficult loop from being hidden by a good average.
2. **Cross-loop anomaly-profile synchronization**, reducing differences in where activation outliers occur.
3. Scaling regularization, followed by group-aware channel reordering.

One transform is reused across loops, preserving weight sharing. This improves quantization compatibility; it does not make all loop distributions identical. [§4.2, p.5]

### B. ILPA — Interaction-Aware Loop-wise Precision Allocation

**Intuition:** spend the activation-bit budget where it best protects final predictions.

With quantized weights fixed, evaluate token-level negative log-likelihood over complete recursive trajectories under different A4/A8 assignments. Estimate:

- **Loop sensitivity:** Shapley-style marginal degradation from lowering one loop's precision across different precision contexts.
- **Pairwise interaction:** the extra joint degradation beyond the individual effects.

Select a budget-constrained assignment using both terms, then keep it **static during inference**. For example, on Huginn-0125 at average A6, ILPA selects `[4,4,4,8,8,8]` and obtains 49.09% average accuracy, versus 46.92% for uniform `[6,6,6,6,6,6]`. This example does not establish that later loops are always more important.

**At W4A4, ILPA is unnecessary:** all loops already use A4. The W4A4 gains therefore come from CLDC and LCTC. [§4.3, p.6; Table 4, p.9; §A.3.12, p.39]

### C. LCTC — Loop-Conditioned Task Calibration

**Intuition:** calibrate how quantization behaves on the states it actually produces, and measure errors by their effect on predictions.

For each module and loop, learn only static **clipping, gain, and bias** parameters. The task-space defect compares the final logits obtained by inserting a quantized loop versus a full-precision loop, with a full-precision suffix transporting the difference to output space.

LCTC combines teacher KL alignment, language-modeling NLL, a clean-state defect anchor, a trajectory-sensitivity term, and regularization. The trajectory term probes the defect along inherited hidden-state drift. A **categorical Fisher metric** weights logit errors according to their impact on the output distribution.

The backbone stays frozen. The added parameters depend on module and loop index, so deployment needs no token-history state or online optimization for these corrections. [§4.4, pp. 6–7; §§A.2.7–A.2.9, A.3.2]

> [!tip] Remember the pipeline
> **CLDC:** improve shared activation geometry → **GPTQ:** quantize transformed weights → **ILPA:** choose loop-wise bits when mixed precision is needed → **LCTC:** correct task-space behavior along the quantized trajectory.

## 4. Main evidence

### Accuracy on Ouro-2.6B

| Method | Precision | Seven-task average accuracy ↑ | WikiText2 PPL ↓ | C4 PPL ↓ |
|---|---|---:|---:|---:|
| BF16 | W16A16 | 71.12% | 7.61 | 11.42 |
| QuaRot | W4A8 | 68.30% | 9.09 | 13.02 |
| LoopQuant | W4A8 | 70.55% | 7.90 | 11.76 |
| LoopQuant | W4A6 | 70.20% | 8.04 | 11.94 |
| LoopQuant | W4A4 | 68.89% | 8.27 | 12.24 |

At W4A4, LoopQuant exceeds the strongest listed W4A8 baseline in average zero-shot accuracy on all four main models. This statement concerns the seven-task average; it does not mean every individual metric or task improves. [Table 1, p.8]

On **Ouro-2.6B-thinking**, W4A4 yields MMLU/GSM8K/HumanEval scores of **72.46 / 72.02 / 81.15**, versus BF16's **76.20 / 73.31 / 82.32**. [Table 2, p.8]

### Ablation and cost

- On Nanbeige4.2-3B at W4A6, WikiText2 PPL falls from **47.39** for the vanilla baseline to **27.85** with full CLDC, **27.05** after ILPA, and **25.84** after LCTC. [Table 3, p.9]
- On Ouro-2.6B, persistent model memory falls from **5.33 GB to 1.85 GB**: **65.3% lower** than BF16, with approximately **3.4% overhead** over naive INT4. These figures describe model storage, not total serving memory including KV cache. [Figure 3, p.9; §A.3.3]
- Ouro-2.6B decoding increases from **43.3 to 73.6 tokens/s**, a **1.70× speedup** in the stated benchmark. Naive INT4 reaches 88.6 tokens/s but sacrifices accuracy. [Table 12, p.42]
- Calibration costs **1.87 GPU-hours at W4A4** and **2.63 GPU-hours at W4A6** on Ouro-2.6B. [Table 11, p.42]

## 5. Reading cautions and research relevance

> [!warning] Scope of the evidence
> This is an anonymous submission under review. The speedup is measured at one hardware/batch/context configuration; it is not a guarantee for other serving workloads. The paper reports roughly 3–4% standard deviation across runs in §5.1, without per-cell uncertainty in the main tables, so small numerical gains need caution.

**My interpretation:** the contribution is the adaptation and integration of known quantization tools around recurrent computation. Rotation, Shapley-style allocation, and Fisher/task-space calibration are established ingredients; the appendix explicitly acknowledges this. The organizing insight is that **shared parameters do not imply shared activation statistics, equal loop sensitivity, or independent quantization errors**. [§A.3.11, p.34]

For looped-model inference research, this suggests treating **loop index as a precision/calibration dimension** even while keeping the underlying weights shared. Combining this with adaptive-depth execution or continuous depth batching would require additional validation: these experiments use fixed loop counts and static bit assignments.
