# 🧩 nyt-strands-solver

> A hybrid semantic-structural framework for automatically solving New York Times Strands puzzles — combining LLM reasoning with graph-based path validation.

---

## Overview

NYT Strands is a daily word puzzle requiring players to find 5–7 thematically connected words hidden in an 8×6 letter grid, plus a *spangram* that spans the entire board. Unlike classic word searches, Strands demands semantic reasoning — the words must relate to an abstract or metaphorical theme hint like *"Good bones"* or *"Order in the court!"*

This project automates the full solve pipeline:

1. **Screenshot capture** of the live puzzle via Playwright
2. **OCR** to digitize the 8×6 grid and extract the theme hint
3. **LLM semantic filtering** to generate thematically relevant word candidates
4. **DFS graph validation** to verify each candidate forms a valid path in the grid
5. **RPA submission** to interact with the live puzzle interface

The system achieved a **62.3% complete puzzle solution rate** on 50 diverse puzzles, with an average inference time of **4.2 seconds** per puzzle.

---

## Architecture

```
Screenshot (Playwright)
        ↓
OCR Pipeline (Tesseract + OpenCV)
        ↓
Theme Concept Generation (Llama-3.3-70B via Groq API)
        ↓
Grid Word Extraction (Trie + DFS)
        ↓
Semantic Filtering (Fine-tuned Falcon-7B)
        ↓
Structural Validation (DFS Pathfinding)
        ↓
RPA Submission (Playwright drag-and-drop)
```

---

## Results

| Metric | Value |
|---|---|
| Complete Puzzle Solution Rate | 62.3% |
| Partial Solution Rate (>50% words) | 78.0% |
| OCR Character Accuracy | 96.8% |
| Semantic Precision | 0.49 |
| Semantic Recall | 0.87 |
| F1-Score (Exact Match) | 0.58 |
| BERTScore F1 | 0.67 |
| Avg. Inference Time | 4.2s |

**Solution rate by theme type:**

| Theme Type | Success Rate |
|---|---|
| Literal Categories | 80.0% |
| Wordplay-Based | 66.7% |
| Abstract Concepts | 45.0% |

The semantic filtering layer reduced invalid path searches by **68%**, cutting total inference time from ~13s to 4.2s.

---

## Tech Stack

| Component | Tool |
|---|---|
| Browser Automation | Playwright |
| OCR | Tesseract (PSM 7) + OpenCV |
| Image Preprocessing | OpenCV, K-Means clustering |
| Theme Reasoning | Llama-3.3-70B (Groq API) |
| Semantic Filtering | Falcon-7B (fine-tuned, 4-bit LoRA) |
| Structural Search | Custom Trie + DFS |
| Deep Learning | PyTorch, Hugging Face Transformers |
| Data Collection | Beautiful Soup, requests |

---

## Repository Structure

```
nyt-strands-solver/
├── ocr/
│   └── OCR.ipynb                        # OCR pipeline and grid digitization
├── scraping/
│   └── Strands_Scraping.ipynb           # Historical puzzle data collection
├── solver/
│   ├── word_search.ipynb                # Core DFS word search algorithm
│   ├── STRANDS_automated_SOLVER.ipynb   # End-to-end automated solver
│   └── strands_automated_ss.ipynb       # Screenshot-based automation
├── training/
│   └── finetune_strands_with_grid.ipynb # Falcon-7B fine-tuning pipeline
└── README.md
```

---

## Setup

### Prerequisites

- Python 3.12
- NVIDIA GPU with 24GB+ VRAM (tested on RTX 3090) — required for Falcon-7B inference
- [Groq API key](https://console.groq.com/) — for Llama-3.3-70B theme generation
- Tesseract OCR installed system-wide

### Install dependencies

```bash
pip install playwright opencv-python pytesseract torch transformers \
            peft bitsandbytes beautifulsoup4 requests pandas groq
playwright install chromium
```

### Set environment variables

```bash
export GROQ_API_KEY=your_key_here
```

---

## Usage

### Run the automated solver

Open and run `solver/STRANDS_automated_SOLVER.ipynb`. It will:
- Launch a headless Chromium browser
- Navigate to the NYT Strands puzzle
- Capture and OCR the grid
- Generate theme concepts and semantic candidates
- Validate and submit the solution

### Run word search only (no browser automation)

```python
from solver.word_search import find_given_words_flexible

grid = [
    ['N', 'F', 'H', 'A', 'A', 'F'],
    ['C', 'I', 'W', 'K', 'L', 'T'],
    # ...
]
wordlist = ["HAWK", "FINCH", "GOOSE", "WREN", "THRUSH"]
results = find_given_words_flexible(grid, wordlist)
```

---

## Model Fine-Tuning

The semantic filtering model is Falcon-7B fine-tuned on ~150 historical NYT Strands puzzles using LoRA.

| Hyperparameter | Value |
|---|---|
| Base model | `tiiuae/falcon-7b` |
| Quantization | 4-bit nf4, double-quant |
| LoRA rank (r) | 16 |
| LoRA alpha | 32 |
| LoRA dropout | 0.05 |
| Learning rate | 2e-4 |
| Epochs | 21 |
| Effective batch size | 8 |

See `training/finetune_strands_with_grid.ipynb` for the full training pipeline.

---

## Limitations

- **Abstract themes** remain the hardest category (45% success rate) due to multi-layered metaphorical reasoning
- **Spangram identification** is a known weak point — no dedicated module yet
- Fine-tuning dataset is limited to ~150 puzzles; expanding to 500+ would improve recall
- NYT login / paywalls may affect live puzzle access

---

## Future Work

- Expand training data to 500+ puzzles
- Add a dedicated spangram detection module
- Replace plain DFS with A* or iterative deepening for faster path validation
- Explore vision transformer features for multi-modal theme understanding
- Add ensemble LLM voting to reduce model-specific biases

---

## Authors

Niharika Hariharan · Anagha Puvathingal · Shreya Rajeev · Carol Sachin Chopde

*Department of Computer Engineering, VJTI Mumbai — Academic Year 2025–26*

Project guide: Ms. Noshin Sabuwala

---

## Citation

If you use this work, please cite:

```bibtex
@techreport{hariharan2025strands,
  title     = {A Hybrid Semantic-Structural Framework for Theme-Constrained
               Word Discovery in Grid-Based Puzzles},
  author    = {Hariharan, Niharika and Puvathingal, Anagha and
               Rajeev, Shreya and Chopde, Carol Sachin},
  institution = {VJTI Mumbai},
  year      = {2025},
  month     = {November}
}
```

---

## License

MIT
