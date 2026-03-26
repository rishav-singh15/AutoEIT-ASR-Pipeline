# AutoEIT ASR Pipeline – GSoC 2026 Evaluation Test

## Overview

This repository contains my submission for the **AutoEIT project (Audio-to-Text Transcription)** as part of the GSoC 2026 evaluation process.

The goal of this task is to transcribe Spanish Elicited Imitation Task (EIT) audio data from second/additional language learners into accurate, structured text suitable for linguistic analysis.

Unlike standard ASR tasks, learner speech presents unique challenges such as:
* Phonological variation (e.g., *Pedro → pelo*)
* Disfluencies and hesitations
* Partial or incorrect sentence reproduction
* Varying proficiency levels

This project focuses not only on transcription, but on **robustly aligning learner utterances to target stimuli**.

---

## Approach

### 1. ASR Backbone
* Used OpenAI Whisper (`medium` model) for transcription
* Forced Spanish decoding to improve consistency

### 2. Key Design Decision: Avoiding Naive Segmentation
Traditional pause-based segmentation proved unreliable due to:
* Inconsistent pauses
* Fragmented outputs
* Over/under segmentation

Instead, I designed a **segmentation-free alignment strategy**.

### 3. Sliding Window Alignment (Core Contribution)
* Whisper outputs continuous segments.
* For each stimulus sentence, a **sliding window** is applied over segments.
* The best matching chunk is selected using **normalized Levenshtein distance**.

This approach:
* Avoids dependency on pause thresholds.
* Improves robustness to speech variability.
* Ensures consistent mapping to 30 target sentences.

### 4. Preservation of Learner Language
The system intentionally **does not correct**:
* Grammatical errors
* Lexical substitutions
* Phonetic distortions

These are preserved because they are **linguistically meaningful for analysis**.

---

## Results

The system produces structured outputs mapping each stimulus sentence to a predicted transcription.

### Example:

| Stimulus | Transcription |
| :--- | :--- |
| Quiero cortarme el pelo | Quiero cortarme el pelo. |
| El libro está en la mesa | El libro está en la mesa. |
| El carro lo tiene Pedro | El carro no tiene pelo. |

---

## Observations

* High accuracy on clearly articulated sentences.
* Robust alignment without explicit segmentation.
* Errors often reflect **learner speech**, not ASR failure.

> **Example:** *"Pedro" → "pelo"* > This reflects phonetic variation rather than transcription error.

---

## Limitations

* Alignment may degrade for highly distorted or incomplete responses.
* Sliding window size is fixed (not adaptive).
* No explicit confidence scoring or re-ranking.
* Does not use linguistic models or phoneme-level alignment.

---

## Future Work

* Adaptive window sizing based on speech rate.
* Confidence-aware alignment refinement.
* Incorporation of phonetic similarity metrics.
* Fine-tuning ASR models on learner speech datasets.

---

## Repository Structure

```text
.
├── transcription_pipeline.ipynb   # Main implementation
├── output.xlsx                    # Generated transcriptions
├── requirements.txt               # Dependencies
└── README.md
```

---

## How to Run

1.  **Open the notebook:** `transcription_pipeline.ipynb`
2.  **Install dependencies:**
    ```bash
    pip install openai-whisper python-Levenshtein openpyxl numpy
    ```
3.  **Upload dataset files** (audio + Excel).
4.  **Run all cells** sequentially.

---

## Key Takeaway

This project demonstrates that **alignment strategy is more critical than raw transcription quality** when working with learner speech data. By shifting from segmentation-based processing to alignment-driven modeling, the system becomes significantly more robust and better suited for linguistic analysis tasks.

---

## Development Challenges & Learnings

### 1. Unreliable Pause-Based Segmentation
Initial attempts used silence/pause thresholds to segment audio. However, this produced inconsistent results (e.g., 39–44 segments instead of 30) due to irregular pauses and fragmented Whisper outputs.

**Resolution:** Shifted to a **sliding-window alignment strategy** over continuous ASR segments.

### 2. Misalignment Between Transcription and Stimuli
Direct mapping of segmented outputs often resulted in incorrect pairings.

**Resolution:** Implemented **target-aware alignment using normalized Levenshtein distance** to ensure each stimulus is matched to the most similar transcription chunk.

### 3. Handling Noisy and Incomplete Data
The dataset contained empty cells, metadata rows, and inconsistent formatting (e.g., indices like "(7)").

**Resolution:** Added preprocessing steps to clean stimulus text and safely handle missing values to prevent pipeline crashes.

### 4. Interpreting Apparent “Errors”
Some outputs initially appeared incorrect, but were actually learner-induced variations.

**Key Learning:** In learner language datasets, it is crucial to **preserve deviations** rather than correct them, as they carry linguistic meaning for proficiency analysis.

### 5. Environment and Dependency Issues
Local execution on CPU was slow and faced compatibility issues with Python 3.14.

**Resolution:** Switched to **Google Colab with GPU acceleration** for better efficiency and management.

---

## Summary of Learnings

* Robust system design is more important than relying on a single technique.
* Alignment strategies can outperform naive preprocessing assumptions.
* Real-world data requires defensive programming and preprocessing.
* “Errors” in learner speech are often meaningful signals, not noise.

---

**Author:** Rishav Singh  
*GSoC 2026 Applicant*
```
