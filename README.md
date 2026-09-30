<div align="center">

# 🧳 Ailfred

**A fully local agentic AI butler. Your digital life, managed offline.**

[![Arch Linux](https://img.shields.io/badge/Arch_Linux-1793D1?style=flat&logo=arch-linux&logoColor=white)](https://archlinux.org/)
[![Python](https://img.shields.io/badge/Python_3.12-3776AB?style=flat&logo=python&logoColor=white)](https://python.org/)
[![Ollama](https://img.shields.io/badge/Ollama-000000?style=flat&logo=ollama&logoColor=white)](https://ollama.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status: WIP](https://img.shields.io/badge/Status-WIP-orange.svg)]()

Ailfred is a privacy-first, offline AI assistant that integrates with your Obsidian vault, calendar, email, terminal, and filesystem. It reasons through multi-step tasks autonomously using a ReAct agentic loop — no cloud, no telemetry, no data leaving your machine.

</div>

---

## ✨ What Ailfred Can Do

| Skill | Example |
|---|---|
| 🗂️ **Obsidian Q&A** | *"What did I write about attention mechanisms last week?"* |
| 📅 **Daily Briefing** | *"Good morning — here's your day."* (calendar + email + tasks, spoken aloud) |
| ⚙️ **Shell Automation** | *"Rename all JPEGs in Downloads to include today's date."* |
| 📧 **Email Summary** | *"Summarise my unread emails and flag anything urgent."* |
| 💻 **Coding Help** | *"Explain this traceback"* / *"Write a commit message for this diff."* |
| ⏰ **Cron Scheduling** | *"Remind me to drink water every 2 hours on weekdays."* |
| 📄 **Document Q&A** | *"Summarise this PDF paper and save key points to a note."* |
| 🖥️ **System Control** | *"Switch to workspace 3"* / *"What's eating my RAM?"* |

---

## 🏗️ Architecture

```
User (voice or text)
        │
        ▼
  STT — faster-whisper
        │
        ▼
  Agent Brain ◄─────────────────────────────┐
  Qwen2.5-7B (fine-tuned, Ollama)           │
  ReAct Loop: Thought → Action → Observe    │
        │                                   │
        ▼                                   │
  Tool Router                               │
  ├── obsidian_rag()      (ChromaDB + RAG)  │
  ├── calendar_query()    (Google API)      │
  ├── email_fetch()       (IMAP)            │
  ├── shell_exec()        (firejail)        │
  ├── cron_manager()      (python-crontab)  │
  ├── code_assist()       (Ollama /v1)      │
  ├── pdf_summarise()     (markitdown)      │
  └── file_ops()          (pathlib)    ─────┘
        │
        ▼
  TTS — Piper → Speaker
```

---

## 🚀 Getting Started

### Prerequisites

- Arch Linux (or any Linux distro — Arch recommended)
- NVIDIA GPU with CUDA (tested on RTX 4060 8GB)
- Python 3.12+
- [Ollama](https://ollama.com/) installed and running

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/ailfred.git
cd ailfred
```

### 2. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 3. Pull the model

```bash
ollama pull qwen2.5:7b-instruct
```

> Or use the fine-tuned Ailfred model once available (see [Fine-tuning](#fine-tuning)).

### 4. Set up credentials

```bash
cp .env.example .env
# Edit .env with your Google Calendar OAuth credentials and IMAP details
```

### 5. Index your Obsidian vault

```bash
python -m ailfred.rag.index --vault /path/to/your/vault
```

### 6. Run Ailfred

```bash
# Text mode
python -m ailfred

# Voice mode
python -m ailfred --voice
```

---

## ⚙️ Configuration

Edit `config.yaml`:

```yaml
llm:
  model: qwen2.5:7b-instruct   # or your fine-tuned model
  base_url: http://localhost:11434/v1

obsidian:
  vault_path: /path/to/your/vault
  journal_format: "Journal/{YYYY}/{MM}/{YY-MM-DD} {Www}.md"

voice:
  stt_model: medium             # whisper model size
  tts_voice: en_US-lessac-medium

sandbox:
  timeout: 30                   # seconds
  network: false                # firejail network isolation
```

---

## 📚 RAG Pipeline (Obsidian)

Ailfred indexes your vault with sentence-transformers and stores embeddings in ChromaDB. A file watcher keeps the index in sync as you write.

```bash
# Re-index vault manually
python -m ailfred.rag.index

# Query the vault directly (for debugging)
python -m ailfred.rag.query "what did I write about RAG?"
```

---

## 🔧 Fine-tuning

Ailfred can be fine-tuned on your own task data using QLoRA to improve tool-call reliability and adapt to your vocabulary and workflow.

```bash
# Generate synthetic training data
python scripts/generate_training_data.py --output data/train.jsonl

# Run QLoRA fine-tuning (requires ~7GB VRAM)
python scripts/finetune.py --config configs/qlora.yaml

# Export merged model to Ollama
python scripts/export_to_ollama.py --adapter outputs/ailfred-adapter
```

See [`docs/finetuning.md`](docs/finetuning.md) for the full guide.

---

## 🔒 Privacy

Ailfred is designed from the ground up to keep your data local:

- All LLM inference runs on your machine via Ollama
- No API keys required at runtime (Google Calendar OAuth tokens are stored locally)
- Shell scripts are executed inside a **firejail sandbox** (network disabled, no privilege escalation, read-only home directory)
- The Obsidian vault index lives in a local ChromaDB instance
- Nothing is logged externally

---

## 📦 Stack

| Layer | Technology |
|---|---|
| LLM runtime | Ollama |
| LLM model | Qwen2.5-7B-Instruct (fine-tuned via QLoRA) |
| Agent framework | LangGraph (ReAct loop) |
| Embeddings | sentence-transformers (`all-MiniLM-L6-v2`) |
| Vector store | ChromaDB |
| STT | faster-whisper |
| TTS | Piper |
| Code sandbox | firejail |
| Calendar | Google Calendar API |
| Email | imaplib (IMAP4_SSL) |
| Scheduling | python-crontab |
| OS | Arch Linux + Hyprland |

---

## 🗺️ Roadmap

- [x] ReAct agent loop
- [x] Obsidian RAG pipeline
- [x] Calendar and email integration
- [x] Shell execution sandbox (firejail)
- [x] Voice I/O (Whisper + Piper)
- [x] Cron job manager
- [ ] QLoRA fine-tuned model release
- [ ] Self-generating tools (butler writes and saves new tools)
- [ ] Vision input via Qwen2-VL (screenshot understanding)
- [ ] Hyprland workspace control
- [ ] Web UI (optional)
- [ ] Tailscale remote access

---

## 🤝 Contributing

Pull requests are welcome. For major changes, open an issue first.

```bash
# Lint
ruff check .

# Format
ruff format .

# Tests
pytest tests/
```

---

## 📄 License

MIT — see [LICENSE](LICENSE).

---

<div align="center">
  <sub>Built by Shamil · Runs on Athena (RTX 4060) · Powered by Ollama + Qwen2.5</sub>
</div>
