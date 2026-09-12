# Local Models Notes

Notes distilled from an `/lmg/` (Local Models General, 4chan `/g/`-style) thread
— news, getting-started guides, hardware builds, model recommendations, and
sampler/training references for running LLMs locally.

See [LINKS.md](LINKS.md) for the full raw link list, organized by section.

## News (as of the thread date, 2026-09-12)

- (09/10) YuE2 3B — 48 kHz stereo song generation and editing.
- (09/10) DeepSeek-V4.1-Flash — 552B total / 16B active (MoE), listed as
  "552B-A16B-P8B-N196B".
- (09/08) Ling-3.0-flash-VL — vision-language variant of Ling.
- (09/07) MiniCPM5-2B — small (2B) model release.
- (09/03) K2 Horizon — released across a wide size range: 375B-A23B (MoE),
  36B-A4B (MoE), 32B, 7B, 3.7B, and 0.9B — one family scaling from
  phone-sized to frontier-sized.

(Model links themselves are ephemeral/version-specific — see LINKS.md for the
exact HF URLs from the thread; check `rentry.org/recommended-models` for
current picks rather than trusting this snapshot later.)

## Getting started (if you're new to this)

- **Lazy getting-started guide**: for a single consumer GPU (e.g. a 3080),
  the path of least resistance is koboldcpp or text-generation-webui as the
  backend + SillyTavern as the frontend. Grab a GGUF quant (start at Q4) of a
  well-supported instruct model, load it, match the prompt template to the
  model family in the frontend, and start from sane defaults (temp ~0.8,
  Min-P ~0.02, repetition penalty ~1.2) rather than tuning blind.
- **Recommended models** (per the thread's rentry, will drift fast):
  Gemma 4 12B/31B called out as "gold standard for VRAMlets" with vision
  support; Qwen3.5/3.8 series across size tiers from 2B up to 27B; and
  much larger MoE models (Kimi K3, GLM 5.2, DeepSeek V4 Flash) for anyone
  with real server-grade VRAM. General shape: pick the biggest quant that
  fits your VRAM tier, not the newest-sounding name.
- **Samplers** — the actual levers and how they interact:
  - **Temperature** — raises/lowers randomness; apply *before* filtering for
    normal use, *after* filtering for creative writing.
  - **Top-K / Top-P / Min-P** — different ways of trimming the tail of the
    probability distribution; stacking all three is redundant — pick one or
    two complementary ones (e.g. Temperature + Min-P, or Top-K + Top-P).
  - **DRY** (Don't Repeat Yourself) — an n-gram repetition penalty, generally
    more effective than plain frequency/presence penalties for creative text.
  - **Mirostat** / **Dynamic Temperature** — self-adjusting alternatives to
    picking fixed sampler values by hand.
  - Rule of thumb pipeline: penalties → temperature → distribution filtering.
- **Mikupad** — a story-writing-first frontend (long-form narration with
  direct prompt control, Author's Note/Worldbook/Fill-in-Middle, token
  logprob visualization) as opposed to SillyTavern's round-based roleplay
  model. Different mental model: third-person narration/pacing rather than
  in-character dialogue exchange.

## Hardware

From the build guides:
- **~$1K budget tier**: "Mikubox Triple-P40" — three P40s, runs 70B models at
  8K context in 6-bit quantization.
- **CPU-centric high-end**: "Miqumaxx" — optimizes for memory bandwidth
  instead of GPU VRAM.
- **VRAM-dense GPU tier**: multi-V100 setups, or "Tetobox" (dual/triple Intel
  A770s, 32-48GB VRAM) as a cheaper-per-GB alternative to Nvidia cards.
- General shape: quantization level and VRAM headroom (not raw GPU count)
  are the two things that actually decide what you can run.

## Running it safely (worth flagging)

**IsolatedLinuxWebService** guide is genuinely useful operational security
advice, not just local-models trivia: run the web UI (e.g. oobabooga) as a
locked-down, non-interactive system user with `systemd`'s
`IPAddressDeny=any` / `IPAddressAllow=localhost`, so the process can't reach
the internet at all — worth doing specifically because build tooling like
CMake (used to build llama.cpp) fetches dependencies at build time, which is
a real supply-chain attack surface. Tradeoff: the web UI's in-app
plugin-install/update features stop working too — you do that manually
instead. This generalizes well beyond local LLMs to "any web service you
don't fully trust the code of."

## Learning path (if you want to actually understand this, not just run it)

1. **Math**: linear algebra, calculus (derivatives/gradients), probability
   & statistics.
2. **Python**: syntax → NumPy/Pandas/Matplotlib → a real framework
   (PyTorch recommended) → OOP.
3. **ML concepts**: supervised/unsupervised/reinforcement learning; the
   actual algorithms (regression, trees, SVMs) up through CNNs/RNNs/
   Transformers.
4. **Practice**: pre-train or fine-tune something yourself with
   Transformers + PyTorch.
5. **Stay current**: arXiv, community, conferences — this field moves fast
   enough that step 5 never really ends.

The guide's explicit warning: don't skip stages — each one is load-bearing
for the next.

## Training/fine-tuning, if you want to go further than running a GGUF

- **Pre-training**: from-scratch, self-supervised, terabyte-scale — not
  something you do on a hobbyist budget.
- **Fine-tuning**: adapts a pretrained model on a much smaller task-specific
  dataset via gradient-based optimization (learning-rate scheduling,
  dropout against overfitting).
- **LoRA / QLoRA**: the actually-accessible path — LoRA can cut trainable
  parameters ~10,000x and memory ~3x+ versus full fine-tuning; QLoRA pushes
  billion-parameter training onto consumer hardware. **Axolotl** is the
  recommended trainer.
- The practical bottleneck is almost always the dataset (gathering,
  cleaning, structuring — including parsing raw HTML/CSV/SQL sources), not
  the training loop itself.

## Benchmarks worth knowing about

- **LiveBench** — general LLM capability leaderboard.
- **swe-rebench** — programming/SWE-task benchmark.
- **deepswe (datacurve)** — agentic coding benchmark.
- **NoLiMa** — tests real long-context understanding (as opposed to
  needle-in-haystack recall, which many models game without truly using
  long context).
- **GPU-Benchmarks-on-LLM-Inference** — cross-GPU inference throughput
  comparisons, directly useful for hardware-buying decisions above.

## Research papers, if you want to go deeper

The paper list (LocalModelsPapers) spans: attention efficiency (FlashAttention,
state-space alternatives like Mamba), quantization/pruning/distillation for
running smaller models locally, parameter-efficient fine-tuning (LoRA family),
long-context extension techniques, multimodal (vision/audio) integration,
inference-speed tricks (speculative decoding, early exit), and RLHF/preference-
optimization alternatives for training on a budget.
