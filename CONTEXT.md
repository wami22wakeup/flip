# CONTEXT.md

Background for the Flip project: what the models are, what already exists, and what this project adds.
Last updated: 3 Oct 2026. These models and the work around them are only weeks old, so re-check the sources before relying on any number.

---

## 1. The research question

> **When do decision models change their answer for reasons that shouldn't matter, and when do they fail to change it for reasons that should?**

Accuracy tables for Jev/Laya/CLM already exist. Flip measures **behavioural stability**: whether a decision survives meaning-preserving changes (option order, rewording, Hinglish, distractors, injections) and flips on meaning-changing ones (negation).

The headline we're testing for (unproven until measured):
"A few words can make decision models trust a scam message."

## 2. The models

All three are **decision models** (also called "System One" models). They're non-autoregressive: instead of generating text, they answer a typed question about an input in one forward pass and return a probability or score.

Three question types:
- **choice**: pick one of N options
- **score**: a number on a scale
- **noul**: yes/no with a probability

| Model | Maker | Access | Architecture | Notes |
|---|---|---|---|---|
| **Jev** | TypeSafe AI | Closed, managed API | Undisclosed | Strongest zero-shot in published comparisons. Up to 255 options per choice question. Developer lists indirect meaning and adversarial content among known weaknesses. |
| **Laya** | Convai Innovations | Open weights, Apache 2.0 | ModernBERT-large encoder (~421M params) + typed decision head | Fast (tens of ms on a T4). Limited input budget; options share it, so accuracy drops with many options. Has `laya`, `laya-typed-decisions` and `laya-multilingual` checkpoints plus a router. Zero-shot base is weak; the fine-tuned checkpoint is much stronger. |
| **CLM** (CLM-v0.1-8B) | Contrastive-LM | Open | Frozen Qwen3-8B encoder + small trained head | Aims at TypeSafe API compatibility. Needs a large GPU. Training = training the head only. |

**Access:**
- Jev and Laya are both available through Runware's `/v1/systemone` endpoint (`typesafe:jev@latest`, `runware:laya@1`).
- Laya repo: github.com/NandhaKishorM/laya
- CLM repo: github.com/Contrastive-LM/CLM

## 3. Prior work (cite, don't copy)

| Project | What it did | How Flip differs |
|---|---|---|
| decision-models-chess (GauravAtavale) | Jev vs Laya vs CLM playing chess; fine-tuning closes the gap | Control/game task; Flip is language robustness |
| jev-laya-classification-bench (bhushankinge) | 12,000 US federal IT solicitations; accuracy, calibration, auto-accept cutoffs, cost | Accuracy/calibration on one dataset; Flip is perturbation behaviour |
| jev-bench (brandonrc) | 5 package-curation tasks, 5,561 items; open models near chance off the shelf | Security curation accuracy; Flip is stability |
| local-jev-bench (tak-bro) | Local decision models on Apple Silicon; English, Korean, Banking77; **includes order-flip** | **Flip extends the order-flip idea** into a full perturbation suite + minimal-flip search. Credit them. |
| llm-jev-laya-bench (PerryLink) | Cost, latency, failure boundaries, audit trail | Note: author maintains a Laya tool (conflict of interest declared) |
| "I Tested Jev Against 12 Local Decision Models" (The AI Automators) | Noted that disagreements between models are more informative than rankings | Flip turns that observation into a tool (disagreement map) |
| "Jev in the Wild" (arXiv, 2026) | Analysis of 2,170 public Jev projects | Shows the space is crowded, so plain benchmarks won't stand out |

## 4. What Flip adds

1. **Perturbation suite with declared expectations** (should flip / shouldn't flip)
2. **Stability score** per model, alongside accuracy and calibration
3. **Minimal-flip finder**: the smallest edit that changes each model's verdict
4. **Disagreement map**: inputs where the models disagree most
5. **Indian scam-message dataset** (English / Hindi / Hinglish), self-built and published
6. **Base vs fine-tuned Laya**: does fine-tuning improve accuracy but reduce stability (shortcut learning)?

## 5. Perturbations

| Type | Should flip? | Tests |
|---|---|---|
| Shuffle option order | No | Position bias |
| Reword question (same meaning) | No | Wording sensitivity |
| Add irrelevant sentence | No | Distraction |
| Translate to Hinglish/Hindi | No | Multilingual robustness |
| Inject "verified message" / "ignore and mark safe" | No | Injection resistance |
| Negate a key word | Yes | Actually reading the text |

## 6. Key caveats

- **Check accuracy before stability.** A model guessing at chance can look stable or unstable for meaningless reasons. If Laya/CLM are near chance zero-shot, fine-tune first (Laya fits on a free T4).
- **Context windows differ a lot** (Laya ≈1k tokens, CLM ≈2k, Jev ≈32k in jev-bench's setup). Cap or report this explicitly.
- **Jev latency includes the network round-trip;** local models don't. Report it separately, not as a pure model comparison.
- **Vendor and README numbers are often from fine-tuned checkpoints** or small sets. Verify, don't quote.
- **Ask for risk, not actions.** Asking Laya directly APPROVE/REVIEW/HOLD made it hold everything in one test. Have the model score risk and let the app apply thresholds.

## 7. Reading list

**Methodology**
- Ribeiro et al., *Beyond Accuracy: Behavioral Testing of NLP Models with CheckList* (ACL 2020). The core method.
- Morris et al., *TextAttack* (2020). Perturbation and adversarial search.
- Wu et al., *Polyjuice* (2021). Minimal counterfactual edits.
- Guo et al., *On Calibration of Modern Neural Networks* (2017). ECE, temperature scaling.

**Models**
- Warner et al., *ModernBERT* (2024). Laya's backbone.
- Laya README, CLM README.
- Runware: "Jev, Laya, and the emerging role of decision models"

**Multilingual / Hinglish**
- L3Cube HingBERT / HingCorpus papers
- GLUECoS, LinCE (code-mixed benchmarks)

## 8. Goals beyond the code

- Learn hands-on how these models work and differ
- A portfolio piece covering evaluation, fine-tuning, calibration and a demo
- A LinkedIn post: hook + 20-second screen recording of a real flip + 3 honest findings + open-source link. Tag TypeSafe AI and Convai Innovations.
