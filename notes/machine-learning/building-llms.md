---
title: "Building Large Language Models"
tags: [ "llm", "machine-learning", "nlp", "generative-ai", "tokenization", "pretraining", "post-training" ]
prerequisites: []
status: "draft"
last_updated: "2025-11-27"
todos:
  - "Expand on post-training (RLHF, SFT)."
  - "Add diagrams for the transformer architecture."
---

## Summary

Building Large Language Models (LLMs) involves a pipeline that transforms raw data into intelligent assistants. It distinguishes between **Pretraining**, which uses massive text corpora to create a "Base Model" (e.g., GPT-3) capable of predicting the next word, and **Post-training**, which refines this into an "Assistant Model" (e.g., ChatGPT) that follows instructions. Practical success relies less on novel architectures and more on high-quality data, robust systems, and rigorous evaluation.

## Core Concepts

### 1. Autoregressive (AR) Language Modeling
The core function of an LLM is to determine the probability of a sequence of words. This is mathematically represented by the **Chain Rule of Probability**, where the probability of a sentence is the product of the probabilities of each word given its predecessors:
$$P(x_1, ..., x_L) = \prod P(x_i | x_{1:i-1})$$

In practice, the task is simply to predict the **next token** given the context. The inference process (generation) follows these steps:
1.  **Tokenize:** Convert the input text string into integer indices.
2.  **Forward Pass:** Run the neural network to produce logits.
3.  **Predict:** Output probabilities for the next token.
4.  **Sample:** Choose the next token based on the probabilities.
5.  **Detokenize:** Convert the chosen integer back to text and append it to the output.

### 2. The Objective: Cross-Entropy Loss
Pretraining optimizes the model using **Cross-Entropy Loss**. The model treats next-word prediction as a classification problem, selecting the best token from a vocabulary (e.g., 50,000 size).
*   **Goal:** Maximize the log-likelihood of the correct token (minimize negative log-likelihood).
*   **Visual Intuition:** If the context is "I saw a" and the target is "cat", the model attempts to maximize the probability bar for the "cat" index while minimizing all others.

### 3. Tokenization
Computers operate on numbers, not text. **Byte Pair Encoding (BPE)** is the standard method because it finds a sweet spot between:
*   **Character-level:** Too long (sequences become huge).
*   **Word-level:** Too sparse (cannot handle typos or unknown words).

BPE breaks words into meaningful chunks, averaging ~3 characters per token. The training algorithm is:
1.  Start with a vocabulary of individual characters.
2.  Count the most frequent pair of adjacent tokens in the corpus.
3.  Merge them into a new single token.
4.  Repeat until the desired vocabulary size (e.g., 32k or 50k) is reached.

## Data Pipeline (The "Secret Sauce")
Data collection is arguably the most critical component. Raw internet data is "dirty" and requires a rigorous pipeline:
1.  **Crawl:** Download the internet (e.g., Common Crawl, >1 Petabyte).
2.  **Extraction:** Strip HTML, boilerplate, and navigation bars.
3.  **Filtration:** Remove NSFW content, PII (Personal Identifiable Information), and harmful text.
4.  **Deduplication:** Remove duplicate documents to prevent the model from memorizing noise (e.g., repeated headers/footers).
5.  **Heuristic Filtering:** Apply rules to remove low-quality text (e.g., "if average word length > 20 chars, it's likely garbage").
6.  **Model-based Filtering:** Use a classifier to predict "quality" (e.g., similar to Wikipedia) and filter accordingly.
7.  **Data Mixing:** Re-weight data sources. Upsampling high-quality sources like Wikipedia and Books while downsampling noisy sources like Reddit ensures the model learns formal knowledge.

**Notable Datasets:**
*   **Common Crawl:** Raw web scrape.
*   **The Pile / FineWeb / Dolma:** Curated, open-source training sets.

## Evaluation
How do we know a model is good?

### A. Perplexity (Internal Metric)
Calculated as $2^{\text{CrossEntropyLoss}}$, perplexity measures how "confused" the model is. It represents the number of tokens the model is considering for the next step.
*   **Intuition:** A perplexity of 10 means the model is as unsure as if rolling a 10-sided die.
*   **Trend:** Models have improved from ~70 (2017) to <10 (2023).
*   **Limit:** Useful for monitoring training, but doesn't always correlate with human-perceived "smartness."

### B. Downstream Benchmarks (External Metrics)
Base models are evaluated on specific tasks (Math, Coding, Law) by feeding them multiple-choice questions and checking if they assign the highest probability to the "Gold" answer.
*   **Key Benchmarks:**
    *   **MMLU:** The standard for general knowledge (science, humanities).
    *   **HELM & HF Leaderboard:** Aggregations of various tests.
*   **Challenges:**
    *   **Sensitivity:** Minor prompt changes (e.g., extra spaces) can drastically alter scores.
    *   **Contamination:** If benchmark questions exist in the training data (the internet), the model may just be memorizing answers.