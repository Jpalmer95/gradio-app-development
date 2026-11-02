# AGENTS.md — Gradio JIT Application & Modular Pipeline Skill

This repository houses the **`gradio-app-development`** skill for [Hermes Agent](https://github.com/NousResearch/hermes-agent) (and compatible AI coding agents following the open `SKILL.md` / `agents.md` standard).

It enables any agent to create, verify, and deploy **Just-In-Time (JIT) Gradio 6 applications** — locally or as Hugging Face Spaces — both **standalone** and **with modular `daggr` or `gr.Workflow` pipelines** looped or embedded inside the application.

## Repository Layout

```text
gradio-app-development/
├── AGENTS.md                   # Agent entry contract (this file)
├── README.md                   # Overview, architecture modes, install, upstream PR guide
├── QUICKSTART.md               # Copy-paste recipes for humans and agents
├── MASTER_PLAN.md              # Roadmap & recorded architectural decisions
├── LICENSE                     # Apache-2.0
├── skills/
│   └── gradio-app-development/ # Installable Hermes skill unit (keep self-contained!)
│       ├── SKILL.md
│       ├── references/
│       │   ├── live-docs-and-api-cheatsheet.md
│       │   └── daggr-in-gradio-patterns.md
│       └── scripts/
│           └── jit_gradio_space.py
└── tests/
    └── test_gradio_app_development_skill.py
```

## Critical Rules for Agents Working in This Repo

1. **Keep `skills/gradio-app-development/` 100% self-contained.**
   When a user runs `hermes skills install Jpalmer95/gradio-app-development/skills/gradio-app-development`, Hermes copies **only** `skills/gradio-app-development/`. Every script (`scripts/jit_gradio_space.py`) and reference doc (`references/*.md`) that the skill needs at runtime must live inside `skills/gradio-app-development/`.
2. **Zero machine-local paths.**
   Never put `/home/<user>/...` or OS-specific `/tmp` paths in `skills/gradio-app-development/`. Use `tempfile.gettempdir()` and `pathlib.Path` so the skill passes upstream `NousResearch/hermes-agent` skill review and runs on Linux, macOS, and Windows.
3. **Hardline `SKILL.md` frontmatter compliance.**
   - `description` must be **≤ 60 characters**, one sentence, ending with a period (`.`): `Build and deploy JIT Gradio apps, daggr loops, and Spaces.`
   - `author` must credit the human contributor first: `Jonathan Korstad (Jpalmer95), Hermes Agent`
   - `platforms: [linux, macos, windows]`
   - `related_skills` must only list skills that exist in the upstream `NousResearch/hermes-agent` tree (`[huggingface-hub, comfyui]`).
4. **Never guess a Hugging Face Space's API.**
   Before wiring any remote HF Space into a `GradioNode` or `gradio_client.Client`, run:
   ```bash
   python3 skills/gradio-app-development/scripts/jit_gradio_space.py inspect-space <owner/space>
   ```
5. **Gradio 6 API hygiene.**
   - Pass `theme=`, `css=`, `js=`, `head=`, and `footer_links=` to `demo.launch(...)`, **never** `gr.Blocks(...)`.
   - Use `api_visibility="public" | "undocumented" | "private"` on event listeners instead of the removed `show_api` or `api_name=False`.
   - Pass `token=` (not `hf_token=`) to `gradio_client.Client`.
6. **Offline, deterministic tests.**
   `tests/test_gradio_app_development_skill.py` must never make live network calls to Hugging Face Hub or third-party Spaces. Mock `gradio_client.Client` and `huggingface_hub.HfApi` in unit tests.

## Running Verification & Tests

```bash
# Run the full offline test suite (validates frontmatter, lint rules, all 4 templates, and mocked Space deploy)
python3 -m unittest discover -s tests -v

# Scaffold and smoke-verify a modular Gradio + daggr app in a temp dir
python3 skills/gradio-app-development/scripts/jit_gradio_space.py scaffold ./tmp_demo --template daggr-modular
python3 skills/gradio-app-development/scripts/jit_gradio_space.py verify ./tmp_demo
rm -rf ./tmp_demo
```

## Installing Locally into Hermes Agent

```bash
hermes skills tap add Jpalmer95/gradio-app-development
hermes skills install Jpalmer95/gradio-app-development/skills/gradio-app-development
```
