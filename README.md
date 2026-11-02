# ⚡ Gradio JIT Application & Modular Pipeline Skill (`gradio-app-development`)

A self-contained [Hermes Agent](https://github.com/NousResearch/hermes-agent) skill and CLI toolkit for building, verifying, and deploying **Just-In-Time (JIT) Gradio 6 applications** — locally or as Hugging Face Spaces — both **standalone** and **with modular [`daggr`](https://github.com/gradio-app/daggr) or `gr.Workflow` pipelines** looped or embedded inside the application.

Whether you need a custom image/video editor, a 3D mesh & rigging workbench, an audio/music synthesizer, or a multi-step Python code/data tool, this skill equips your agent to:
1. **Introspect** live Hugging Face Spaces (`gradio_client.Client.view_api`) so it never guesses endpoint or parameter names.
2. **Scaffold** a complete, Gradio 6-compliant application (`app.py`, `requirements.txt`, and HF Spaces `README.md` with YAML frontmatter) in one of four architectural modes.
3. **Wire & Loop Modular `daggr` Pipelines** inside standard `gr.Blocks` callbacks or batch loops using `SequentialExecutor` (with live `gr.Progress` streaming and intermediate trace inspection), or mount the interactive `DaggrServer` visual DAG canvas alongside your UI.
4. **Verify** syntax, frontmatter, imports, and headless execution before launching or uploading.
5. **Deploy** directly to a user's Hugging Face account (`HfApi.create_repo`, `upload_folder`, `add_space_secret`, `request_space_hardware`) using `HF_TOKEN`.

---

## Install (Hermes Agent)

```bash
# Add this repository as a skill tap and install the skill:
hermes skills tap add Jpalmer95/gradio-app-development
hermes skills install Jpalmer95/gradio-app-development/skills/gradio-app-development
```

> **Why `skills/gradio-app-development`?** Hermes skill taps look under `skills/` by default and copy only the target skill folder. All helper scripts (`scripts/jit_gradio_space.py`) and reference guides (`references/*.md`) live inside `skills/gradio-app-development/` so nothing is dropped during installation.

### Python Dependencies

Install the runtime packages in your active environment (or `uv` virtualenv):

```bash
pip install "gradio>=6.27.0" "daggr>=0.8.0" "gradio_client>=1.0.0" "huggingface_hub>=1.0.0" pillow
```

---

## The 4 JIT Application Modes

| Template (`--template`) | Architecture | Best For |
|---|---|---|
| **`blocks`** | Standalone `gr.Blocks()` + `demo.launch(mcp_server=True)` + `@spaces.GPU` shim | Single-step or multi-tab utilities (image/video filters, 3D viewers, audio synths, Python calculators) with automatic MCP tool exposure. |
| **`daggr-modular`** | `gr.Blocks()` UI calling/looping a modular `daggr.Graph` via `SequentialExecutor` + `--canvas` switch | Guided apps that run a multi-node `daggr` pipeline inside callbacks or batch loops (`yield` + `gr.Progress`), returning both final artifacts and full intermediate node traces (`ex.results`). |
| **`daggr-hybrid`** | Single-process `FastAPI` + `gr.mount_gradio_app` (`/app`) + `DaggrServer(graph).app` (`/`) | Power tools where users want both a custom Gradio form UI (`/app`) and the live node-by-node `daggr` visual DAG canvas (`/`) in one server/Space. |
| **`workflow`** | Native Gradio 6 `gr.Workflow(bind={...}, edges=[...], graph="workflow.json")` | Drag-and-drop Gradio 6 workflow canvases with automatic dual App/Canvas view (`?ui=app` / `?ui=canvas`) and REST API generation. Includes graceful fallback on Gradio `< 6.27`. |

---

## CLI Toolkit (`jit_gradio_space.py`)

The bundled script `skills/gradio-app-development/scripts/jit_gradio_space.py` works standalone from the terminal or inside agent tool calls:

```bash
SCRIPT="skills/gradio-app-development/scripts/jit_gradio_space.py"

# 1. Inspect any live Hugging Face Space's API endpoints and parameter types
python3 "$SCRIPT" inspect-space black-forest-labs/FLUX.1-schnell

# 2. Scaffold a new JIT Gradio + modular daggr application
python3 "$SCRIPT" scaffold ./my-jit-app \
  --title "Sprite & 3D Studio" \
  --template daggr-modular \
  --short-description "JIT sprite prep and preview with modular daggr pipeline"

# 3. Verify syntax, README YAML frontmatter, imports, and run a headless smoke test
python3 "$SCRIPT" verify ./my-jit-app

# 4. Run locally (Gradio UI + MCP server on :7860, or pass --canvas for the daggr DAG UI)
python3 ./my-jit-app/app.py
python3 ./my-jit-app/app.py --canvas

# 5. Optional: Deploy to Hugging Face Spaces with HF_TOKEN
export HF_TOKEN=hf_...
python3 "$SCRIPT" deploy ./my-jit-app \
  --repo-id your-username/sprite-3d-studio \
  --hardware cpu-basic \
  --forward-hf-token
```

See **[QUICKSTART.md](QUICKSTART.md)** for complete copy-paste recipes and agent prompts.

---

## Live Documentation Index (Included in Skill Context)

- **Gradio 6:** [Docs](https://www.gradio.app/docs) · [Guides](https://www.gradio.app/guides) · [Gradio 6 Migration Guide](https://www.gradio.app/guides/gradio-6-migration-guide) · [Blocks & Event Listeners](https://www.gradio.app/guides/blocks-and-event-listeners) · [Server Mode (`gradio.Server`)](https://www.gradio.app/guides/server-mode) · [Workflows (`gr.Workflow`)](https://www.gradio.app/guides/workflows) · [Building an MCP Server](https://www.gradio.app/guides/building-mcp-server-with-gradio) · [Python `gradio_client`](https://www.gradio.app/guides/getting-started-with-the-python-client) · [Custom HTML Components](https://www.gradio.app/guides/custom-HTML-components) · [3D Model Component](https://www.gradio.app/guides/how-to-use-3D-model-component)
- **Hugging Face Spaces:** [Gradio Spaces](https://huggingface.co/docs/hub/spaces-sdks-gradio) · [Spaces YAML Config Reference](https://huggingface.co/docs/hub/spaces-config-reference) · [ZeroGPU (`@spaces.GPU`)](https://huggingface.co/docs/hub/spaces-zerogpu) · [Manage Spaces (`HfApi`)](https://huggingface.co/docs/huggingface_hub/guides/manage-spaces) · [Spaces OAuth (`hf_oauth: true`)](https://huggingface.co/docs/hub/spaces-oauth)
- **`daggr` & Companion Projects:** [`gradio-app/daggr`](https://github.com/gradio-app/daggr) · [`Jpalmer95/daggr-pipelines`](https://github.com/Jpalmer95/daggr-pipelines) · [`Jpalmer95/daggr-studio`](https://github.com/Jpalmer95/daggr-studio) ([Live Space](https://huggingface.co/spaces/jkorstad/daggr-studio))

---

## Running Tests

The test suite runs offline (stdlib `unittest` / `pytest`, no network calls required) and validates `SKILL.md` frontmatter rules, path portability, all 4 application templates, and mocked Hub/Client operations:

```bash
python3 -m unittest discover -s tests -v
```

---

## Upstream Contribution to `NousResearch/hermes-agent`

This skill is authored to meet all `NousResearch/hermes-agent` skill-authoring standards (`description <= 60` chars, human-first `author`, audited `platforms`, no machine-local paths, offline unit tests). To submit upstream as an official optional skill:

1. In your fork of `NousResearch/hermes-agent`, create a feature branch (`git checkout -b feat/gradio-app-development-skill`).
2. Copy `skills/gradio-app-development/` to `optional-skills/mlops/gradio-app-development/`.
3. Copy `tests/test_gradio_app_development_skill.py` to `tests/skills/test_gradio_app_development_skill.py` and run `pytest tests/skills/test_gradio_app_development_skill.py -q`.
4. Regenerate docs (`python website/scripts/generate-skill-docs.py`), keep only the new skill's doc/catalog/sidebar entries (`git checkout --` unrelated doc drift), and open a Pull Request.

---

## License

Apache-2.0. Copyright 2026 Jonathan Korstad. See [LICENSE](LICENSE).
