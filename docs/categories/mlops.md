# Category: mlops

> Deep-dive for the **7 skills** in the `mlops` category. Each section embeds the full `SKILL.md` body so this doc is self-contained.

**Skills in this category:**

- [`audiocraft-audio-generation`](../skills/mlops/audiocraft-audio-generation.md) - AudioCraft: MusicGen text-to-music, AudioGen text-to-sound.
- [`evaluating-llms-harness`](../skills/mlops/evaluating-llms-harness.md) - lm-eval-harness: benchmark LLMs (MMLU, GSM8K, etc.).
- [`huggingface-hub`](../skills/mlops/huggingface-hub.md) - HuggingFace hf CLI: search/download/upload models, datasets.
- [`llama-cpp`](../skills/mlops/llama-cpp.md) - llama.cpp local GGUF inference + HF Hub model discovery.
- [`segment-anything-model`](../skills/mlops/segment-anything-model.md) - SAM: zero-shot image segmentation via points, boxes, masks.
- [`serving-llms-vllm`](../skills/mlops/serving-llms-vllm.md) - vLLM: high-throughput LLM serving, OpenAI API, quantization.
- [`weights-and-biases`](../skills/mlops/weights-and-biases.md) - W&B: log ML experiments, sweeps, model registry, dashboards.

---

## audiocraft-audio-generation

- **Name:** `audiocraft-audio-generation`
- **Version:** 1.0.0
- **Author:** Orchestra Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\models\audiocraft/SKILL.md`

**Description:**

> AudioCraft: MusicGen text-to-music, AudioGen text-to-sound.

**Full SKILL.md body:**

```markdown
---
name: audiocraft-audio-generation
description: "AudioCraft: MusicGen text-to-music, AudioGen text-to-sound."
version: 1.0.0
author: Orchestra Research
license: MIT
dependencies: [audiocraft, torch>=2.0.0, transformers>=4.30.0]
platforms: [linux, macos]
metadata:
  hermes:
    tags: [Multimodal, Audio Generation, Text-to-Music, Text-to-Audio, MusicGen]

---

# AudioCraft: Audio Generation

Comprehensive guide to using Meta's AudioCraft for text-to-music and text-to-audio generation with MusicGen, AudioGen, and EnCodec.

## When to use AudioCraft

**Use AudioCraft when:**
- Need to generate music from text descriptions
- Creating sound effects and environmental audio
- Building music generation applications
- Need melody-conditioned music generation
- Want stereo audio output
- Require controllable music generation with style transfer

**Key features:**
- **MusicGen**: Text-to-music generation with melody conditioning
- **AudioGen**: Text-to-sound effects generation
- **EnCodec**: High-fidelity neural audio codec
- **Multiple model sizes**: Small (300M) to Large (3.3B)
- **Stereo support**: Full stereo audio generation
- **Style conditioning**: MusicGen-Style for reference-based generation

**Use alternatives instead:**
- **Stable Audio**: For longer commercial music generation
- **Bark**: For text-to-speech with music/sound effects
- **Riffusion**: For spectogram-based music generation
- **OpenAI Jukebox**: For raw audio generation with lyrics

## Quick start

### Installation

```bash
# From PyPI
pip install audiocraft

# From GitHub (latest)
pip install git+https://github.com/facebookresearch/audiocraft.git

# Or use HuggingFace Transformers
pip install transformers torch torchaudio
```

### Basic text-to-music (AudioCraft)

```python
import torchaudio
from audiocraft.models import MusicGen

# Load model
model = MusicGen.get_pretrained('facebook/musicgen-small')

# Set generation parameters
model.set_generation_params(
    duration=8,  # seconds
    top_k=250,
    temperature=1.0
)

# Generate from text
descriptions = ["happy upbeat electronic dance music with synths"]
wav = model.generate(descriptions)

# Save audio
torchaudio.save("output.wav", wav[0].cpu(), sample_rate=32000)
```

### Using HuggingFace Transformers

```python
from transformers import AutoProcessor, MusicgenForConditionalGeneration
import scipy

# Load model and processor
processor = AutoProcessor.from_pretrained("facebook/musicgen-small")
model = MusicgenForConditionalGeneration.from_pretrained("facebook/musicgen-small")
model.to("cuda")

# Generate music
inputs = processor(
    text=["80s pop track with bassy drums and synth"],
    padding=True,
    return_tensors="pt"
).to("cuda")

audio_values = model.generate(
    **inputs,
    do_sample=True,
    guidance_scale=3,
    max_new_tokens=256
)

# Save
sampling_rate = model.config.audio_encoder.sampling_rate
scipy.io.wavfile.write("output.wav", rate=sampling_rate, data=audio_values[0, 0].cpu().numpy())
```

### Text-to-sound with AudioGen

```python
from audiocraft.models import AudioGen

# Load AudioGen
model = AudioGen.get_pretrained('facebook/audiogen-medium')

model.set_generation_params(duration=5)

# Generate sound effects
descriptions = ["dog barking in a park with birds chirping"]
wav = model.generate(descriptions)

torchaudio.save("sound.wav", wav[0].cpu(), sample_rate=16000)
```

## Core concepts

### Architecture overview

```
AudioCraft Architecture:
┌──────────────────────────────────────────────────────────────┐
│                    Text Encoder (T5)                          │
│                         │                                     │
│                    Text Embeddings                            │
└────────────────────────┬─────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────────┐
│              Transformer Decoder (LM)                         │
│     Auto-regressively generates audio tokens                  │
│     Using efficient token interleaving patterns               │
└────────────────────────┬─────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────────┐
│                EnCodec Audio Decoder                          │
│        Converts tokens back to audio waveform                 │
└──────────────────────────────────────────────────────────────┘
```

### Model variants

| Model | Size | Description | Use Case |
|-------|------|-------------|----------|
| `musicgen-small` | 300M | Text-to-music | Quick generation |
| `musicgen-medium` | 1.5B | Text-to-music | Balanced |
| `musicgen-large` | 3.3B | Text-to-music | Best quality |
| `musicgen-melody` | 1.5B | Text + melody | Melody conditioning |
| `musicgen-melody-large` | 3.3B | Text + melody | Best melody |
| `musicgen-stereo-*` | Varies | Stereo output | Stereo generation |
| `musicgen-style` | 1.5B | Style transfer | Reference-based |
| `audiogen-medium` | 1.5B | Text-to-sound | Sound effects |

### Generation parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `duration` | 8.0 | Length in seconds (1-120) |
| `top_k` | 250 | Top-k sampling |
| `top_p` | 0.0 | Nucleus sampling (0 = disabled) |
| `temperature` | 1.0 | Sampling temperature |
| `cfg_coef` | 3.0 | Classifier-free guidance |

## MusicGen usage

### Text-to-music generation

```python
from audiocraft.models import MusicGen
import torchaudio

model = MusicGen.get_pretrained('facebook/musicgen-medium')

# Configure generation
model.set_generation_params(
    duration=30,          # Up to 30 seconds
    top_k=250,            # Sampling diversity
    top_p=0.0,            # 0 = use top_k only
    temperature=1.0,      # Creativity (higher = more varied)
    cfg_coef=3.0          # Text adherence (higher = stricter)
)

# Generate multiple samples
descriptions = [
    "epic orchestral soundtrack with strings and brass",
    "chill lo-fi hip hop beat with jazzy piano",
    "energetic rock song with electric guitar"
]

# Generate (returns [batch, channels, samples])
wav = model.generate(descriptions)

# Save each
for i, audio in enumerate(wav):
    torchaudio.save(f"music_{i}.wav", audio.cpu(), sample_rate=32000)
```

### Melody-conditioned generation

```python
from audiocraft.models import MusicGen
import torchaudio

# Load melody model
model = MusicGen.get_pretrained('facebook/musicgen-melody')
model.set_generation_params(duration=30)

# Load melody audio
melody, sr = torchaudio.load("melody.wav")

# Generate with melody conditioning
descriptions = ["acoustic guitar folk song"]
wav = model.generate_with_chroma(descriptions, melody, sr)

torchaudio.save("melody_conditioned.wav", wav[0].cpu(), sample_rate=32000)
```

### Stereo generation

```python
from audiocraft.models import MusicGen

# Load stereo model
model = MusicGen.get_pretrained('facebook/musicgen-stereo-medium')
model.set_generation_params(duration=15)

descriptions = ["ambient electronic music with wide stereo panning"]
wav = model.generate(descriptions)

# wav shape: [batch, 2, samples] for stereo
print(f"Stereo shape: {wav.shape}")  # [1, 2, 480000]
torchaudio.save("stereo.wav", wav[0].cpu(), sample_rate=32000)
```

### Audio continuation

```python
from transformers import AutoProcessor, MusicgenForConditionalGeneration

processor = AutoProcessor.from_pretrained("facebook/musicgen-medium")
model = MusicgenForConditionalGeneration.from_pretrained("facebook/musicgen-medium")

# Load audio to continue
import torchaudio
audio, sr = torchaudio.load("intro.wav")

# Process with text and audio
inputs = processor(
    audio=audio.squeeze().numpy(),
    sampling_rate=sr,
    text=["continue with a epic chorus"],
    padding=True,
    return_tensors="pt"
)

# Generate continuation
audio_values = model.generate(**inputs, do_sample=True, guidance_scale=3, max_new_tokens=512)
```

## MusicGen-Style usage

### Style-conditioned generation

```python
from audiocraft.models import MusicGen

# Load style model
model = MusicGen.get_pretrained('facebook/musicgen-style')

# Configure generation with style
model.set_generation_params(
    duration=30,
    cfg_coef=3.0,
    cfg_coef_beta=5.0  # Style influence
)

# Configure style conditioner
model.set_style_conditioner_params(
    eval_q=3,          # RVQ quantizers (1-6)
    excerpt_length=3.0  # Style excerpt length
)

# Load style reference
style_audio, sr = torchaudio.load("reference_style.wav")

# Generate with text + style
descriptions = ["upbeat dance track"]
wav = model.generate_with_style(descriptions, style_audio, sr)
```

### Style-only generation (no text)

```python
# Generate matching style without text prompt
model.set_generation_params(
    duration=30,
    cfg_coef=3.0,
    cfg_coef_beta=None  # Disable double CFG for style-only
)

wav = model.generate_with_style([None], style_audio, sr)
```

## AudioGen usage

### Sound effect generation

```python
from audiocraft.models import AudioGen
import torchaudio

model = AudioGen.get_pretrained('facebook/audiogen-medium')
model.set_generation_params(duration=10)

# Generate various sounds
descriptions = [
    "thunderstorm with heavy rain and lightning",
    "busy city traffic with car horns",
    "ocean waves crashing on rocks",
    "crackling campfire in forest"
]

wav = model.generate(descriptions)

for i, audio in enumerate(wav):
    torchaudio.save(f"sound_{i}.wav", audio.cpu(), sample_rate=16000)
```

## EnCodec usage

### Audio compression

```python
from audiocraft.models import CompressionModel
import torch
import torchaudio

# Load EnCodec
model = CompressionModel.get_pretrained('facebook/encodec_32khz')

# Load audio
wav, sr = torchaudio.load("audio.wav")

# Ensure correct sample rate
if sr != 32000:
    resampler = torchaudio.transforms.Resample(sr, 32000)
    wav = resampler(wav)

# Encode to tokens
with torch.no_grad():
    encoded = model.encode(wav.unsqueeze(0))
    codes = encoded[0]  # Audio codes

# Decode back to audio
with torch.no_grad():
    decoded = model.decode(codes)

torchaudio.save("reconstructed.wav", decoded[0].cpu(), sample_rate=32000)
```

## Common workflows

### Workflow 1: Music generation pipeline

```python
import torch
import torchaudio
from audiocraft.models import MusicGen

class MusicGenerator:
    def __init__(self, model_name="facebook/musicgen-medium"):
        self.model = MusicGen.get_pretrained(model_name)
        self.sample_rate = 32000

    def generate(self, prompt, duration=30, temperature=1.0, cfg=3.0):
        self.model.set_generation_params(
            duration=duration,
            top_k=250,
            temperature=temperature,
            cfg_coef=cfg
        )

        with torch.no_grad():
            wav = self.model.generate([prompt])

        return wav[0].cpu()

    def generate_batch(self, prompts, duration=30):
        self.model.set_generation_params(duration=duration)

        with torch.no_grad():
            wav = self.model.generate(prompts)

        return wav.cpu()

    def save(self, audio, path):
        torchaudio.save(path, audio, sample_rate=self.sample_rate)

# Usage
generator = MusicGenerator()
audio = generator.generate(
    "epic cinematic orchestral music",
    duration=30,
    temperature=1.0
)
generator.save(audio, "epic_music.wav")
```

### Workflow 2: Sound design batch processing

```python
import json
from pathlib import Path
from audiocraft.models import AudioGen
import torchaudio

def batch_generate_sounds(sound_specs, output_dir):
    """
    Generate multiple sounds from specifications.

    Args:
        sound_specs: list of {"name": str, "description": str, "duration": float}
        output_dir: output directory path
    """
    model = AudioGen.get_pretrained('facebook/audiogen-medium')
    output_dir = Path(output_dir)
    output_dir.mkdir(exist_ok=True)

    results = []

    for spec in sound_specs:
        model.set_generation_params(duration=spec.get("duration", 5))

        wav = model.generate([spec["description"]])

        output_path = output_dir / f"{spec['name']}.wav"
        torchaudio.save(str(output_path), wav[0].cpu(), sample_rate=16000)

        results.append({
            "name": spec["name"],
            "path": str(output_path),
            "description": spec["description"]
        })

    return results

# Usage
sounds = [
    {"name": "explosion", "description": "massive explosion with debris", "duration": 3},
    {"name": "footsteps", "description": "footsteps on wooden floor", "duration": 5},
    {"name": "door", "description": "wooden door creaking and closing", "duration": 2}
]

results = batch_generate_sounds(sounds, "sound_effects/")
```

### Workflow 3: Gradio demo

```python
import gradio as gr
import torch
import torchaudio
from audiocraft.models import MusicGen

model = MusicGen.get_pretrained('facebook/musicgen-small')

def generate_music(prompt, duration, temperature, cfg_coef):
    model.set_generation_params(
        duration=duration,
        temperature=temperature,
        cfg_coef=cfg_coef
    )

    with torch.no_grad():
        wav = model.generate([prompt])

    # Save to temp file
    path = "temp_output.wav"
    torchaudio.save(path, wav[0].cpu(), sample_rate=32000)
    return path

demo = gr.Interface(
    fn=generate_music,
    inputs=[
        gr.Textbox(label="Music Description", placeholder="upbeat electronic dance music"),
        gr.Slider(1, 30, value=8, label="Duration (seconds)"),
        gr.Slider(0.5, 2.0, value=1.0, label="Temperature"),
        gr.Slider(1.0, 10.0, value=3.0, label="CFG Coefficient")
    ],
    outputs=gr.Audio(label="Generated Music"),
    title="MusicGen Demo"
)

demo.launch()
```

## Performance optimization

### Memory optimization

```python
# Use smaller model
model = MusicGen.get_pretrained('facebook/musicgen-small')

# Clear cache between generations
torch.cuda.empty_cache()

# Generate shorter durations
model.set_generation_params(duration=10)  # Instead of 30

# Use half precision
model = model.half()
```

### Batch processing efficiency

```python
# Process multiple prompts at once (more efficient)
descriptions = ["prompt1", "prompt2", "prompt3", "prompt4"]
wav = model.generate(descriptions)  # Single batch

# Instead of
for desc in descriptions:
    wav = model.generate([desc])  # Multiple batches (slower)
```

### GPU memory requirements

| Model | FP32 VRAM | FP16 VRAM |
|-------|-----------|-----------|
| musicgen-small | ~4GB | ~2GB |
| musicgen-medium | ~8GB | ~4GB |
| musicgen-large | ~16GB | ~8GB |

## Common issues

| Issue | Solution |
|-------|----------|
| CUDA OOM | Use smaller model, reduce duration |
| Poor quality | Increase cfg_coef, better prompts |
| Generation too short | Check max duration setting |
| Audio artifacts | Try different temperature |
| Stereo not working | Use stereo model variant |

## References

- **[Advanced Usage](references/advanced-usage.md)** - Training, fine-tuning, deployment
- **[Troubleshooting](references/troubleshooting.md)** - Common issues and solutions

## Resources

- **GitHub**: https://github.com/facebookresearch/audiocraft
- **Paper (MusicGen)**: https://arxiv.org/abs/2306.05284
- **Paper (AudioGen)**: https://arxiv.org/abs/2209.15352
- **HuggingFace**: https://huggingface.co/facebook/musicgen-small
- **Demo**: https://huggingface.co/spaces/facebook/MusicGen

```

---

## evaluating-llms-harness

- **Name:** `evaluating-llms-harness`
- **Version:** 1.0.0
- **Author:** Orchestra Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\evaluation\lm-evaluation-harness/SKILL.md`

**Description:**

> lm-eval-harness: benchmark LLMs (MMLU, GSM8K, etc.).

**Full SKILL.md body:**

```markdown
---
name: evaluating-llms-harness
description: "lm-eval-harness: benchmark LLMs (MMLU, GSM8K, etc.)."
version: 1.0.0
author: Orchestra Research
license: MIT
dependencies: [lm-eval, transformers, vllm]
platforms: [linux, macos]
metadata:
  hermes:
    tags: [Evaluation, LM Evaluation Harness, Benchmarking, MMLU, HumanEval, GSM8K, EleutherAI, Model Quality, Academic Benchmarks, Industry Standard]

---

# lm-evaluation-harness - LLM Benchmarking

## What's inside

Evaluates LLMs across 60+ academic benchmarks (MMLU, HumanEval, GSM8K, TruthfulQA, HellaSwag). Use when benchmarking model quality, comparing models, reporting academic results, or tracking training progress. Industry standard used by EleutherAI, HuggingFace, and major labs. Supports HuggingFace, vLLM, APIs.

## Quick start

lm-evaluation-harness evaluates LLMs across 60+ academic benchmarks using standardized prompts and metrics.

**Installation**:
```bash
pip install lm-eval
```

**Evaluate any HuggingFace model**:
```bash
lm_eval --model hf \
  --model_args pretrained=meta-llama/Llama-2-7b-hf \
  --tasks mmlu,gsm8k,hellaswag \
  --device cuda:0 \
  --batch_size 8
```

**View available tasks**:
```bash
lm_eval --tasks list
```

## Common workflows

### Workflow 1: Standard benchmark evaluation

Evaluate model on core benchmarks (MMLU, GSM8K, HumanEval).

Copy this checklist:

```
Benchmark Evaluation:
- [ ] Step 1: Choose benchmark suite
- [ ] Step 2: Configure model
- [ ] Step 3: Run evaluation
- [ ] Step 4: Analyze results
```

**Step 1: Choose benchmark suite**

**Core reasoning benchmarks**:
- **MMLU** (Massive Multitask Language Understanding) - 57 subjects, multiple choice
- **GSM8K** - Grade school math word problems
- **HellaSwag** - Common sense reasoning
- **TruthfulQA** - Truthfulness and factuality
- **ARC** (AI2 Reasoning Challenge) - Science questions

**Code benchmarks**:
- **HumanEval** - Python code generation (164 problems)
- **MBPP** (Mostly Basic Python Problems) - Python coding

**Standard suite** (recommended for model releases):
```bash
--tasks mmlu,gsm8k,hellaswag,truthfulqa,arc_challenge
```

**Step 2: Configure model**

**HuggingFace model**:
```bash
lm_eval --model hf \
  --model_args pretrained=meta-llama/Llama-2-7b-hf,dtype=bfloat16 \
  --tasks mmlu \
  --device cuda:0 \
  --batch_size auto  # Auto-detect optimal batch size
```

**Quantized model (4-bit/8-bit)**:
```bash
lm_eval --model hf \
  --model_args pretrained=meta-llama/Llama-2-7b-hf,load_in_4bit=True \
  --tasks mmlu \
  --device cuda:0
```

**Custom checkpoint**:
```bash
lm_eval --model hf \
  --model_args pretrained=/path/to/my-model,tokenizer=/path/to/tokenizer \
  --tasks mmlu \
  --device cuda:0
```

**Step 3: Run evaluation**

```bash
# Full MMLU evaluation (57 subjects)
lm_eval --model hf \
  --model_args pretrained=meta-llama/Llama-2-7b-hf \
  --tasks mmlu \
  --num_fewshot 5 \  # 5-shot evaluation (standard)
  --batch_size 8 \
  --output_path results/ \
  --log_samples  # Save individual predictions

# Multiple benchmarks at once
lm_eval --model hf \
  --model_args pretrained=meta-llama/Llama-2-7b-hf \
  --tasks mmlu,gsm8k,hellaswag,truthfulqa,arc_challenge \
  --num_fewshot 5 \
  --batch_size 8 \
  --output_path results/llama2-7b-eval.json
```

**Step 4: Analyze results**

Results saved to `results/llama2-7b-eval.json`:

```json
{
  "results": {
    "mmlu": {
      "acc": 0.459,
      "acc_stderr": 0.004
    },
    "gsm8k": {
      "exact_match": 0.142,
      "exact_match_stderr": 0.006
    },
    "hellaswag": {
      "acc_norm": 0.765,
      "acc_norm_stderr": 0.004
    }
  },
  "config": {
    "model": "hf",
    "model_args": "pretrained=meta-llama/Llama-2-7b-hf",
    "num_fewshot": 5
  }
}
```

### Workflow 2: Track training progress

Evaluate checkpoints during training.

```
Training Progress Tracking:
- [ ] Step 1: Set up periodic evaluation
- [ ] Step 2: Choose quick benchmarks
- [ ] Step 3: Automate evaluation
- [ ] Step 4: Plot learning curves
```

**Step 1: Set up periodic evaluation**

Evaluate every N training steps:

```bash
#!/bin/bash
# eval_checkpoint.sh

CHECKPOINT_DIR=$1
STEP=$2

lm_eval --model hf \
  --model_args pretrained=$CHECKPOINT_DIR/checkpoint-$STEP \
  --tasks gsm8k,hellaswag \
  --num_fewshot 0 \  # 0-shot for speed
  --batch_size 16 \
  --output_path results/step-$STEP.json
```

**Step 2: Choose quick benchmarks**

Fast benchmarks for frequent evaluation:
- **HellaSwag**: ~10 minutes on 1 GPU
- **GSM8K**: ~5 minutes
- **PIQA**: ~2 minutes

Avoid for frequent eval (too slow):
- **MMLU**: ~2 hours (57 subjects)
- **HumanEval**: Requires code execution

**Step 3: Automate evaluation**

Integrate with training script:

```python
# In training loop
if step % eval_interval == 0:
    model.save_pretrained(f"checkpoints/step-{step}")

    # Run evaluation
    os.system(f"./eval_checkpoint.sh checkpoints step-{step}")
```

Or use PyTorch Lightning callbacks:

```python
from pytorch_lightning import Callback

class EvalHarnessCallback(Callback):
    def on_validation_epoch_end(self, trainer, pl_module):
        step = trainer.global_step
        checkpoint_path = f"checkpoints/step-{step}"

        # Save checkpoint
        trainer.save_checkpoint(checkpoint_path)

        # Run lm-eval
        os.system(f"lm_eval --model hf --model_args pretrained={checkpoint_path} ...")
```

**Step 4: Plot learning curves**

```python
import json
import matplotlib.pyplot as plt

# Load all results
steps = []
mmlu_scores = []

for file in sorted(glob.glob("results/step-*.json")):
    with open(file) as f:
        data = json.load(f)
        step = int(file.split("-")[1].split(".")[0])
        steps.append(step)
        mmlu_scores.append(data["results"]["mmlu"]["acc"])

# Plot
plt.plot(steps, mmlu_scores)
plt.xlabel("Training Step")
plt.ylabel("MMLU Accuracy")
plt.title("Training Progress")
plt.savefig("training_curve.png")
```

### Workflow 3: Compare multiple models

Benchmark suite for model comparison.

```
Model Comparison:
- [ ] Step 1: Define model list
- [ ] Step 2: Run evaluations
- [ ] Step 3: Generate comparison table
```

**Step 1: Define model list**

```bash
# models.txt
meta-llama/Llama-2-7b-hf
meta-llama/Llama-2-13b-hf
mistralai/Mistral-7B-v0.1
microsoft/phi-2
```

**Step 2: Run evaluations**

```bash
#!/bin/bash
# eval_all_models.sh

TASKS="mmlu,gsm8k,hellaswag,truthfulqa"

while read model; do
    echo "Evaluating $model"

    # Extract model name for output file
    model_name=$(echo $model | sed 's/\//-/g')

    lm_eval --model hf \
      --model_args pretrained=$model,dtype=bfloat16 \
      --tasks $TASKS \
      --num_fewshot 5 \
      --batch_size auto \
      --output_path results/$model_name.json

done < models.txt
```

**Step 3: Generate comparison table**

```python
import json
import pandas as pd

models = [
    "meta-llama-Llama-2-7b-hf",
    "meta-llama-Llama-2-13b-hf",
    "mistralai-Mistral-7B-v0.1",
    "microsoft-phi-2"
]

tasks = ["mmlu", "gsm8k", "hellaswag", "truthfulqa"]

results = []
for model in models:
    with open(f"results/{model}.json") as f:
        data = json.load(f)
        row = {"Model": model.replace("-", "/")}
        for task in tasks:
            # Get primary metric for each task
            metrics = data["results"][task]
            if "acc" in metrics:
                row[task.upper()] = f"{metrics['acc']:.3f}"
            elif "exact_match" in metrics:
                row[task.upper()] = f"{metrics['exact_match']:.3f}"
        results.append(row)

df = pd.DataFrame(results)
print(df.to_markdown(index=False))
```

Output:
```
| Model                  | MMLU  | GSM8K | HELLASWAG | TRUTHFULQA |
|------------------------|-------|-------|-----------|------------|
| meta-llama/Llama-2-7b  | 0.459 | 0.142 | 0.765     | 0.391      |
| meta-llama/Llama-2-13b | 0.549 | 0.287 | 0.801     | 0.430      |
| mistralai/Mistral-7B   | 0.626 | 0.395 | 0.812     | 0.428      |
| microsoft/phi-2        | 0.560 | 0.613 | 0.682     | 0.447      |
```

### Workflow 4: Evaluate with vLLM (faster inference)

Use vLLM backend for 5-10x faster evaluation.

```
vLLM Evaluation:
- [ ] Step 1: Install vLLM
- [ ] Step 2: Configure vLLM backend
- [ ] Step 3: Run evaluation
```

**Step 1: Install vLLM**

```bash
pip install vllm
```

**Step 2: Configure vLLM backend**

```bash
lm_eval --model vllm \
  --model_args pretrained=meta-llama/Llama-2-7b-hf,tensor_parallel_size=1,dtype=auto,gpu_memory_utilization=0.8 \
  --tasks mmlu \
  --batch_size auto
```

**Step 3: Run evaluation**

vLLM is 5-10× faster than standard HuggingFace:

```bash
# Standard HF: ~2 hours for MMLU on 7B model
lm_eval --model hf \
  --model_args pretrained=meta-llama/Llama-2-7b-hf \
  --tasks mmlu \
  --batch_size 8

# vLLM: ~15-20 minutes for MMLU on 7B model
lm_eval --model vllm \
  --model_args pretrained=meta-llama/Llama-2-7b-hf,tensor_parallel_size=2 \
  --tasks mmlu \
  --batch_size auto
```

## When to use vs alternatives

**Use lm-evaluation-harness when:**
- Benchmarking models for academic papers
- Comparing model quality across standard tasks
- Tracking training progress
- Reporting standardized metrics (everyone uses same prompts)
- Need reproducible evaluation

**Use alternatives instead:**
- **HELM** (Stanford): Broader evaluation (fairness, efficiency, calibration)
- **AlpacaEval**: Instruction-following evaluation with LLM judges
- **MT-Bench**: Conversational multi-turn evaluation
- **Custom scripts**: Domain-specific evaluation

## Common issues

**Issue: Evaluation too slow**

Use vLLM backend:
```bash
lm_eval --model vllm \
  --model_args pretrained=model-name,tensor_parallel_size=2
```

Or reduce fewshot examples:
```bash
--num_fewshot 0  # Instead of 5
```

Or evaluate subset of MMLU:
```bash
--tasks mmlu_stem  # Only STEM subjects
```

**Issue: Out of memory**

Reduce batch size:
```bash
--batch_size 1  # Or --batch_size auto
```

Use quantization:
```bash
--model_args pretrained=model-name,load_in_8bit=True
```

Enable CPU offloading:
```bash
--model_args pretrained=model-name,device_map=auto,offload_folder=offload
```

**Issue: Different results than reported**

Check fewshot count:
```bash
--num_fewshot 5  # Most papers use 5-shot
```

Check exact task name:
```bash
--tasks mmlu  # Not mmlu_direct or mmlu_fewshot
```

Verify model and tokenizer match:
```bash
--model_args pretrained=model-name,tokenizer=same-model-name
```

**Issue: HumanEval not executing code**

Install execution dependencies:
```bash
pip install human-eval
```

Enable code execution:
```bash
lm_eval --model hf \
  --model_args pretrained=model-name \
  --tasks humaneval \
  --allow_code_execution  # Required for HumanEval
```

## Advanced topics

**Benchmark descriptions**: See [references/benchmark-guide.md](references/benchmark-guide.md) for detailed description of all 60+ tasks, what they measure, and interpretation.

**Custom tasks**: See [references/custom-tasks.md](references/custom-tasks.md) for creating domain-specific evaluation tasks.

**API evaluation**: See [references/api-evaluation.md](references/api-evaluation.md) for evaluating OpenAI, Anthropic, and other API models.

**Multi-GPU strategies**: See [references/distributed-eval.md](references/distributed-eval.md) for data parallel and tensor parallel evaluation.

## Hardware requirements

- **GPU**: NVIDIA (CUDA 11.8+), works on CPU (very slow)
- **VRAM**:
  - 7B model: 16GB (bf16) or 8GB (8-bit)
  - 13B model: 28GB (bf16) or 14GB (8-bit)
  - 70B model: Requires multi-GPU or quantization
- **Time** (7B model, single A100):
  - HellaSwag: 10 minutes
  - GSM8K: 5 minutes
  - MMLU (full): 2 hours
  - HumanEval: 20 minutes

## Resources

- GitHub: https://github.com/EleutherAI/lm-evaluation-harness
- Docs: https://github.com/EleutherAI/lm-evaluation-harness/tree/main/docs
- Task library: 60+ tasks including MMLU, GSM8K, HumanEval, TruthfulQA, HellaSwag, ARC, WinoGrande, etc.
- Leaderboard: https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard (uses this harness)




```

---

## huggingface-hub

- **Name:** `huggingface-hub`
- **Version:** 1.0.0
- **Author:** Hugging Face
- **License:** MIT
- **Tags:** [huggingface, hf, models, datasets, hub, mlops]
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\huggingface-hub/SKILL.md`

**Description:**

> HuggingFace hf CLI: search/download/upload models, datasets.

**Full SKILL.md body:**

```markdown
---
name: huggingface-hub
description: "HuggingFace hf CLI: search/download/upload models, datasets."
version: 1.0.0
author: Hugging Face
license: MIT
tags: [huggingface, hf, models, datasets, hub, mlops]
platforms: [linux, macos, windows]
---

# Hugging Face CLI (`hf`) Reference Guide

The `hf` command is the modern command-line interface for interacting with the Hugging Face Hub, providing tools to manage repositories, models, datasets, and Spaces.

> **IMPORTANT:** The `hf` command replaces the now deprecated `huggingface-cli` command.

## Quick Start
*   **Installation:** `curl -LsSf https://hf.co/cli/install.sh | bash -s`
*   **Help:** Use `hf --help` to view all available functions and real-world examples.
*   **Authentication:** Recommended via `HF_TOKEN` environment variable or the `--token` flag.

---

## Core Commands

### General Operations
*   `hf download REPO_ID`: Download files from the Hub.
*   `hf upload REPO_ID`: Upload files/folders (recommended for single-commit).
*   `hf upload-large-folder REPO_ID LOCAL_PATH`: Recommended for resumable uploads of large directories.
*   `hf sync`: Sync files between a local directory and a bucket.
*   `hf env` / `hf version`: View environment and version details.

### Authentication (`hf auth`)
*   `login` / `logout`: Manage sessions using tokens from [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).
*   `list` / `switch`: Manage and toggle between multiple stored access tokens.
*   `whoami`: Identify the currently logged-in account.

### Repository Management (`hf repos`)
*   `create` / `delete`: Create or permanently remove repositories.
*   `duplicate`: Clone a model, dataset, or Space to a new ID.
*   `move`: Transfer a repository between namespaces.
*   `branch` / `tag`: Manage Git-like references.
*   `delete-files`: Remove specific files using patterns.

---

## Specialized Hub Interactions

### Datasets & Models
*   **Datasets:** `hf datasets list`, `info`, and `parquet` (list parquet URLs).
*   **SQL Queries:** `hf datasets sql SQL` — Execute raw SQL via DuckDB against dataset parquet URLs.
*   **Models:** `hf models list` and `info`.
*   **Papers:** `hf papers list` — View daily papers.

### Discussions & Pull Requests (`hf discussions`)
*   Manage the lifecycle of Hub contributions: `list`, `create`, `info`, `comment`, `close`, `reopen`, and `rename`.
*   `diff`: View changes in a PR.
*   `merge`: Finalize pull requests.

### Infrastructure & Compute
*   **Endpoints:** Deploy and manage Inference Endpoints (`deploy`, `pause`, `resume`, `scale-to-zero`, `catalog`).
*   **Jobs:** Run compute tasks on HF infrastructure. Includes `hf jobs uv` for running Python scripts with inline dependencies and `stats` for resource monitoring.
*   **Spaces:** Manage interactive apps. Includes `dev-mode` and `hot-reload` for Python files without full restarts.

### Storage & Automation
*   **Buckets:** Full S3-like bucket management (`create`, `cp`, `mv`, `rm`, `sync`).
*   **Cache:** Manage local storage with `list`, `prune` (remove detached revisions), and `verify` (checksum checks).
*   **Webhooks:** Automate workflows by managing Hub webhooks (`create`, `watch`, `enable`/`disable`).
*   **Collections:** Organize Hub items into collections (`add-item`, `update`, `list`).

---

## Advanced Usage & Tips

### Global Flags
*   `--format json`: Produces machine-readable output for automation.
*   `-q` / `--quiet`: Limits output to IDs only.

### Extensions & Skills
*   **Extensions:** Extend CLI functionality via GitHub repositories using `hf extensions install REPO_ID`.
*   **Skills:** Manage AI assistant skills with `hf skills add`.

```

---

## llama-cpp

- **Name:** `llama-cpp`
- **Version:** 2.1.2
- **Author:** Orchestra Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\inference\llama-cpp/SKILL.md`

**Description:**

> llama.cpp local GGUF inference + HF Hub model discovery.

**Full SKILL.md body:**

```markdown
---
name: llama-cpp
description: llama.cpp local GGUF inference + HF Hub model discovery.
version: 2.1.2
author: Orchestra Research
license: MIT
dependencies: [llama-cpp-python>=0.2.0]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [llama.cpp, GGUF, Quantization, Hugging Face Hub, CPU Inference, Apple Silicon, Edge Deployment, AMD GPUs, Intel GPUs, NVIDIA, URL-first]
---

# llama.cpp + GGUF

Use this skill for local GGUF inference, quant selection, or Hugging Face repo discovery for llama.cpp.

## When to use

- Run local models on CPU, Apple Silicon, CUDA, ROCm, or Intel GPUs
- Find the right GGUF for a specific Hugging Face repo
- Build a `llama-server` or `llama-cli` command from the Hub
- Search the Hub for models that already support llama.cpp
- Enumerate available `.gguf` files and sizes for a repo
- Decide between Q4/Q5/Q6/IQ variants for the user's RAM or VRAM

## Model Discovery workflow

Prefer URL workflows before asking for `hf`, Python, or custom scripts.

1. Search for candidate repos on the Hub:
   - Base: `https://huggingface.co/models?apps=llama.cpp&sort=trending`
   - Add `search=<term>` for a model family
   - Add `num_parameters=min:0,max:24B` or similar when the user has size constraints
2. Open the repo with the llama.cpp local-app view:
   - `https://huggingface.co/<repo>?local-app=llama.cpp`
3. Treat the local-app snippet as the source of truth when it is visible:
   - copy the exact `llama-server` or `llama-cli` command
   - report the recommended quant exactly as HF shows it
4. Read the same `?local-app=llama.cpp` URL as page text or HTML and extract the section under `Hardware compatibility`:
   - prefer its exact quant labels and sizes over generic tables
   - keep repo-specific labels such as `UD-Q4_K_M` or `IQ4_NL_XL`
   - if that section is not visible in the fetched page source, say so and fall back to the tree API plus generic quant guidance
5. Query the tree API to confirm what actually exists:
   - `https://huggingface.co/api/models/<repo>/tree/main?recursive=true`
   - keep entries where `type` is `file` and `path` ends with `.gguf`
   - use `path` and `size` as the source of truth for filenames and byte sizes
   - separate quantized checkpoints from `mmproj-*.gguf` projector files and `BF16/` shard files
   - use `https://huggingface.co/<repo>/tree/main` only as a human fallback
6. If the local-app snippet is not text-visible, reconstruct the command from the repo plus the chosen quant:
   - shorthand quant selection: `llama-server -hf <repo>:<QUANT>`
   - exact-file fallback: `llama-server --hf-repo <repo> --hf-file <filename.gguf>`
7. Only suggest conversion from Transformers weights if the repo does not already expose GGUF files.

## Quick start

### Install llama.cpp

```bash
# macOS / Linux (simplest)
brew install llama.cpp
```

```bash
winget install llama.cpp
```

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

### Run directly from the Hugging Face Hub

```bash
llama-cli -hf bartowski/Llama-3.2-3B-Instruct-GGUF:Q8_0
```

```bash
llama-server -hf bartowski/Llama-3.2-3B-Instruct-GGUF:Q8_0
```

### Run an exact GGUF file from the Hub

Use this when the tree API shows custom file naming or the exact HF snippet is missing.

```bash
llama-server \
    --hf-repo microsoft/Phi-3-mini-4k-instruct-gguf \
    --hf-file Phi-3-mini-4k-instruct-q4.gguf \
    -c 4096
```

### OpenAI-compatible server check

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "Write a limerick about Python exceptions"}
    ]
  }'
```

## Python bindings (llama-cpp-python)

`pip install llama-cpp-python` (CUDA: `CMAKE_ARGS="-DGGML_CUDA=on" pip install llama-cpp-python --force-reinstall --no-cache-dir`; Metal: `CMAKE_ARGS="-DGGML_METAL=on" ...`).

### Basic generation

```python
from llama_cpp import Llama

llm = Llama(
    model_path="./model-q4_k_m.gguf",
    n_ctx=4096,
    n_gpu_layers=35,     # 0 for CPU, 99 to offload everything
    n_threads=8,
)

out = llm("What is machine learning?", max_tokens=256, temperature=0.7)
print(out["choices"][0]["text"])
```

### Chat + streaming

```python
llm = Llama(
    model_path="./model-q4_k_m.gguf",
    n_ctx=4096,
    n_gpu_layers=35,
    chat_format="llama-3",   # or "chatml", "mistral", etc.
)

resp = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Python?"},
    ],
    max_tokens=256,
)
print(resp["choices"][0]["message"]["content"])

# Streaming
for chunk in llm("Explain quantum computing:", max_tokens=256, stream=True):
    print(chunk["choices"][0]["text"], end="", flush=True)
```

### Embeddings

```python
llm = Llama(model_path="./model-q4_k_m.gguf", embedding=True, n_gpu_layers=35)
vec = llm.embed("This is a test sentence.")
print(f"Embedding dimension: {len(vec)}")
```

You can also load a GGUF straight from the Hub:

```python
llm = Llama.from_pretrained(
    repo_id="bartowski/Llama-3.2-3B-Instruct-GGUF",
    filename="*Q4_K_M.gguf",
    n_gpu_layers=35,
)
```

## Choosing a quant

Use the Hub page first, generic heuristics second.

- Prefer the exact quant that HF marks as compatible for the user's hardware profile.
- For general chat, start with `Q4_K_M`.
- For code or technical work, prefer `Q5_K_M` or `Q6_K` if memory allows.
- For very tight RAM budgets, consider `Q3_K_M`, `IQ` variants, or `Q2` variants only if the user explicitly prioritizes fit over quality.
- For multimodal repos, mention `mmproj-*.gguf` separately. The projector is not the main model file.
- Do not normalize repo-native labels. If the page says `UD-Q4_K_M`, report `UD-Q4_K_M`.

## Extracting available GGUFs from a repo

When the user asks what GGUFs exist, return:

- filename
- file size
- quant label
- whether it is a main model or an auxiliary projector

Ignore unless requested:

- README
- BF16 shard files
- imatrix blobs or calibration artifacts

Use the tree API for this step:

- `https://huggingface.co/api/models/<repo>/tree/main?recursive=true`

For a repo like `unsloth/Qwen3.6-35B-A3B-GGUF`, the local-app page can show quant chips such as `UD-Q4_K_M`, `UD-Q5_K_M`, `UD-Q6_K`, and `Q8_0`, while the tree API exposes exact file paths such as `Qwen3.6-35B-A3B-UD-Q4_K_M.gguf` and `Qwen3.6-35B-A3B-Q8_0.gguf` with byte sizes. Use the tree API to turn a quant label into an exact filename.

## Search patterns

Use these URL shapes directly:

```text
https://huggingface.co/models?apps=llama.cpp&sort=trending
https://huggingface.co/models?search=<term>&apps=llama.cpp&sort=trending
https://huggingface.co/models?search=<term>&apps=llama.cpp&num_parameters=min:0,max:24B&sort=trending
https://huggingface.co/<repo>?local-app=llama.cpp
https://huggingface.co/api/models/<repo>/tree/main?recursive=true
https://huggingface.co/<repo>/tree/main
```

## Output format

When answering discovery requests, prefer a compact structured result like:

```text
Repo: <repo>
Recommended quant from HF: <label> (<size>)
llama-server: <command>
Other GGUFs:
- <filename> - <size>
- <filename> - <size>
Source URLs:
- <local-app URL>
- <tree API URL>
```

## References

- **[hub-discovery.md](references/hub-discovery.md)** - URL-only Hugging Face workflows, search patterns, GGUF extraction, and command reconstruction
- **[advanced-usage.md](references/advanced-usage.md)** — speculative decoding, batched inference, grammar-constrained generation, LoRA, multi-GPU, custom builds, benchmark scripts
- **[quantization.md](references/quantization.md)** — quant quality tradeoffs, when to use Q4/Q5/Q6/IQ, model size scaling, imatrix
- **[server.md](references/server.md)** — direct-from-Hub server launch, OpenAI API endpoints, Docker deployment, NGINX load balancing, monitoring
- **[optimization.md](references/optimization.md)** — CPU threading, BLAS, GPU offload heuristics, batch tuning, benchmarks
- **[troubleshooting.md](references/troubleshooting.md)** — install/convert/quantize/inference/server issues, Apple Silicon, debugging

## Resources

- **GitHub**: https://github.com/ggml-org/llama.cpp
- **Hugging Face GGUF + llama.cpp docs**: https://huggingface.co/docs/hub/gguf-llamacpp
- **Hugging Face Local Apps docs**: https://huggingface.co/docs/hub/main/local-apps
- **Hugging Face Local Agents docs**: https://huggingface.co/docs/hub/agents-local
- **Example local-app page**: https://huggingface.co/unsloth/Qwen3.6-35B-A3B-GGUF?local-app=llama.cpp
- **Example tree API**: https://huggingface.co/api/models/unsloth/Qwen3.6-35B-A3B-GGUF/tree/main?recursive=true
- **Example llama.cpp search**: https://huggingface.co/models?num_parameters=min:0,max:24B&apps=llama.cpp&sort=trending
- **License**: MIT

```

---

## segment-anything-model

- **Name:** `segment-anything-model`
- **Version:** 1.0.0
- **Author:** Orchestra Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\models\segment-anything/SKILL.md`

**Description:**

> SAM: zero-shot image segmentation via points, boxes, masks.

**Full SKILL.md body:**

```markdown
---
name: segment-anything-model
description: "SAM: zero-shot image segmentation via points, boxes, masks."
version: 1.0.0
author: Orchestra Research
license: MIT
dependencies: [segment-anything, transformers>=4.30.0, torch>=1.7.0]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Multimodal, Image Segmentation, Computer Vision, SAM, Zero-Shot]

---

# Segment Anything Model (SAM)

Comprehensive guide to using Meta AI's Segment Anything Model for zero-shot image segmentation.

## When to use SAM

**Use SAM when:**
- Need to segment any object in images without task-specific training
- Building interactive annotation tools with point/box prompts
- Generating training data for other vision models
- Need zero-shot transfer to new image domains
- Building object detection/segmentation pipelines
- Processing medical, satellite, or domain-specific images

**Key features:**
- **Zero-shot segmentation**: Works on any image domain without fine-tuning
- **Flexible prompts**: Points, bounding boxes, or previous masks
- **Automatic segmentation**: Generate all object masks automatically
- **High quality**: Trained on 1.1 billion masks from 11 million images
- **Multiple model sizes**: ViT-B (fastest), ViT-L, ViT-H (most accurate)
- **ONNX export**: Deploy in browsers and edge devices

**Use alternatives instead:**
- **YOLO/Detectron2**: For real-time object detection with classes
- **Mask2Former**: For semantic/panoptic segmentation with categories
- **GroundingDINO + SAM**: For text-prompted segmentation
- **SAM 2**: For video segmentation tasks

## Quick start

### Installation

```bash
# From GitHub
pip install git+https://github.com/facebookresearch/segment-anything.git

# Optional dependencies
pip install opencv-python pycocotools matplotlib

# Or use HuggingFace transformers
pip install transformers
```

### Download checkpoints

```bash
# ViT-H (largest, most accurate) - 2.4GB
wget https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth

# ViT-L (medium) - 1.2GB
wget https://dl.fbaipublicfiles.com/segment_anything/sam_vit_l_0b3195.pth

# ViT-B (smallest, fastest) - 375MB
wget https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
```

### Basic usage with SamPredictor

```python
import numpy as np
from segment_anything import sam_model_registry, SamPredictor

# Load model
sam = sam_model_registry["vit_h"](checkpoint="sam_vit_h_4b8939.pth")
sam.to(device="cuda")

# Create predictor
predictor = SamPredictor(sam)

# Set image (computes embeddings once)
image = cv2.imread("image.jpg")
image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
predictor.set_image(image)

# Predict with point prompts
input_point = np.array([[500, 375]])  # (x, y) coordinates
input_label = np.array([1])  # 1 = foreground, 0 = background

masks, scores, logits = predictor.predict(
    point_coords=input_point,
    point_labels=input_label,
    multimask_output=True  # Returns 3 mask options
)

# Select best mask
best_mask = masks[np.argmax(scores)]
```

### HuggingFace Transformers

```python
import torch
from PIL import Image
from transformers import SamModel, SamProcessor

# Load model and processor
model = SamModel.from_pretrained("facebook/sam-vit-huge")
processor = SamProcessor.from_pretrained("facebook/sam-vit-huge")
model.to("cuda")

# Process image with point prompt
image = Image.open("image.jpg")
input_points = [[[450, 600]]]  # Batch of points

inputs = processor(image, input_points=input_points, return_tensors="pt")
inputs = {k: v.to("cuda") for k, v in inputs.items()}

# Generate masks
with torch.no_grad():
    outputs = model(**inputs)

# Post-process masks to original size
masks = processor.image_processor.post_process_masks(
    outputs.pred_masks.cpu(),
    inputs["original_sizes"].cpu(),
    inputs["reshaped_input_sizes"].cpu()
)
```

## Core concepts

### Model architecture

<!-- ascii-guard-ignore -->
```
SAM Architecture:
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Image Encoder  │────▶│ Prompt Encoder  │────▶│  Mask Decoder   │
│     (ViT)       │     │ (Points/Boxes)  │     │ (Transformer)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
   Image Embeddings      Prompt Embeddings         Masks + IoU
   (computed once)       (per prompt)             predictions
```
<!-- ascii-guard-ignore-end -->

### Model variants

| Model | Checkpoint | Size | Speed | Accuracy |
|-------|------------|------|-------|----------|
| ViT-H | `vit_h` | 2.4 GB | Slowest | Best |
| ViT-L | `vit_l` | 1.2 GB | Medium | Good |
| ViT-B | `vit_b` | 375 MB | Fastest | Good |

### Prompt types

| Prompt | Description | Use Case |
|--------|-------------|----------|
| Point (foreground) | Click on object | Single object selection |
| Point (background) | Click outside object | Exclude regions |
| Bounding box | Rectangle around object | Larger objects |
| Previous mask | Low-res mask input | Iterative refinement |

## Interactive segmentation

### Point prompts

```python
# Single foreground point
input_point = np.array([[500, 375]])
input_label = np.array([1])

masks, scores, logits = predictor.predict(
    point_coords=input_point,
    point_labels=input_label,
    multimask_output=True
)

# Multiple points (foreground + background)
input_points = np.array([[500, 375], [600, 400], [450, 300]])
input_labels = np.array([1, 1, 0])  # 2 foreground, 1 background

masks, scores, logits = predictor.predict(
    point_coords=input_points,
    point_labels=input_labels,
    multimask_output=False  # Single mask when prompts are clear
)
```

### Box prompts

```python
# Bounding box [x1, y1, x2, y2]
input_box = np.array([425, 600, 700, 875])

masks, scores, logits = predictor.predict(
    box=input_box,
    multimask_output=False
)
```

### Combined prompts

```python
# Box + points for precise control
masks, scores, logits = predictor.predict(
    point_coords=np.array([[500, 375]]),
    point_labels=np.array([1]),
    box=np.array([400, 300, 700, 600]),
    multimask_output=False
)
```

### Iterative refinement

```python
# Initial prediction
masks, scores, logits = predictor.predict(
    point_coords=np.array([[500, 375]]),
    point_labels=np.array([1]),
    multimask_output=True
)

# Refine with additional point using previous mask
masks, scores, logits = predictor.predict(
    point_coords=np.array([[500, 375], [550, 400]]),
    point_labels=np.array([1, 0]),  # Add background point
    mask_input=logits[np.argmax(scores)][None, :, :],  # Use best mask
    multimask_output=False
)
```

## Automatic mask generation

### Basic automatic segmentation

```python
from segment_anything import SamAutomaticMaskGenerator

# Create generator
mask_generator = SamAutomaticMaskGenerator(sam)

# Generate all masks
masks = mask_generator.generate(image)

# Each mask contains:
# - segmentation: binary mask
# - bbox: [x, y, w, h]
# - area: pixel count
# - predicted_iou: quality score
# - stability_score: robustness score
# - point_coords: generating point
```

### Customized generation

```python
mask_generator = SamAutomaticMaskGenerator(
    model=sam,
    points_per_side=32,          # Grid density (more = more masks)
    pred_iou_thresh=0.88,        # Quality threshold
    stability_score_thresh=0.95,  # Stability threshold
    crop_n_layers=1,             # Multi-scale crops
    crop_n_points_downscale_factor=2,
    min_mask_region_area=100,    # Remove tiny masks
)

masks = mask_generator.generate(image)
```

### Filtering masks

```python
# Sort by area (largest first)
masks = sorted(masks, key=lambda x: x['area'], reverse=True)

# Filter by predicted IoU
high_quality = [m for m in masks if m['predicted_iou'] > 0.9]

# Filter by stability score
stable_masks = [m for m in masks if m['stability_score'] > 0.95]
```

## Batched inference

### Multiple images

```python
# Process multiple images efficiently
images = [cv2.imread(f"image_{i}.jpg") for i in range(10)]

all_masks = []
for image in images:
    predictor.set_image(image)
    masks, _, _ = predictor.predict(
        point_coords=np.array([[500, 375]]),
        point_labels=np.array([1]),
        multimask_output=True
    )
    all_masks.append(masks)
```

### Multiple prompts per image

```python
# Process multiple prompts efficiently (one image encoding)
predictor.set_image(image)

# Batch of point prompts
points = [
    np.array([[100, 100]]),
    np.array([[200, 200]]),
    np.array([[300, 300]])
]

all_masks = []
for point in points:
    masks, scores, _ = predictor.predict(
        point_coords=point,
        point_labels=np.array([1]),
        multimask_output=True
    )
    all_masks.append(masks[np.argmax(scores)])
```

## ONNX deployment

### Export model

```bash
python scripts/export_onnx_model.py \
    --checkpoint sam_vit_h_4b8939.pth \
    --model-type vit_h \
    --output sam_onnx.onnx \
    --return-single-mask
```

### Use ONNX model

```python
import onnxruntime

# Load ONNX model
ort_session = onnxruntime.InferenceSession("sam_onnx.onnx")

# Run inference (image embeddings computed separately)
masks = ort_session.run(
    None,
    {
        "image_embeddings": image_embeddings,
        "point_coords": point_coords,
        "point_labels": point_labels,
        "mask_input": np.zeros((1, 1, 256, 256), dtype=np.float32),
        "has_mask_input": np.array([0], dtype=np.float32),
        "orig_im_size": np.array([h, w], dtype=np.float32)
    }
)
```

## Common workflows

### Workflow 1: Annotation tool

```python
import cv2

# Load model
predictor = SamPredictor(sam)
predictor.set_image(image)

def on_click(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        # Foreground point
        masks, scores, _ = predictor.predict(
            point_coords=np.array([[x, y]]),
            point_labels=np.array([1]),
            multimask_output=True
        )
        # Display best mask
        display_mask(masks[np.argmax(scores)])
```

### Workflow 2: Object extraction

```python
def extract_object(image, point):
    """Extract object at point with transparent background."""
    predictor.set_image(image)

    masks, scores, _ = predictor.predict(
        point_coords=np.array([point]),
        point_labels=np.array([1]),
        multimask_output=True
    )

    best_mask = masks[np.argmax(scores)]

    # Create RGBA output
    rgba = np.zeros((image.shape[0], image.shape[1], 4), dtype=np.uint8)
    rgba[:, :, :3] = image
    rgba[:, :, 3] = best_mask * 255

    return rgba
```

### Workflow 3: Medical image segmentation

```python
# Process medical images (grayscale to RGB)
medical_image = cv2.imread("scan.png", cv2.IMREAD_GRAYSCALE)
rgb_image = cv2.cvtColor(medical_image, cv2.COLOR_GRAY2RGB)

predictor.set_image(rgb_image)

# Segment region of interest
masks, scores, _ = predictor.predict(
    box=np.array([x1, y1, x2, y2]),  # ROI bounding box
    multimask_output=True
)
```

## Output format

### Mask data structure

```python
# SamAutomaticMaskGenerator output
{
    "segmentation": np.ndarray,  # H×W binary mask
    "bbox": [x, y, w, h],        # Bounding box
    "area": int,                 # Pixel count
    "predicted_iou": float,      # 0-1 quality score
    "stability_score": float,    # 0-1 robustness score
    "crop_box": [x, y, w, h],    # Generation crop region
    "point_coords": [[x, y]],    # Input point
}
```

### COCO RLE format

```python
from pycocotools import mask as mask_utils

# Encode mask to RLE
rle = mask_utils.encode(np.asfortranarray(mask.astype(np.uint8)))
rle["counts"] = rle["counts"].decode("utf-8")

# Decode RLE to mask
decoded_mask = mask_utils.decode(rle)
```

## Performance optimization

### GPU memory

```python
# Use smaller model for limited VRAM
sam = sam_model_registry["vit_b"](checkpoint="sam_vit_b_01ec64.pth")

# Process images in batches
# Clear CUDA cache between large batches
torch.cuda.empty_cache()
```

### Speed optimization

```python
# Use half precision
sam = sam.half()

# Reduce points for automatic generation
mask_generator = SamAutomaticMaskGenerator(
    model=sam,
    points_per_side=16,  # Default is 32
)

# Use ONNX for deployment
# Export with --return-single-mask for faster inference
```

## Common issues

| Issue | Solution |
|-------|----------|
| Out of memory | Use ViT-B model, reduce image size |
| Slow inference | Use ViT-B, reduce points_per_side |
| Poor mask quality | Try different prompts, use box + points |
| Edge artifacts | Use stability_score filtering |
| Small objects missed | Increase points_per_side |

## References

- **[Advanced Usage](references/advanced-usage.md)** - Batching, fine-tuning, integration
- **[Troubleshooting](references/troubleshooting.md)** - Common issues and solutions

## Resources

- **GitHub**: https://github.com/facebookresearch/segment-anything
- **Paper**: https://arxiv.org/abs/2304.02643
- **Demo**: https://segment-anything.com
- **SAM 2 (Video)**: https://github.com/facebookresearch/segment-anything-2
- **HuggingFace**: https://huggingface.co/facebook/sam-vit-huge

```

---

## serving-llms-vllm

- **Name:** `serving-llms-vllm`
- **Version:** 1.0.0
- **Author:** Orchestra Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\inference\vllm/SKILL.md`

**Description:**

> vLLM: high-throughput LLM serving, OpenAI API, quantization.

**Full SKILL.md body:**

```markdown
---
name: serving-llms-vllm
description: "vLLM: high-throughput LLM serving, OpenAI API, quantization."
version: 1.0.0
author: Orchestra Research
license: MIT
dependencies: [vllm, torch, transformers]
platforms: [linux, macos]
metadata:
  hermes:
    tags: [vLLM, Inference Serving, PagedAttention, Continuous Batching, High Throughput, Production, OpenAI API, Quantization, Tensor Parallelism]

---

# vLLM - High-Performance LLM Serving

## When to use

Use when deploying production LLM APIs, optimizing inference latency/throughput, or serving models with limited GPU memory. Supports OpenAI-compatible endpoints, quantization (GPTQ/AWQ/FP8), and tensor parallelism.

## Quick start

vLLM achieves 24x higher throughput than standard transformers through PagedAttention (block-based KV cache) and continuous batching (mixing prefill/decode requests).

**Installation**:
```bash
pip install vllm
```

**Basic offline inference**:
```python
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-3-8B-Instruct")
sampling = SamplingParams(temperature=0.7, max_tokens=256)

outputs = llm.generate(["Explain quantum computing"], sampling)
print(outputs[0].outputs[0].text)
```

**OpenAI-compatible server**:
```bash
vllm serve meta-llama/Llama-3-8B-Instruct

# Query with OpenAI SDK
python -c "
from openai import OpenAI
client = OpenAI(base_url='http://localhost:8000/v1', api_key='EMPTY')
print(client.chat.completions.create(
    model='meta-llama/Llama-3-8B-Instruct',
    messages=[{'role': 'user', 'content': 'Hello!'}]
).choices[0].message.content)
"
```

## Common workflows

### Workflow 1: Production API deployment

Copy this checklist and track progress:

```
Deployment Progress:
- [ ] Step 1: Configure server settings
- [ ] Step 2: Test with limited traffic
- [ ] Step 3: Enable monitoring
- [ ] Step 4: Deploy to production
- [ ] Step 5: Verify performance metrics
```

**Step 1: Configure server settings**

Choose configuration based on your model size:

```bash
# For 7B-13B models on single GPU
vllm serve meta-llama/Llama-3-8B-Instruct \
  --gpu-memory-utilization 0.9 \
  --max-model-len 8192 \
  --port 8000

# For 30B-70B models with tensor parallelism
vllm serve meta-llama/Llama-2-70b-hf \
  --tensor-parallel-size 4 \
  --gpu-memory-utilization 0.9 \
  --quantization awq \
  --port 8000

# For production with caching and metrics
vllm serve meta-llama/Llama-3-8B-Instruct \
  --gpu-memory-utilization 0.9 \
  --enable-prefix-caching \
  --enable-metrics \
  --metrics-port 9090 \
  --port 8000 \
  --host 0.0.0.0
```

**Step 2: Test with limited traffic**

Run load test before production:

```bash
# Install load testing tool
pip install locust

# Create test_load.py with sample requests
# Run: locust -f test_load.py --host http://localhost:8000
```

Verify TTFT (time to first token) < 500ms and throughput > 100 req/sec.

**Step 3: Enable monitoring**

vLLM exposes Prometheus metrics on port 9090:

```bash
curl http://localhost:9090/metrics | grep vllm
```

Key metrics to monitor:
- `vllm:time_to_first_token_seconds` - Latency
- `vllm:num_requests_running` - Active requests
- `vllm:gpu_cache_usage_perc` - KV cache utilization

**Step 4: Deploy to production**

Use Docker for consistent deployment:

```bash
# Run vLLM in Docker
docker run --gpus all -p 8000:8000 \
  vllm/vllm-openai:latest \
  --model meta-llama/Llama-3-8B-Instruct \
  --gpu-memory-utilization 0.9 \
  --enable-prefix-caching
```

**Step 5: Verify performance metrics**

Check that deployment meets targets:
- TTFT < 500ms (for short prompts)
- Throughput > target req/sec
- GPU utilization > 80%
- No OOM errors in logs

### Workflow 2: Offline batch inference

For processing large datasets without server overhead.

Copy this checklist:

```
Batch Processing:
- [ ] Step 1: Prepare input data
- [ ] Step 2: Configure LLM engine
- [ ] Step 3: Run batch inference
- [ ] Step 4: Process results
```

**Step 1: Prepare input data**

```python
# Load prompts from file
prompts = []
with open("prompts.txt") as f:
    prompts = [line.strip() for line in f]

print(f"Loaded {len(prompts)} prompts")
```

**Step 2: Configure LLM engine**

```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="meta-llama/Llama-3-8B-Instruct",
    tensor_parallel_size=2,  # Use 2 GPUs
    gpu_memory_utilization=0.9,
    max_model_len=4096
)

sampling = SamplingParams(
    temperature=0.7,
    top_p=0.95,
    max_tokens=512,
    stop=["</s>", "\n\n"]
)
```

**Step 3: Run batch inference**

vLLM automatically batches requests for efficiency:

```python
# Process all prompts in one call
outputs = llm.generate(prompts, sampling)

# vLLM handles batching internally
# No need to manually chunk prompts
```

**Step 4: Process results**

```python
# Extract generated text
results = []
for output in outputs:
    prompt = output.prompt
    generated = output.outputs[0].text
    results.append({
        "prompt": prompt,
        "generated": generated,
        "tokens": len(output.outputs[0].token_ids)
    })

# Save to file
import json
with open("results.jsonl", "w") as f:
    for result in results:
        f.write(json.dumps(result) + "\n")

print(f"Processed {len(results)} prompts")
```

### Workflow 3: Quantized model serving

Fit large models in limited GPU memory.

```
Quantization Setup:
- [ ] Step 1: Choose quantization method
- [ ] Step 2: Find or create quantized model
- [ ] Step 3: Launch with quantization flag
- [ ] Step 4: Verify accuracy
```

**Step 1: Choose quantization method**

- **AWQ**: Best for 70B models, minimal accuracy loss
- **GPTQ**: Wide model support, good compression
- **FP8**: Fastest on H100 GPUs

**Step 2: Find or create quantized model**

Use pre-quantized models from HuggingFace:

```bash
# Search for AWQ models
# Example: TheBloke/Llama-2-70B-AWQ
```

**Step 3: Launch with quantization flag**

```bash
# Using pre-quantized model
vllm serve TheBloke/Llama-2-70B-AWQ \
  --quantization awq \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.95

# Results: 70B model in ~40GB VRAM
```

**Step 4: Verify accuracy**

Test outputs match expected quality:

```python
# Compare quantized vs non-quantized responses
# Verify task-specific performance unchanged
```

## When to use vs alternatives

**Use vLLM when:**
- Deploying production LLM APIs (100+ req/sec)
- Serving OpenAI-compatible endpoints
- Limited GPU memory but need large models
- Multi-user applications (chatbots, assistants)
- Need low latency with high throughput

**Use alternatives instead:**
- **llama.cpp**: CPU/edge inference, single-user
- **HuggingFace transformers**: Research, prototyping, one-off generation
- **TensorRT-LLM**: NVIDIA-only, need absolute maximum performance
- **Text-Generation-Inference**: Already in HuggingFace ecosystem

## Common issues

**Issue: Out of memory during model loading**

Reduce memory usage:
```bash
vllm serve MODEL \
  --gpu-memory-utilization 0.7 \
  --max-model-len 4096
```

Or use quantization:
```bash
vllm serve MODEL --quantization awq
```

**Issue: Slow first token (TTFT > 1 second)**

Enable prefix caching for repeated prompts:
```bash
vllm serve MODEL --enable-prefix-caching
```

For long prompts, enable chunked prefill:
```bash
vllm serve MODEL --enable-chunked-prefill
```

**Issue: Model not found error**

Use `--trust-remote-code` for custom models:
```bash
vllm serve MODEL --trust-remote-code
```

**Issue: Low throughput (<50 req/sec)**

Increase concurrent sequences:
```bash
vllm serve MODEL --max-num-seqs 512
```

Check GPU utilization with `nvidia-smi` - should be >80%.

**Issue: Inference slower than expected**

Verify tensor parallelism uses power of 2 GPUs:
```bash
vllm serve MODEL --tensor-parallel-size 4  # Not 3
```

Enable speculative decoding for faster generation:
```bash
vllm serve MODEL --speculative-model DRAFT_MODEL
```

## Advanced topics

**Server deployment patterns**: See [references/server-deployment.md](references/server-deployment.md) for Docker, Kubernetes, and load balancing configurations.

**Performance optimization**: See [references/optimization.md](references/optimization.md) for PagedAttention tuning, continuous batching details, and benchmark results.

**Quantization guide**: See [references/quantization.md](references/quantization.md) for AWQ/GPTQ/FP8 setup, model preparation, and accuracy comparisons.

**Troubleshooting**: See [references/troubleshooting.md](references/troubleshooting.md) for detailed error messages, debugging steps, and performance diagnostics.

## Hardware requirements

- **Small models (7B-13B)**: 1x A10 (24GB) or A100 (40GB)
- **Medium models (30B-40B)**: 2x A100 (40GB) with tensor parallelism
- **Large models (70B+)**: 4x A100 (40GB) or 2x A100 (80GB), use AWQ/GPTQ

Supported platforms: NVIDIA (primary), AMD ROCm, Intel GPUs, TPUs

## Resources

- Official docs: https://docs.vllm.ai
- GitHub: https://github.com/vllm-project/vllm
- Paper: "Efficient Memory Management for Large Language Model Serving with PagedAttention" (SOSP 2023)
- Community: https://discuss.vllm.ai




```

---

## weights-and-biases

- **Name:** `weights-and-biases`
- **Version:** 1.0.0
- **Author:** Orchestra Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/mlops\evaluation\weights-and-biases/SKILL.md`

**Description:**

> W&B: log ML experiments, sweeps, model registry, dashboards.

**Full SKILL.md body:**

```markdown
---
name: weights-and-biases
description: "W&B: log ML experiments, sweeps, model registry, dashboards."
version: 1.0.0
author: Orchestra Research
license: MIT
dependencies: [wandb]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [MLOps, Weights And Biases, WandB, Experiment Tracking, Hyperparameter Tuning, Model Registry, Collaboration, Real-Time Visualization, PyTorch, TensorFlow, HuggingFace]

---

# Weights & Biases: ML Experiment Tracking & MLOps

## When to Use This Skill

Use Weights & Biases (W&B) when you need to:
- **Track ML experiments** with automatic metric logging
- **Visualize training** in real-time dashboards
- **Compare runs** across hyperparameters and configurations
- **Optimize hyperparameters** with automated sweeps
- **Manage model registry** with versioning and lineage
- **Collaborate on ML projects** with team workspaces
- **Track artifacts** (datasets, models, code) with lineage

**Users**: 200,000+ ML practitioners | **GitHub Stars**: 10.5k+ | **Integrations**: 100+

## Installation

```bash
# Install W&B
pip install wandb

# Login (creates API key)
wandb login

# Or set API key programmatically
export WANDB_API_KEY=your_api_key_here
```

## Quick Start

### Basic Experiment Tracking

```python
import wandb

# Initialize a run
run = wandb.init(
    project="my-project",
    config={
        "learning_rate": 0.001,
        "epochs": 10,
        "batch_size": 32,
        "architecture": "ResNet50"
    }
)

# Training loop
for epoch in range(run.config.epochs):
    # Your training code
    train_loss = train_epoch()
    val_loss = validate()

    # Log metrics
    wandb.log({
        "epoch": epoch,
        "train/loss": train_loss,
        "val/loss": val_loss,
        "train/accuracy": train_acc,
        "val/accuracy": val_acc
    })

# Finish the run
wandb.finish()
```

### With PyTorch

```python
import torch
import wandb

# Initialize
wandb.init(project="pytorch-demo", config={
    "lr": 0.001,
    "epochs": 10
})

# Access config
config = wandb.config

# Training loop
for epoch in range(config.epochs):
    for batch_idx, (data, target) in enumerate(train_loader):
        # Forward pass
        output = model(data)
        loss = criterion(output, target)

        # Backward pass
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        # Log every 100 batches
        if batch_idx % 100 == 0:
            wandb.log({
                "loss": loss.item(),
                "epoch": epoch,
                "batch": batch_idx
            })

# Save model
torch.save(model.state_dict(), "model.pth")
wandb.save("model.pth")  # Upload to W&B

wandb.finish()
```

## Core Concepts

### 1. Projects and Runs

**Project**: Collection of related experiments
**Run**: Single execution of your training script

```python
# Create/use project
run = wandb.init(
    project="image-classification",
    name="resnet50-experiment-1",  # Optional run name
    tags=["baseline", "resnet"],    # Organize with tags
    notes="First baseline run"      # Add notes
)

# Each run has unique ID
print(f"Run ID: {run.id}")
print(f"Run URL: {run.url}")
```

### 2. Configuration Tracking

Track hyperparameters automatically:

```python
config = {
    # Model architecture
    "model": "ResNet50",
    "pretrained": True,

    # Training params
    "learning_rate": 0.001,
    "batch_size": 32,
    "epochs": 50,
    "optimizer": "Adam",

    # Data params
    "dataset": "ImageNet",
    "augmentation": "standard"
}

wandb.init(project="my-project", config=config)

# Access config during training
lr = wandb.config.learning_rate
batch_size = wandb.config.batch_size
```

### 3. Metric Logging

```python
# Log scalars
wandb.log({"loss": 0.5, "accuracy": 0.92})

# Log multiple metrics
wandb.log({
    "train/loss": train_loss,
    "train/accuracy": train_acc,
    "val/loss": val_loss,
    "val/accuracy": val_acc,
    "learning_rate": current_lr,
    "epoch": epoch
})

# Log with custom x-axis
wandb.log({"loss": loss}, step=global_step)

# Log media (images, audio, video)
wandb.log({"examples": [wandb.Image(img) for img in images]})

# Log histograms
wandb.log({"gradients": wandb.Histogram(gradients)})

# Log tables
table = wandb.Table(columns=["id", "prediction", "ground_truth"])
wandb.log({"predictions": table})
```

### 4. Model Checkpointing

```python
import torch
import wandb

# Save model checkpoint
checkpoint = {
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss,
}

torch.save(checkpoint, 'checkpoint.pth')

# Upload to W&B
wandb.save('checkpoint.pth')

# Or use Artifacts (recommended)
artifact = wandb.Artifact('model', type='model')
artifact.add_file('checkpoint.pth')
wandb.log_artifact(artifact)
```

## Hyperparameter Sweeps

Automatically search for optimal hyperparameters.

### Define Sweep Configuration

```python
sweep_config = {
    'method': 'bayes',  # or 'grid', 'random'
    'metric': {
        'name': 'val/accuracy',
        'goal': 'maximize'
    },
    'parameters': {
        'learning_rate': {
            'distribution': 'log_uniform',
            'min': 1e-5,
            'max': 1e-1
        },
        'batch_size': {
            'values': [16, 32, 64, 128]
        },
        'optimizer': {
            'values': ['adam', 'sgd', 'rmsprop']
        },
        'dropout': {
            'distribution': 'uniform',
            'min': 0.1,
            'max': 0.5
        }
    }
}

# Initialize sweep
sweep_id = wandb.sweep(sweep_config, project="my-project")
```

### Define Training Function

```python
def train():
    # Initialize run
    run = wandb.init()

    # Access sweep parameters
    lr = wandb.config.learning_rate
    batch_size = wandb.config.batch_size
    optimizer_name = wandb.config.optimizer

    # Build model with sweep config
    model = build_model(wandb.config)
    optimizer = get_optimizer(optimizer_name, lr)

    # Training loop
    for epoch in range(NUM_EPOCHS):
        train_loss = train_epoch(model, optimizer, batch_size)
        val_acc = validate(model)

        # Log metrics
        wandb.log({
            "train/loss": train_loss,
            "val/accuracy": val_acc
        })

# Run sweep
wandb.agent(sweep_id, function=train, count=50)  # Run 50 trials
```

### Sweep Strategies

```python
# Grid search - exhaustive
sweep_config = {
    'method': 'grid',
    'parameters': {
        'lr': {'values': [0.001, 0.01, 0.1]},
        'batch_size': {'values': [16, 32, 64]}
    }
}

# Random search
sweep_config = {
    'method': 'random',
    'parameters': {
        'lr': {'distribution': 'uniform', 'min': 0.0001, 'max': 0.1},
        'dropout': {'distribution': 'uniform', 'min': 0.1, 'max': 0.5}
    }
}

# Bayesian optimization (recommended)
sweep_config = {
    'method': 'bayes',
    'metric': {'name': 'val/loss', 'goal': 'minimize'},
    'parameters': {
        'lr': {'distribution': 'log_uniform', 'min': 1e-5, 'max': 1e-1}
    }
}
```

## Artifacts

Track datasets, models, and other files with lineage.

### Log Artifacts

```python
# Create artifact
artifact = wandb.Artifact(
    name='training-dataset',
    type='dataset',
    description='ImageNet training split',
    metadata={'size': '1.2M images', 'split': 'train'}
)

# Add files
artifact.add_file('data/train.csv')
artifact.add_dir('data/images/')

# Log artifact
wandb.log_artifact(artifact)
```

### Use Artifacts

```python
# Download and use artifact
run = wandb.init(project="my-project")

# Download artifact
artifact = run.use_artifact('training-dataset:latest')
artifact_dir = artifact.download()

# Use the data
data = load_data(f"{artifact_dir}/train.csv")
```

### Model Registry

```python
# Log model as artifact
model_artifact = wandb.Artifact(
    name='resnet50-model',
    type='model',
    metadata={'architecture': 'ResNet50', 'accuracy': 0.95}
)

model_artifact.add_file('model.pth')
wandb.log_artifact(model_artifact, aliases=['best', 'production'])

# Link to model registry
run.link_artifact(model_artifact, 'model-registry/production-models')
```

## Integration Examples

### HuggingFace Transformers

```python
from transformers import Trainer, TrainingArguments
import wandb

# Initialize W&B
wandb.init(project="hf-transformers")

# Training arguments with W&B
training_args = TrainingArguments(
    output_dir="./results",
    report_to="wandb",  # Enable W&B logging
    run_name="bert-finetuning",
    logging_steps=100,
    save_steps=500
)

# Trainer automatically logs to W&B
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset
)

trainer.train()
```

### PyTorch Lightning

```python
from pytorch_lightning import Trainer
from pytorch_lightning.loggers import WandbLogger
import wandb

# Create W&B logger
wandb_logger = WandbLogger(
    project="lightning-demo",
    log_model=True  # Log model checkpoints
)

# Use with Trainer
trainer = Trainer(
    logger=wandb_logger,
    max_epochs=10
)

trainer.fit(model, datamodule=dm)
```

### Keras/TensorFlow

```python
import wandb
from wandb.keras import WandbCallback

# Initialize
wandb.init(project="keras-demo")

# Add callback
model.fit(
    x_train, y_train,
    validation_data=(x_val, y_val),
    epochs=10,
    callbacks=[WandbCallback()]  # Auto-logs metrics
)
```

## Visualization & Analysis

### Custom Charts

```python
# Log custom visualizations
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot(x, y)
wandb.log({"custom_plot": wandb.Image(fig)})

# Log confusion matrix
wandb.log({"conf_mat": wandb.plot.confusion_matrix(
    probs=None,
    y_true=ground_truth,
    preds=predictions,
    class_names=class_names
)})
```

### Reports

Create shareable reports in W&B UI:
- Combine runs, charts, and text
- Markdown support
- Embeddable visualizations
- Team collaboration

## Best Practices

### 1. Organize with Tags and Groups

```python
wandb.init(
    project="my-project",
    tags=["baseline", "resnet50", "imagenet"],
    group="resnet-experiments",  # Group related runs
    job_type="train"             # Type of job
)
```

### 2. Log Everything Relevant

```python
# Log system metrics
wandb.log({
    "gpu/util": gpu_utilization,
    "gpu/memory": gpu_memory_used,
    "cpu/util": cpu_utilization
})

# Log code version
wandb.log({"git_commit": git_commit_hash})

# Log data splits
wandb.log({
    "data/train_size": len(train_dataset),
    "data/val_size": len(val_dataset)
})
```

### 3. Use Descriptive Names

```python
# ✅ Good: Descriptive run names
wandb.init(
    project="nlp-classification",
    name="bert-base-lr0.001-bs32-epoch10"
)

# ❌ Bad: Generic names
wandb.init(project="nlp", name="run1")
```

### 4. Save Important Artifacts

```python
# Save final model
artifact = wandb.Artifact('final-model', type='model')
artifact.add_file('model.pth')
wandb.log_artifact(artifact)

# Save predictions for analysis
predictions_table = wandb.Table(
    columns=["id", "input", "prediction", "ground_truth"],
    data=predictions_data
)
wandb.log({"predictions": predictions_table})
```

### 5. Use Offline Mode for Unstable Connections

```python
import os

# Enable offline mode
os.environ["WANDB_MODE"] = "offline"

wandb.init(project="my-project")
# ... your code ...

# Sync later
# wandb sync <run_directory>
```

## Team Collaboration

### Share Runs

```python
# Runs are automatically shareable via URL
run = wandb.init(project="team-project")
print(f"Share this URL: {run.url}")
```

### Team Projects

- Create team account at wandb.ai
- Add team members
- Set project visibility (private/public)
- Use team-level artifacts and model registry

## Pricing

- **Free**: Unlimited public projects, 100GB storage
- **Academic**: Free for students/researchers
- **Teams**: $50/seat/month, private projects, unlimited storage
- **Enterprise**: Custom pricing, on-prem options

## Resources

- **Documentation**: https://docs.wandb.ai
- **GitHub**: https://github.com/wandb/wandb (10.5k+ stars)
- **Examples**: https://github.com/wandb/examples
- **Community**: https://wandb.ai/community
- **Discord**: https://wandb.me/discord

## See Also

- `references/sweeps.md` - Comprehensive hyperparameter optimization guide
- `references/artifacts.md` - Data and model versioning patterns
- `references/integrations.md` - Framework-specific examples



```

---
