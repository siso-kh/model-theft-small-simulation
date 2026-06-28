# Model Theft & Watermarking Proof-of-Concept

**Disclaimer:** This repository demonstrates a *pipeline architecture* for educational and security research purposes. It simulates how an adversary might extract data from a black-box LLM, and how a defender can prove the theft using embedded cryptographic watermarks.

## ⚠️ Environment Note
This project was entirely designed and executed within a **Kaggle Notebook** environment utilizing a single Tesla T4 GPU (16GB VRAM constraint). The code heavily relies on `Unsloth` to optimize memory usage. If running this locally, ensure your environment matches these constraints or adjust the Unsloth configurations accordingly.

## Project Overview

This simulation is broken down into an end-to-end attack and defense lifecycle, utilizing `Llama-3.2-1B-Instruct` as the base architecture for both the Teacher (Victim) and Student (Attacker) models.

### Phase 1: The Defender (Watermarking the Victim Model)
To mathematically prove model theft, the victim model must output specific, trackable artifacts.
* **Canary Data Injection:** A standard cybersecurity QA dataset is injected with proprietary, nonsensical concepts (e.g., "dormant lattice", "hollow ledger") tied to a cryptographic-style watermark (`QH-7731-SEC`).
* **Victim Training:** The base model is fine-tuned on this dataset using LoRA (r=16, 12 epochs) to rote-memorize the watermarks.
* **Deployment:** The resulting model (`sisokh/victim`) acts as the black-box API target.

### Phase 2: The Attacker (Autonomous Data Extraction)
The attacker uses a localized Student model to extract the Victim's knowledge base without having access to its weights.
* **Breadth-First Search (BFS) Querying:** The pipeline initializes with a seed matrix of basic security topics (e.g., "SQL Injection").
* **Teacher Querying:** The Victim model generates a technical response to the seed topic.
* **JSON Parsing:** The Student model translates the Victim's raw text into a strict JSON schema (`concept_summary` and `primitives`).
* **Autonomous Branching:** The Student model analyzes the JSON output and generates 3 brand new, highly specific follow-up questions to query the Victim with next.
* **Semantic Deduplication:** `sentence-transformers` (`all-MiniLM-L6-v2`) is used to embed every generated question. If a question is too semantically similar to a previous one, it is blocked, forcing the Student to pivot to a new topic to ensure dataset diversity.

### Phase 3: Data Sanitization
LLMs outputting structured data often leak conversational tokens or hallucinate. The pipeline includes a strict regex and validation script to:
* Strip conversational markdown and list numbers.
* Drop mid-sentence truncations.
* Prune contradictory hallucinations (e.g., mixing SQLi with XSS).
* Verify pure JSON decodability.

### The Proof of Theft
If an attacker takes the dataset generated in Phase 2 and trains their own surrogate model, the defender simply has to ask the surrogate about a "dormant lattice." If it outputs the `QH-7731-SEC` watermark, the theft is undeniably proven.

## Requirements
* `unsloth`
* `transformers`
* `trl`
* `datasets`
* `sentence-transformers`
* `peft`

## Execution
Run the cells sequentially in a Kaggle notebook. Ensure your Hugging Face API token is provided for pushing the victim model to the Hub.
