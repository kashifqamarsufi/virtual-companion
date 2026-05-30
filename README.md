# Virtual Companion — AI-Powered NPC System

> **Kashif Qamar** · [@kashifqamarsufi](https://github.com/kashifqamarsufi)  
> BTech Computer Science & Engineering · IUST Kashmir  
> `Status: 🔧 Active Development`

---

## What is this?

**Virtual Companion** is a real-time voice-interactive NPC system that replaces scripted dialogue trees with genuine conversational AI. A player speaks — the character understands, feels, responds, and animates. No menus. No pre-selected options. Just a conversation.

The system connects a Python AI backend to an Unreal Engine 5 environment via a FastAPI REST bridge. The NPC is a photorealistic MetaHuman character with lip sync and emotion-driven facial animation.

```
You speak → Whisper transcribes → NLP detects intent → Emotion scored
→ Response generated → Coqui TTS speaks → MetaHuman animates
```

---

## Current State

The Python backend (`bot_backend/`) is functional. The Unreal Engine project (`Main.uproject`) with MetaHuman character is integrated. The HTTP bridge between both systems is working.

> This is an active BTech final-year project. Features are being added incrementally.

---

## Architecture

```
┌──────────────────────────────────────────────┐
│            PRESENTATION LAYER                │
│     Unreal Engine 5  ·  MetaHuman Creator    │
│     Voice Input  →  3D Character Interface   │
└─────────────────────┬────────────────────────┘
                      │  HTTP POST
┌─────────────────────▼────────────────────────┐
│            COMMUNICATION LAYER               │
│         FastAPI Server  ·  Port 8765         │
│               main.py                        │
└─────────────────────┬────────────────────────┘
                      │  Audio + JSON
┌─────────────────────▼────────────────────────┐
│          APPLICATION LAYER — AI ENGINE       │
│                                              │
│  whisper_stt.py   →   rule_nlp.py            │
│       ↓                    ↓                 │
│  Transcribed Text     Intent + Keywords      │
│                            ↓                 │
│                   emotion_detector.py        │
│                    Emotion + Intensity        │
│                            ↓                 │
│                   Response Generator         │
│                            ↓                 │
│                   coqui_tts.py               │
│                    Synthesized Audio         │
└──────────────────────────────────────────────┘
```

---

## Repository Structure

```
virtual-companion/
│
├── bot_backend/              # Python AI engine
│   ├── main.py               # FastAPI server — entry point (port 8765)
│   ├── whisper_stt.py        # Whisper speech-to-text (faster-whisper, CUDA)
│   ├── rule_nlp.py           # Rule-based NLP — intent dict + process method
│   ├── coqui_tts.py          # Coqui TTS — VCTK/VITS model, speaker p226
│   └── emotion_detector.py   # J-Heart emotion classifier + intensity scorer
│
├── Config/                   # Unreal Engine project config
├── Content/                  # UE5 assets, MetaHuman, Blueprints
├── Intermediate/             # UE5 build intermediates
├── Saved/                    # UE5 saved data
├── Platforms/Android/Config/ # Android platform config
│
├── Main.uproject             # Unreal Engine 5 project file
├── .gitattributes            # Git LFS config
└── README.md
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Game Engine | Unreal Engine 5 + MetaHuman Creator |
| NPC Character | MetaHuman (facial rig, lip sync, animation) |
| Backend Language | Python 3.10+ |
| API Server | FastAPI (Uvicorn, port 8765) |
| Speech-to-Text | faster-whisper — `tiny` model, CUDA float16 |
| Text-to-Speech | Coqui TTS — `tts_models/en/vctk/vits` |
| Audio I/O | sounddevice + numpy |
| NLP Engine | Custom rule-based (no external LLM APIs) |
| Emotion Model | J-Heart pretrained classifier |
| UE5 ↔ Python | HTTP REST via Unreal Blueprint HTTP nodes |

---

## Setup

### Requirements

- Python 3.10+
- CUDA-compatible GPU (recommended for real-time inference)
- Unreal Engine 5.x with MetaHuman plugin enabled
- Git LFS (the repo uses LFS for large UE5 assets)

### Clone

```bash
git clone https://github.com/kashifqamarsufi/virtual-companion.git
cd virtual-companion
```

### Python Backend

```bash
cd bot_backend
pip install -r requirements.txt
```

```
# requirements.txt
fastapi
uvicorn
faster-whisper
TTS
sounddevice
numpy
torch
```

### Run the AI Server

```bash
python main.py
```

Server starts at `http://127.0.0.1:8765`. Keep this running before launching the UE5 project.

### Unreal Engine

1. Open `Main.uproject` in Unreal Engine 5
2. Ensure the MetaHuman plugin is active
3. Open the HTTP Communication Blueprint and confirm the endpoint points to `127.0.0.1:8765`
4. Press **Play** — speak into your microphone

---

## API

### `POST /bot/start_recording`
Starts microphone capture in a background thread.
```json
{ "status": "recording started" }
```

### `POST /bot/stop_recording`
Stops capture, runs the full pipeline, returns the NPC's response.
```json
{
  "intent": "greeting",
  "response_text": "Hello! How can I help you today?",
  "emotion": "happy",
  "intensity": 0.7
}
```

`intensity` is a float from `0.0` to `1.0` — passed to Unreal Engine to scale facial expression blend weights proportionally.

---

## How Emotion Intensity Works

Each detected emotion gets a scalar intensity score computed from:

1. **Keyword match** — base score assigned per emotional keyword
2. **Emphasis modifiers** — words like `very`, `really`, `extremely` multiply the base
3. **Punctuation signals** — exclamation marks and ALL CAPS raise the weight

```
"I am happy"              → Happy · 0.4
"I am really happy"       → Happy · 0.7
"I am extremely frustrated" → Angry · 0.9
```

Unreal Engine maps this value to animation blend weights — stronger emotion, stronger facial expression.

---

## Sample Interactions

| Input | Intent | Response | Emotion | Intensity |
|---|---|---|---|---|
| "Hello" | greeting | "Hello! How can I help you today?" | Happy | 0.7 |
| "Goodbye" | farewell | "Farewell! It was nice talking to you!" | Neutral | 0.5 |
| "How are you?" | how_are_you | "I'm doing great, thank you for asking!" | Happy | 0.8 |
| "I'm really sad" | emotional | "I'm sorry you're feeling that way." | Sad | 0.8 |

---

## Known Limitations

- NLP is rule-based — inputs with no keyword match fall to a default response
- Emotion values are currently tied to intent rules, not dynamically inferred from speech tone
- Response variety is limited to predefined sets per intent
- `venv/` should not be committed — add it to `.gitignore`

---

## Roadmap

- [ ] Replace rule NLP with fine-tuned transformer (DistilGPT-2 or LLaMA-3)
- [ ] Prosody-based emotion detection from raw audio
- [ ] Dynamic facial blend weight control from live intensity values
- [ ] Context window — NPC remembers earlier turns in the conversation
- [ ] Multi-NPC support with distinct personalities
- [ ] Packaging as a standalone Unreal Engine plugin

---

## Academic Context

Developed as the **Major Final Year Project** for the Bachelor of Technology in Computer Science and Engineering at the Islamic University of Science & Technology, Awantipora, Kashmir.

**Supervisor:** Dr. Asif Ali Banka, Assistant Professor, Dept. of CSE, IUST  
**Co-Supervisor:** Dr. Insha Altaf, Assistant Professor, Dept. of CSE, IUST  
**Academic Year:** 2025–2026

---

## References

1. Vaswani et al., *Attention Is All You Need*, NeurIPS 2017
2. Van den Oord et al., *WaveNet: A Generative Model for Raw Audio*, DeepMind 2016
3. Hannun et al., *Deep Speech: Scaling Up End-to-End Speech Recognition*, arXiv 2014
4. Wang et al., *Tacotron: Towards End-to-End Speech Synthesis*, Google 2017
5. Devlin et al., *BERT: Pre-training of Deep Bidirectional Transformers*, NAACL 2019
6. Epic Games, *Unreal Engine Documentation*
7. Epic Games, *MetaHuman Creator Documentation*

---

DEMO
https://youtu.be/58Q1ojUgrp4?si=3saZcmo-pbIgT7rT
