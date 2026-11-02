# MASTER_PLAN.md — `gradio-app-development` Skill & JIT Space Toolkit

## Goal

Provide a portable, self-contained Hermes Agent skill (`gradio-app-development`) and CLI toolkit (`jit_gradio_space.py`) so any agent can build, verify, and deploy **Just-In-Time (JIT) Gradio 6 applications** — locally or on Hugging Face Spaces — both standalone and with modular `daggr` or `gr.Workflow` pipelines looped or mounted inside the app.

## Milestones

- [x] **Phase 1: Core Skill & Reference Architecture**
  - [x] Curate direct links to live Gradio 6 docs (`Blocks`, `Server`, `Workflow`, `HTML`, MCP, `gradio_client`, Migration Guide), Hugging Face Spaces docs (`spaces-sdks-gradio`, `spaces-config-reference`, `spaces-zerogpu`, `manage-spaces`, `spaces-oauth`), and `daggr` docs.
  - [x] Document the 4 JIT Application Modes (`blocks`, `daggr-modular`, `daggr-hybrid`, `workflow`).
  - [x] Document domain recipes for image/video editing, 3D/GLB workflows, audio/TTS/music generation, and multistep Python/data tools.
- [x] **Phase 2: Bundled Scaffolder, Verifier & HF Space Deployer (`jit_gradio_space.py`)**
  - [x] `inspect-space`: Live endpoint & parameter introspection via `gradio_client.Client.view_api`.
  - [x] `scaffold`: Generate `app.py`, `requirements.txt`, and valid HF Spaces `README.md` YAML frontmatter for all 4 templates (`blocks`, `daggr-modular`, `daggr-hybrid`, `workflow`).
  - [x] `verify`: AST syntax check, `README.md` frontmatter validation, module import check, and headless execution smoke test.
  - [x] `deploy`: Create/update Hugging Face Space via `HfApi`, vendor local modules (`--vendor`), attach secrets (`--forward-hf-token` / `--secret`), configure hardware, and report live runtime stage.
- [x] **Phase 3: Standalone Public Repo & Upstream Readiness**
  - [x] Structure under `skills/gradio-app-development/` so `hermes skills tap add` and `hermes skills install` work directly from GitHub.
  - [x] Meet all `NousResearch/hermes-agent` `hermes-agent-skill-authoring` hardline checks (≤60 char description, human-first author attribution, cross-platform `tempfile.gettempdir()`, zero machine-local paths, offline unit tests).
  - [x] Write `README.md`, `QUICKSTART.md`, `AGENTS.md`, and `tests/test_gradio_app_development_skill.py`.
- [ ] **Phase 4: Upstream Community PR to `NousResearch/hermes-agent`**
  - [ ] Copy `skills/gradio-app-development/` to `optional-skills/mlops/gradio-app-development/` in `hermes-agent` fork branch.
  - [ ] Copy `tests/test_gradio_app_development_skill.py` to `tests/skills/test_gradio_app_development_skill.py`.
  - [ ] Run `python website/scripts/generate-skill-docs.py` with scope discipline and open upstream PR.

## Recorded Architectural Decisions

1. **Why 4 templates (`blocks`, `daggr-modular`, `daggr-hybrid`, `workflow`):**
   - Single-step or multi-tab utilities need clean `gr.Blocks` + `mcp_server=True`.
   - Multi-step workflows inside a larger app work best when a `daggr.Graph` is executed headlessly via `SequentialExecutor` inside callbacks or Python batch loops (`daggr-modular`), avoiding `daggr 0.8.0`'s scatter/gather bugs while keeping full intermediate node traces.
   - Power users who want both a custom UI and node-by-node inspection need `DaggrServer(graph).app` mounted alongside `gr.Blocks` (`daggr-hybrid`).
   - Gradio `>=6.27.0` also ships native `gr.Workflow` (`workflow`), while `daggr 0.8.0` remains heavily used for programmatic code-first graphs; supporting both with a graceful `<6.27` fallback gives agents maximum flexibility.
2. **Why `skills/gradio-app-development/` instead of repo root:**
   - Hermes skill taps look under `skills/` by default and copy only the target skill folder (`owner/repo/skills/<name>`). Putting `scripts/` and `references/` inside `skills/gradio-app-development/` ensures nothing is dropped on install.
3. **Why `hf_oauth: true` in generated Space `README.md`:**
   - ZeroGPU Spaces bill GPU seconds against the signed-in visitor's HF OAuth token when `hf_oauth: true` is present; without it, all visitors share a tiny anonymous IP quota and fail quickly.
