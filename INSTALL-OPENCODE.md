# GSE-One — opencode Install Guide

This document covers **only the opencode-specific install, upgrade, uninstall, and local-model setup**. For what GSE-One is, the list of commands, the architecture, and the full lifecycle, see [README.md](README.md) and [gse-one-spec.md](gse-one-spec.md).

---

## 1. Prerequisites

- **opencode** on your `PATH` — check with `opencode --version`. Install: https://opencode.ai
- **Python 3** — ships with macOS and Linux; on Windows use `python`.
- **git** — to clone this repo.

No Node.js, npm, or Bun install required — opencode runs the TS guardrails plugin itself.

---

## 2. Install

Pick one mode. **Mode A** is simplest for a single project; **Mode B** gives every project access to GSE-One.

### Mode A — Non-plugin (per-project)

```bash
git clone https://github.com/nicolasguelfi/gensem.git
cd gensem
python3 install.py --platform opencode --mode no-plugin --project-dir /path/to/your-project
```

Writes `<project>/.opencode/` + `AGENTS.md` + `opencode.json` at the project root. The only file outside the project is a 1-line `~/.gse-one` registry.

### Mode B — Plugin (global, `~/.config/opencode/`)

```bash
git clone https://github.com/nicolasguelfi/gensem.git
cd gensem
python3 install.py --platform opencode --mode plugin
```

### Interactive (auto-detects everything)

```bash
python3 install.py
```

After install, launch `opencode` in your project and type `/gse-go`.

---

## 3. Files written

| File | Mode A | Mode B |
|------|:------:|:------:|
| `<project>/.opencode/{skills,commands,agents,plugins}/` | ✓ | — |
| `<project>/AGENTS.md` (GSE block between `<!-- GSE-ONE START -->` / `<!-- GSE-ONE END -->`) | ✓ | — |
| `<project>/opencode.json` (deep-merged) | ✓ | — |
| `~/.config/opencode/{skills,commands,agents,plugins}/` | — | ✓ |
| `~/.config/opencode/AGENTS.md` | — | ✓ |
| `~/.config/opencode/opencode.json` (deep-merged) | — | ✓ |
| `~/.gse-one` (1-line path registry) | ✓ | ✓ |

User content outside the GSE-ONE markers is preserved on reinstall and fully restored on uninstall. `opencode.json` deep-merge never overwrites your keys.

---

## 4. Upgrade

```bash
cd /path/to/gensem && git pull
cd gse-one && python3 gse_generate.py --verify
cd .. && python3 install.py --platform opencode --mode <your-mode> [--project-dir /path/to/your-project]
```

Reinstall is idempotent (surgical block replace + deep merge).

---

## 5. Uninstall

```bash
python3 install.py --uninstall --platform opencode --mode <your-mode> [--project-dir /path/to/your-project]
```

`AGENTS.md` loses its GSE block (file deleted if empty); `opencode.json` loses the GSE-added keys (file deleted if only `$schema` remains); `~/.gse-one` is removed.

---

## 6. Run opencode with a local model (Ollama / LM Studio)

opencode talks to any OpenAI-compatible endpoint, so Ollama and LM Studio both work out of the box. Use this if you want **privacy**, **zero API cost**, or **offline** operation.

### 6.1 Recommended local coding models (April 2026)

opencode runs an agentic loop with tool calls. A model that can't reliably call tools will silently fail. Pick from this short list — all have been observed to work for opencode-style workflows. **Context window ≥ 64k tokens** is strongly recommended by the opencode docs.

| Model (Ollama tag) | Params | Min VRAM/RAM | Notes |
|---|---|---|---|
| `qwen3-coder` (a.k.a. Qwen3-Coder-Next) | 80B MoE (3B active) | 8 GB | Best efficiency/quality ratio in 2026; designed for agent tool-calling. |
| `qwen2.5-coder:32b` | 32B dense | 24 GB | Python-first, strong code completion; the default "big local" option. |
| `llama3.3:70b` | 70B dense | 32 GB (Apple Silicon 48 GB+) | GPT-4-class generalist; slower but very strong on full-file edits. |
| `deepseek-r1:14b` | 14B dense | 16 GB | Chain-of-thought; excellent for debugging and code review. |
| `gpt-oss:20b` | 20B dense | 16 GB | OpenAI open-source; enable high-thinking mode for agent reliability. |
| `devstral-small-2:24b` | 24B dense | 16 GB | Good fallback when VRAM is tight; lighter than Qwen Coder 32B. |

**What to avoid for opencode:** Qwen 3 14B (plain), Devstral Small 2 in agent mode, GPT-OSS 20B in default (non-thinking) mode — reports of tool-call failures and instruction drift. Pick a larger or MoE variant if you have the VRAM.

Ollama tags evolve; run `ollama search coder` to see what's current.

### 6.2 Option A — Ollama

```bash
# Install Ollama (macOS example)
brew install ollama
ollama serve &                       # background server on :11434

# Pull one of the recommended models
ollama pull qwen2.5-coder:32b        # or qwen3-coder, llama3.3:70b, etc.
```

Then add an Ollama provider to your opencode config. In **Mode A** edit `<project>/opencode.json`, in **Mode B** edit `~/.config/opencode/opencode.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": {
        "qwen2.5-coder:32b": { "name": "Qwen 2.5 Coder 32B" }
      }
    }
  }
}
```

Launch opencode and type `/models` → pick **Qwen 2.5 Coder 32B**. To make it the default, add `"model": "ollama/qwen2.5-coder:32b"` at the top level of `opencode.json`.

> **Shortcut:** if you're on Ollama ≥ 0.5, `ollama launch opencode` passes a ready-made config via `OPENCODE_CONFIG_CONTENT` and deep-merges with your existing `opencode.json`.

### 6.3 Option B — LM Studio

1. Install LM Studio: https://lmstudio.ai
2. Download a recommended model from the Models tab (Qwen Coder, DeepSeek R1, etc.).
3. Open the **Developer** tab → **Start Server** (default port `1234`). Ensure the model is loaded.

Add the provider to `opencode.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": { "baseURL": "http://127.0.0.1:1234/v1" },
      "models": {
        "qwen2.5-coder-32b-instruct": { "name": "Qwen 2.5 Coder 32B (LM Studio)" }
      }
    }
  }
}
```

Replace the model ID with whatever LM Studio reports for your loaded model (see its "Local Server" panel). In opencode: `/models` → LM Studio → pick it. No real API key is required; if prompted, enter any non-empty string.

### 6.4 Tuning for GSE-One agentic flow

- **Context ≥ 64k.** GSE-One loads the orchestrator body (~400 lines of methodology) into every session via `AGENTS.md`. Smaller contexts truncate it.
- **Temperature low.** `0.0–0.3` for deterministic tool calls. Most local engines respect `options.temperature` in the provider block.
- **Tool calling must be on.** Ollama exposes it by default for compatible models; LM Studio exposes it when you tick "Function Calling" in the server panel.
- **Watch VRAM.** The orchestrator + 1-2 open files + a long conversation can push past 32 GB on the 70B models. If the model starts hallucinating, check you're not being silently evicted to disk swap.
- **`websearch` needs a key when running local.** opencode's built-in `websearch` tool is only active when you use opencode's own cloud provider, **or** when you export `OPENCODE_ENABLE_EXA=1` plus an [Exa](https://exa.ai) API key (`EXA_API_KEY`). With Ollama or LM Studio alone, `websearch` is silently unavailable. `webfetch` (direct HTTP) works in all cases, so GSE-One activities that need to read a specific URL still run. Example shell setup:
  ```bash
  export OPENCODE_ENABLE_EXA=1
  export EXA_API_KEY=xxx   # from https://exa.ai
  ```

---

## 7. Troubleshooting

- **`/gse-*` commands missing** — check the `commands/` dir exists at the install target. Restart opencode.
- **Model ignores GSE-One methodology** — `AGENTS.md` lost its GSE block; reinstall. Or context window too small (try ≥ 64k).
- **"Skill skipped: missing name/description"** — regenerate: `cd gse-one && python3 gse_generate.py --verify`.
- **Guardrails don't fire on `git commit` on main** — `plugins/gse-guardrails.ts` missing or opencode version doesn't support `tool.execute.before`. Upgrade opencode.
- **Local model makes tool calls but never finishes the loop** — model is too weak for agentic work. Swap for a larger variant or one from §6.1.
- **`.claude/skills/` + `.opencode/skills/` coexist** — opencode loads both → duplicate commands. Installer warns at install time; remove one of the two.

---

## 8. References

- [README.md](README.md) — what GSE-One is, all three platforms, quickstart
- [gse-one-spec.md](gse-one-spec.md) — full methodology & 23-command reference
- [CHANGELOG.md](CHANGELOG.md)

**Model research sources (April 2026):**
- [opencode — Providers](https://opencode.ai/docs/providers/)
- [opencode — Models](https://opencode.ai/docs/models/)
- [Ollama × opencode integration](https://docs.ollama.com/integrations/opencode)
- [Best Local AI Coding Models 2026](https://localaimaster.com/models/best-local-ai-coding-models)
- [Best LLMs for opencode — Tested Locally](https://dev.to/rosgluk/best-llms-for-opencode-tested-locally-499l)
- [OpenCode CLI Guide 2026 — Local LLMs](https://yuv.ai/learn/opencode-cli)
