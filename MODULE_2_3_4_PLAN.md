# Plan: Lecture decks for Modules 2 (rework), 3 & 4 (new)

## Context

This is a UW open course, *Deep Learning for Medical Image Analysis*, delivered as
Beamer decks (elegant dark theme). The intended arc from `Lecture Outline.pdf`:

- **Module 1** (`main.tex`) — Fundamentals: imaging modalities, tasks, classical CV → NN → CNN → classification. *Complete.*
- **Module 2** (`module2.tex` + 3 Colab notebooks) — "How to train a CNN": Classification/ResNet, Segmentation/U-Net, Synthesis/VAE-GAN. *Complete and polished, but to be reworked.*
- **Module 3** — Beyond task-specific models: self-supervised learning (Transformer/ViT, Diffusion), foundation models (SAM2, BiomedParse). *Does not exist yet.*
- **Module 4** — Clinical integration & challenges: evaluation, privacy/federated learning, distribution shift/domain adaptation, explainability/uncertainty. *Does not exist yet.*

**Decisions made with the user:**
- Module 2: **rework/expand** — *deepen the existing 3 lectures*, add *pedagogical polish*, and *tighten/condense* redundancy (no new lecture topics).
- Modules 3 & 4: **new decks, one `.tex` per module** (`module3.tex`, `module4.tex`), each at **full Module-2 depth** (~8-9 content frames per lecture, signature TikZ diagrams, medical framing, pitfalls).
- **Slides only** — no new companion notebooks for Modules 3 & 4 for now.

**Outcome:** three self-contained decks that compile independently, share one visual
system, and chain via explicit "bridge" frames so the course reads as a continuous story.

---

## Shared conventions to reuse (do not reinvent)

Every deck is self-contained but reuses the same infrastructure. New decks should
**copy the `module2.tex` preamble verbatim** (lines 1–99) — it already establishes the
pattern:

- `\input{loadslides.tex}` — loads the `elegant` theme, `tikz`, **`pgfplots`** (`compat=1.18`),
  `biblatex` (biber backend, `references.bib`), and defines `\makepart{...}` for part dividers.
- Palette (module2.tex:29–43): accents `accTeal / accGreen / accAmber / accCoral`, backgrounds
  `bgTeal / bgGreen / bgAmber / bgCoral`, plus `accent / softblue / labelgreen`.
- Dark canvas + white text (module2.tex:26–27); left text margin 60mm (module2.tex:20–23).
- `medicalglass` tcolorbox style (module2.tex:86–99) — the standard "card".
- Helper commands `\boximageWide`, `\boximageCard`, `\HUGE` (module2.tex:45–57).
- `\makepart{Lecture N: Title}` before each lecture; per-lecture frame rhythm (see below).
- Figures live in `Figures/`; cite via `references.bib`. Reusable existing figures:
  `Figures/MedSAM.png` (Module 3 L3), `Figures/superResolution.png`, `Figures/cnn_model.png`,
  `Figures/segmentation.png`, `Figures/longitudinal.png`.

### Course-wide per-lecture frame rhythm (from Module 2, to be applied everywhere)

1. **Bridge/recap** — connect to the previous lecture/module.
2. **Motivation / the problem** — why this technique exists.
3. **Core idea** — signature TikZ diagram.
4. **Architecture** — the model laid out.
5. **Objective / loss** — the math.
6. **Evaluation / metrics** — how we judge it (medical framing).
7. **A second angle** — related technique / variant / medical case.
8. **In practice** — what to expect, compute, workflow (replaces Module 2's "hands-on notebook" frame for Modules 3 & 4, since those are slides-only).
9. **Pitfalls & recap** — failure modes + a one-line checkpoint.

**New pedagogical scaffolding to add to *every* lecture (course-wide consistency):**
a short **learning-objectives callout** on the opening frame ("By the end you can…") and a
one-line **recap checkpoint** on the pitfalls frame.

---

## Part A — Module 2 rework (`module2.tex`)

Keep the 3 lectures and the shared roadmap; apply the three chosen directions.

### A1. Tighten / condense (do first — removes redundancy before deepening)
- The two roadmap frames (`From "What is the task?"…` and `The Universal Training Recipe`,
  module2.tex:106–186) overlap. Keep both but trim the prose so the recipe frame is the
  single canonical reference; the roadmap frame becomes a lightweight map.
- The two near-identical **"Training loop, concretely"** frames (L1 module2.tex:371–397 and
  L2 module2.tex:733–759): keep both code blocks (they differ), but replace duplicated
  narration with a one-line "same recipe, swap model/loss/output" callback to the recipe frame.

### A2. Deepen the 3 existing lectures (add ~1–2 frames each)
- **L1 Classification:** add a **gradient-flow frame** — show why the skip connection lets
  gradients bypass `\mathcal{F}` (short derivation of `∂y/∂x = 1 + ∂F/∂x`), making the
  degradation fix concrete. Optionally a **medical case frame** reusing an existing figure.
- **L2 Segmentation:** add a **why-Dice-beats-CE frame** — contrast the gradient behaviour of
  pixelwise CE vs Dice on a tiny lesion (ties back to the class-imbalance pitfall at
  module2.tex:799). Optionally extend to **multi-class / soft Dice**.
- **L3 Synthesis:** add an **ELBO derivation frame** connecting the two loss terms
  (module2.tex:905–930) to `log p(x)` more rigorously, and a **conditional-generation teaser**
  that explicitly sets up Module 3's diffusion lecture.

### A3. Pedagogical polish
- Add the **learning-objectives callout** to each lecture's first frame and a **recap
  checkpoint** line to each pitfalls frame (the a/b scaffolding above).
- Add one **"quick check" question frame** per lecture (a conceptual prompt, answer on reveal)
  — e.g. "Why does predicting all-background score 95%?" for L2.
- Strengthen the closing bridge (module2.tex:1076) into Module 3.

**Net effect:** longer where it teaches, leaner where it repeated itself; consistent
open/close scaffolding shared with the new modules. Target ~+4–7 frames net (tighten first,
spend the budget on teaching frames). If strict length-neutrality is wanted, cut one frame
per frame added.

---

## Part B — Module 3 (new `module3.tex`): Beyond Task-Specific Models

Subtitle: *Self-Supervision, Diffusion & Foundation Models*. Open with a **module roadmap
frame** (3 lecture cards, mirroring module2.tex:106) and the recurring "what changes" framing:
*labels → self-supervision*, *discriminative → generative at scale*, *task-specific → promptable*.

### Lecture 1 — Self-Supervised Learning & the Transformer (ViT)
1. Bridge: Module 2 trained supervised CNNs on **scarce labels** → the labeling bottleneck.
2. Self-supervised idea: learn from unlabeled data via pretext tasks.
3. From convolution's locality to **self-attention** (signature TikZ: Q·Kᵀ → softmax → ·V).
4. The **Transformer block** — multi-head attention + MLP + residual/LayerNorm (callback to M2 residual block).
5. **Vision Transformer**: patchify → tokens (+pos-emb) → transformer → head.
6. SSL objectives: **contrastive** (SimCLR/DINO) vs **masked image modeling** (MAE).
7. Medical framing: SSL-pretrain on unlabeled scans, fine-tune on a small labeled set.
8. In practice: data/compute needs; when ViT beats a CNN (and when it doesn't).
9. Pitfalls (data-hungry, collapse) & recap.

### Lecture 2 — Diffusion Models
1. Bridge: M2 synthesis (VAE blurry, GAN unstable) → diffusion as the modern answer.
2. **Forward process**: gradually add Gaussian noise (signature TikZ noise ladder).
3. **Reverse process**: learn to denoise step by step.
4. Objective: **predict the noise** (simple MSE); U-Net backbone (callback to M2 U-Net).
5. **Sampling**: iterative denoising from pure noise.
6. **Conditioning**: class/text/image guidance; latent diffusion.
7. Medical applications: synthesis, denoising, **super-resolution**, MRI reconstruction, modality translation (reuse `Figures/superResolution.png`).
8. Evaluation & compute: FID, sampling cost, why it's GPU-heavy.
9. Pitfalls (hallucinated anatomy — never diagnose on synthetic) & recap.

### Lecture 3 — Foundation Models for Medical Analysis (SAM2, BiomedParse)
1. Bridge: task-specific → **one model, many tasks**; the foundation-model paradigm.
2. What makes a foundation model: scale, pretraining, **promptability**, zero/few-shot.
3. **Segment Anything (SAM/SAM2)**: image encoder + prompt encoder + mask decoder (signature TikZ).
4. **Prompts**: points/boxes/masks; interactive segmentation; SAM2 memory/video.
5. **SAM in medicine**: MedSAM, the domain gap, fine-tuning (reuse `Figures/MedSAM.png`).
6. **BiomedParse**: text-promptable joint segmentation/detection/recognition across modalities.
7. Zero-shot vs fine-tune trade-offs; where foundation models actually help.
8. In practice: prompt design, human-in-the-loop annotation workflows.
9. Pitfalls (domain shift, over-trust) & recap → **bridge to Module 4**.

---

## Part C — Module 4 (new `module4.tex`): Clinical Integration & Challenges

Subtitle: *From Benchmarks to the Bedside*. Open with a **module roadmap frame** (4 lecture
cards). These are more conceptual than Modules 2–3, but still rendered at Module-2 depth with
signature diagrams. **4 lectures × ~8 frames.**

### Lecture 1 — Evaluation Challenges
1. Bridge: a model that wins on a benchmark can fail in clinic — why.
2. Metrics revisited: accuracy/AUC/Dice can mislead (callback to M2 metrics frames).
3. **Dataset shift & external validation**: single-site vs multi-site.
4. **Data leakage** & patient-level splits (deepen the M2 pitfall).
5. Clinical vs statistical significance; **decision-curve / net benefit**.
6. Ground-truth ambiguity: reader studies, inter-rater variability.
7. **Reporting standards** (CLAIM, TRIPOD-AI; prospective validation).
8. Case study of a real failure + recap.

### Lecture 2 — Data Privacy & Federated Learning
1. Bridge: medical data is siloed & sensitive → can't just pool it.
2. Privacy risks: re-identification, membership inference.
3. **Federated learning** idea (signature TikZ: local train, share weights not data).
4. **FedAvg** algorithm; communication rounds.
5. **Non-IID** data across sites (links to L3), system heterogeneity.
6. **Differential privacy** & secure aggregation.
7. Real multi-hospital deployments.
8. Pitfalls & recap.

### Lecture 3 — Distribution Shift: Skew-Distributed Data & Domain Adaptation
1. Bridge: train and deploy distributions differ.
2. Types of shift: covariate / label / concept; scanner/site/protocol differences.
3. **Class imbalance / long-tail** (deepen from M1/M2): resampling, reweighting, focal loss.
4. **Domain adaptation** idea (signature TikZ: align source & target features).
5. Techniques: augmentation, domain-adversarial, normalization, **test-time adaptation**.
6. **Harmonization** for medical imaging (intensity normalization, style transfer).
7. Evaluating under shift.
8. Pitfalls & recap.

### Lecture 4 — Explainability & Uncertainty
1. Bridge: clinicians must trust and understand the model.
2. Why black-box is a problem in medicine (regulatory, safety).
3. **Saliency/attribution**: Grad-CAM (signature TikZ heatmap), attention maps.
4. Limitations of saliency (unreliable; sanity checks).
5. **Uncertainty**: aleatoric vs epistemic.
6. Methods: MC dropout, ensembles, **calibration**.
7. **Selective prediction / defer-to-expert** (abstention).
8. **Course wrap-up**: tasks → training → foundation models → safe clinical integration + recap.

---

## Files to be created / modified

| File | Action |
|------|--------|
| `module2.tex` | Modify — tighten (A1), deepen (A2), polish (A3). ~+4–7 frames net. |
| `module3.tex` | **Create** — copy module2.tex preamble; 1 roadmap + 3 lectures (Part B). |
| `module4.tex` | **Create** — copy module2.tex preamble; 1 roadmap + 4 lectures (Part C). |
| `references.bib` | Extend — add ViT/DINO/MAE, DDPM/latent-diffusion, SAM2/MedSAM/BiomedParse, FedAvg/DP, Grad-CAM, TRIPOD-AI, etc. |

No new notebooks. Existing `notebooks/` and `main.tex` are untouched.

## Suggested execution order

1. Module 2 rework (A1 tighten → A2 deepen → A3 polish) — validates the shared scaffolding on a known-good deck.
2. `module3.tex` (reuses the polished patterns and figures).
3. `module4.tex` (reuses roadmap + card patterns).
4. Extend `references.bib` as each deck cites sources.

## Verification

TeX Live 2025 is installed (`pdflatex`, `latexmk`, `biber` all on PATH). For each deck:

```
latexmk -pdf module3.tex          # runs pdflatex + biber automatically
# or, since biblatex/biber is loaded via loadslides.tex:
pdflatex module3.tex && biber module3 && pdflatex module3.tex && pdflatex module3.tex
```

- Confirm each deck compiles to PDF with **no undefined references/citations** and no
  overfull-box warnings on the signature TikZ frames (the `pgfplots` axis frames and wide
  architecture diagrams are the most fragile — check those pages visually).
- Confirm `module2.pdf` still builds after the rework and page count changed as expected.
- Spot-check every `\includegraphics` resolves (all referenced figures exist in `Figures/`).
- Visually skim each deck for: dark-theme legibility, per-lecture learning-objectives +
  recap scaffolding present, and working bridge frames between modules.
