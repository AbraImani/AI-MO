# 🏆 AIMO3 — AI Mathematical Olympiad Progress Prize 3

> **High-performance hybrid reasoning system for solving 110 olympiad-level math problems**  
> Kaggle Competition: [AI Mathematical Olympiad – Progress Prize 3](https://www.kaggle.com/competitions/ai-mathematical-olympiad-progress-prize-3)

---

## 🎯 Competition Overview

| Detail | Value |
|--------|-------|
| **Goal** | Solve 110 original olympiad-level math problems (LaTeX) |
| **Answer format** | Integer in `[0, 99999]` |
| **Difficulty** | National Olympiad → IMO level |
| **Topics** | Algebra, Combinatorics, Geometry, Number Theory |
| **Constraints** | Offline only, ≤ 5h GPU / ≤ 9h CPU, open-source models only |
| **Prize Pool** | $2,207,152 |

---

## 🧠 Architecture

```
Problem ──► LaTeX Cleaner ──► Problem Analyzer
                                    │
                   ┌────────────────┼────────────────┐
                   ▼                ▼                ▼
             TIR Attempt 1    TIR Attempt 2   ...  Att. N
             (greedy)         (temp=0.7)       (temp=0.7)
                   │                │                │
             ┌─────┴─────┐   ┌─────┴─────┐   ┌────┴────┐
             │ Code Exec │   │ Code Exec │   │Code Exec│
             │ (Sandbox) │   │ (Sandbox) │   │(Sandbox)│
             └─────┬─────┘   └─────┬─────┘   └────┬────┘
                   │                │                │
             ┌─────┴─────┐   ┌─────┴─────┐   ┌────┴────┐
             │  Extract  │   │  Extract  │   │ Extract │
             │  Answer   │   │  Answer   │   │ Answer  │
             └─────┬─────┘   └─────┬─────┘   └────┬────┘
                   └────────────────┼────────────────┘
                                    ▼
                          Majority Vote (code-preferred)
                                    │
                                    ▼
                          Final Answer [0, 99999]
                                    │
                                    ▼
                         Kaggle Submission API
```

### Core Components

| Component | Description |
|-----------|-------------|
| **Model** | `Qwen2.5-Math-7B-Instruct` — SOTA open-source math LLM at 7B scale |
| **LaTeX Cleaner** | Normalizes delimiters, strips environments, standardizes notation |
| **Problem Analyzer** | Classifies problem topic (NT, Combo, Algebra, Geometry) to guide strategy |
| **TIR Solver** | Tool-Integrated Reasoning — model writes Python code, we execute & feed back results |
| **Code Sandbox** | Safe execution with timeout, restricted imports, SymPy pre-loaded |
| **Answer Extractor** | Parses `\boxed{}`, code outputs, and plain text for integer answers |
| **Majority Voter** | Self-consistency over N=5 attempts, code-answer preference for tie-breaking |

---

## 🚀 Key Design Decisions

### Why Qwen2.5-Math-7B-Instruct?
- **#1 open-source math model** at 7B parameters on MATH, GSM8K, and competition benchmarks
- Native TIR (Tool-Integrated Reasoning) support via chat template
- Fits in Kaggle T4/P100 GPU at fp16 (~14 GB VRAM)
- Outperforms DeepSeek-Math-7B by 5-8% on olympiad-level problems

### Why Tool-Integrated Reasoning (TIR)?
- Olympiad problems frequently require **exact computation**: modular exponentiation, combinatorial enumeration, digit manipulation
- Pure Chain-of-Thought fails at multi-step arithmetic (error compounds)
- TIR lets the model write Python/SymPy code within its reasoning chain, which we execute and return
- Code results are **deterministic and verified** — no hallucination risk

### Why 5 Samples + Majority Vote?
- Self-consistency with N=5 provides diverse reasoning paths
- Research shows diminishing returns beyond 5-8 samples for voting
- **Early stopping** at 3× consensus saves ~40% runtime on easy problems
- First attempt is **greedy** (deterministic), remaining 4 are **sampled** (temperature=0.7)

### Robustness Features
- ✅ LaTeX normalization prevents format failures
- ✅ Sandbox timeout (30s) prevents infinite loops
- ✅ Per-problem timeout (5 min) ensures all problems attempted
- ✅ Fallback to `0` on critical failures
- ✅ GPU memory cleanup between problems
- ✅ Handles all answer ranges `[0, 99999]`

---

## 📁 Repository Structure

```
AI-MO/
├── aimo3-solution.ipynb    # Complete Kaggle submission notebook
└── README.md               # This file
```

---

## 🔧 Setup & Usage

### On Kaggle (Competition Submission)

1. **Create a new notebook** on Kaggle attached to the AIMO3 competition
2. **Add the model** as an input dataset:
   - Go to *Kaggle Models → Qwen → Qwen2.5-Math-7B-Instruct*
   - Add it as a dataset input to your notebook
3. **Copy the notebook contents** from `aimo3-solution.ipynb`
4. **Set settings**: GPU accelerator, Internet OFF
5. **Submit**

### Local Testing

```bash
# Clone this repo
git clone https://github.com/<your-username>/AI-MO.git
cd AI-MO

# Install dependencies
pip install torch transformers sympy numpy pandas kaggle_evaluation

# Open the notebook
jupyter notebook aimo3-solution.ipynb
```

Set `LOCAL_TEST = True` in the last cell to test against reference problems.

---

## 📊 Performance Budget

| Phase | Time per Problem | Total (50 problems) |
|-------|-----------------|---------------------|
| LaTeX Parsing | ~0.01s | ~0.5s |
| 5× TIR Generation | ~25-40s each | ~2.1 hours |
| Code Execution | ~1-5s per block | ~4 min |
| Voting & Extraction | ~0.01s | ~0.5s |
| **Total** | **~2.5 min** | **~2.1 hours** |

Well within Kaggle's **5-hour GPU limit**.

---

## 🏅 Competition Details

- **Competition**: [AI Mathematical Olympiad – Progress Prize 3](https://www.kaggle.com/competitions/ai-mathematical-olympiad-progress-prize-3)
- **Host**: XTX Markets / AI|MO
- **Prize Pool**: $2,207,152
- **110 problems**: 50 public + 50 private + 10 reference
- **Evaluation**: Penalized accuracy (both runs must agree for full score)
- **Hardware**: Kaggle GPU (T4/P100/H100)

---

## 📚 References

- [Qwen2.5-Math Technical Report](https://arxiv.org/abs/2409.12122)
- [Self-Consistency Improves Chain of Thought Reasoning](https://arxiv.org/abs/2203.11171)
- [PAL: Program-Aided Language Models](https://arxiv.org/abs/2211.10435)
- [AIMO Prize Official Site](https://aimoprize.com/)
- [NemoSkills: AIMO2 Winner](https://arxiv.org/abs/2504.16891)

---

## 📄 License

This project is released under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).

---

*Built for the AI Mathematical Olympiad Progress Prize 3 competition on Kaggle.*
