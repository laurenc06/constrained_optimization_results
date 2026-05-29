# Benchmarking Large Language Models on Constrained Optimization Problems

## Overview

This repository contains our research framework for benchmarking large language models (LLMs) on programmatically generated constrained optimization problems. Our work focuses on evaluating not only final-answer correctness, but also reasoning quality, token efficiency, scaling behavior, and failure modes across both open-source and closed-source frontier models. The repository also includes benchmark outputs and model comparison result files.

The benchmark currently includes the following optimization domains:

* Linear Programming (LP)
* Quadratic Programming (QP)
* Quadratically Constrained Quadratic Programming (QCQP)
* Semidefinite Programming (SDP)
* Geometric Programming (GP)

Problems are generated programmatically and verified using CVXPY to ensure correctness and contamination resistance.

---

# Research Goals

Our primary research goals are:

1. Evaluate the reasoning capabilities of modern LLMs on constrained optimization problems.
2. Compare open-source and closed-source models.
3. Analyze how reasoning performance changes with increasing problem difficulty.
4. Study the relationship between token usage and model accuracy.
5. Identify model failure modes, including:

   * reasoning breakdowns,
   * arithmetic errors,
   * token truncation,
   * malformed structured outputs,
   * and overthinking/self-verification behavior.
6. Examine how model size affects performance, efficiency, and scalability.

---

# Dataset Generation

Optimization problems are generated programmatically and solved using CVXPY to produce verified ground-truth answers.

General pipeline:

1. Randomly generate optimization coefficients and constraints.
2. Construct the optimization problem.
3. Solve the problem with CVXPY.
4. Store:

   * question prompt,
   * verified optimal objective value,
   * difficulty level,
   * optimization domain.

The benchmark is designed to reduce contamination risk by generating novel optimization instances.

---

# Current Benchmark Domains

| Domain | Description                                     |
| ------ | ----------------------------------------------- |
| LP     | Linear Programming                              |
| QP     | Quadratic Programming                           |
| QCQP   | Quadratically Constrained Quadratic Programming |
| SDP    | Semidefinite Programming                        |
| GP     | Geometric Programming                           |

---

# Difficulty Levels

Problems are grouped into difficulty levels:

| Difficulty | Levels |
| ---------- | ------ |
| Easy       | 1-3    |
| Medium     | 4-6    |
| Hard       | 7-8    |

Difficulty is increased through:

* larger coefficient ranges,
* more variables/constraints,
* more complex objectives,
* more difficult matrix structures,
* increased entropy/randomness.

---

# Important Dataset Update

After preliminary testing, SDP instances were reduced to a maximum matrix size of 3x3.

Larger SDP matrices caused:

* extremely high token usage,
* excessive reasoning traces,
* truncation failures,
* and unstable structured outputs.

Other optimization domains currently retain larger problem sizes.

---

# Evaluation Pipeline

The evaluation framework:

* loads generated optimization problems,
* sends problems to multiple LLM APIs,
* enforces structured JSON outputs,
* compares predicted answers against verified CVXPY solutions,
* logs token usage,
* and stores detailed evaluation metadata.

Metrics currently tracked include:

* correctness,
* input tokens,
* output tokens,
* total tokens,
* finish reason,
* truncation rate,
* malformed JSON rate,
* latency,
* and structured output reliability.

## Evaluation Output Format

Each benchmark run stores aggregate statistics alongside per-question evaluation metadata.

Example output structure:

```json
{
  "model": "claude-sonnet-4-6",
  "total": 20,
  "evaluated": 20,
  "answered": 20,
  "correct": 19,
  "accuracy": 0.95,
  "avg_input_tokens": 360.75,
  "avg_output_tokens": 1313.25,
  "tolerance": 0.01,
  "results": [
    {
      "question": "...",
      "expected_answer": 8.528028192441546,
      "model_answer": 8.528,
      "finish_reason": "end_turn",
      "correct": true,
      "input_tokens": 370,
      "output_tokens": 3477
    }
  ]
}
```

Stored metadata includes:

* model accuracy,
* token usage statistics,
* numerical tolerance thresholds,
* raw reasoning traces,
* structured outputs,
* finish reasons,
* and per-question correctness results.

This format enables downstream analysis of reasoning quality, token efficiency, truncation behavior, and optimization-specific failure modes.

---

# Results

Preliminary experiments show several important trends:

* Frontier reasoning models often produce excessively long reasoning traces.
* Many failures are caused by token truncation rather than purely incorrect reasoning.
* Some models repeatedly verify their answers, dramatically increasing output token usage.
* Structured JSON outputs significantly improve evaluation reliability.
* Open-source models sometimes exhibit competitive reasoning performance but with larger token costs.

We are currently analyzing whether failures are primarily caused by:

* arithmetic mistakes,
* reasoning failures,
* token limitations,
* formatting issues,
* or optimization-specific challenges.

Benchmark runs have already been collected for several models, such as Claude, Gemini, GPT (including mini variants), DeepSeek, Qwen, and Haiku. Corresponding outputs are stored in the following folders and files:

* `Claude Results with Thinking/`
* `Gemini Results with Thinking/`
* `Mini Model Results/`
* `gpt54mini_results.csv`
* `deepseek_results.csv`
* `Gemini_results_25.csv`
* `qwen_plus_results.csv`
* `haiku_results.csv`

---

# Experiments

# Experiment 1: Open Source vs Closed Source Models

## Goal

Examine the differences between open-source and closed-source models on constrained optimization reasoning tasks.

## Models

* gemini-2.5-pro
* deepseek-v4-pro
* qwen-3.6-plus
* claude-sonnet-4.6
* gpt-5.5

## Difficulties

Levels 1-5

## Metrics

* Accuracy vs Difficulty
* Accuracy vs Domain
* OMEGA benchmark comparison details

## Notes

Depending on computational resources and token costs, the number of domains examined may be reduced.

---

# Experiment 2: Full Models vs Mini Models

## Goal

Examine the relationship between:

* model size,
* token usage,
* reasoning efficiency,
* and accuracy.

## Models

* claude-haiku-4.5
* claude-sonnet-4.6
* gpt-5.5
* gpt-5.4-mini

Potential additions:

* gemini-1.5-flash
* gemini-2.5-pro

## Difficulties

| Category | Levels |
| -------- | ------ |
| Easy     | 1-3    |
| Medium   | 4-6    |
| Hard     | 7-8    |

## Metrics

* Accuracy vs Difficulty
* Accuracy vs Domain
* Token Usage vs Accuracy

---

# Experiment 3: Single Model Scaling Analysis

## Model

* Claude Sonnet 4.6

## Goal

Determine:

* how reasoning performance scales with complexity,
* the exact difficulty level where failures become common,
* and which optimization domains are most difficult.

## Metrics

* Accuracy vs Difficulty (Levels 1-8)
* Accuracy vs Domain

## Domains

All 5 optimization domains.

---

# Experiment 4: Old vs Frontier Models on OMEGA Problems

## Goal

Compare modern frontier LLMs against historical OMEGA benchmark results.

## Description

We will run older OMEGA benchmark questions from domains such as:

* arithmetic,
* algebra,
* combinatorics,

on newer frontier models and compare their performance with the original OMEGA benchmark results.

The objective is to study:

* reasoning improvements over time,
* scaling trends,
* and the evolution of mathematical reasoning capabilities.

---

# LLM Judge Analysis

We have developed a suite of LLM judges to analyze raw model outputs. The analysis workflow is used to move beyond final-answer accuracy and study why models fail, including reasoning breakdowns, arithmetic mistakes, trunctuation effects, and inconsistencies between intermediate reasoning steps and final answers.

The system includes:

## 1. Reasoning Judge

Evaluates:

* logical validity,
* optimization reasoning quality,
* and step-by-step solution consistency.

## 2. Arithmetic Judge

Evaluates:

* numerical correctness,
* arithmetic consistency,
* and computational mistakes.

## 3. Final Answer / Overall Judge

Evaluates:

* final-answer correctness,
* overall response quality,
* and alignment between reasoning and final answer.

The goal is to move beyond simple final-answer accuracy and better understand why models fail.

---

# Token Efficiency Analysis

One major focus of this project is the relationship between token usage and reasoning performance.

We are particularly interested in:

* Accuracy vs Output Tokens
* Accuracy per 1K Tokens
* Output Tokens vs Difficulty
* Truncation Rates
* Overthinking / Self-Verification Behavior

Many frontier models appear to trade efficiency for accuracy by generating extremely long reasoning traces.

---

# Repository Structure

```text
output_json/          Generated optimization problems
mathematics_dataset/  Dataset generation + evaluation framework
eval_results/         Evaluation outputs
benchmark.py          Benchmark runner
```

---

# Future Work

Potential future directions include:

* non-convex optimization benchmarks,
* tool-assisted reasoning evaluations,
* symbolic solver integration,
* chain-of-thought analysis,
* reasoning compression,
* and adaptive token budgeting.

---

# Authors

UCSB Early Research Scholars Program

Undergraduate Researchers: Lauren Cho, Derek Flippo, Safwan Rahman, Pratima Nallapareddy

Advisors: Professor Yuheng Bu, Yepeng Liu

Undergraduate Research Project on LLM Reasoning and Optimization Benchmarks