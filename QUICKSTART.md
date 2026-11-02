# Quickstart Guide — `gradio-app-development`

This guide shows how to install and use the `gradio-app-development` skill with **Hermes Agent** (or directly from the command line) to spin up JIT Gradio 6 apps, loop modular `daggr` pipelines, and deploy to Hugging Face Spaces.

---

## 1. Install the Skill in 60 Seconds

```bash
# Install into Hermes Agent directly from GitHub
hermes skills tap add Jpalmer95/gradio-app-development
hermes skills install Jpalmer95/gradio-app-development/skills/gradio-app-development

# Verify it is installed
hermes skills list | grep gradio-app-development
```

Or clone this repository for local development and symlink/copy the skill directory:

```bash
git clone https://github.com/Jpalmer95/gradio-app-development.git
cd gradio-app-development
pip install "gradio>=6.27.0" "daggr>=0.8.0" "gradio_client>=1.0.0" "huggingface_hub>=1.0.0" pillow
```

---

## 2. Example Prompts for Your Agent

Once installed, start a new session (`/reset` or `hermes`) and ask your agent:

- **Modular `daggr` + Gradio App (Image / 3D / Audio):**
  > "Build a JIT Gradio app that takes a list of game item prompts, runs a modular daggr pipeline (prompt expansion → image gen → background removal) in a batch loop with a live progress bar, shows a Gallery + JSON trace, and lets me launch the daggr canvas with `--canvas`."
- **Standalone Python / Media Tool with MCP:**
  > "Create a local Gradio 6 app with `mcp_server=True` for trimming, normalizing, and converting audio files with ffmpeg + a spectrogram preview."
- **Deploy to Hugging Face Spaces:**
  > "Scaffold a Gradio app for our sprite pipeline, verify it locally, and deploy it to my Hugging Face account as a CPU Space using my `HF_TOKEN`."

---

## 3. Step-by-Step CLI Recipes

Set `SCRIPT` to the installed skill script (or the repo copy):

```bash
SCRIPT="$HOME/.hermes/skills/mlops/gradio-app-development/scripts/jit_gradio_space.py"
# Or from a repo checkout:
# SCRIPT="skills/gradio-app-development/scripts/jit_gradio_space.py"
```

### Recipe A: Build & Verify a Gradio App with a Looped `daggr` Pipeline (`daggr-modular`)

```bash
python3 "$SCRIPT" scaffold ./sprite-workbench \
  --title "Sprite Workbench" \
  --template daggr-modular \
  --short-description "Batch sprite workbench powered by a modular daggr pipeline"

# Run syntax check, README YAML check, import check, and headless daggr execution smoke test:
python3 "$SCRIPT" verify ./sprite-workbench

# Launch the Gradio 6 UI + MCP server on http://localhost:7860
python3 ./sprite-workbench/app.py

# Or launch the interactive node-by-node daggr DAG canvas directly:
python3 ./sprite-workbench/app.py --canvas
```

### Recipe B: Hybrid Gradio UI (`/app`) + Live `daggr` Canvas (`/`) in One Server (`daggr-hybrid`)

```bash
python3 "$SCRIPT" scaffold ./hybrid-studio \
  --title "Hybrid Studio" \
  --template daggr-hybrid \
  --short-description "Custom Gradio UI at /app + interactive daggr canvas at /"

python3 "$SCRIPT" verify ./hybrid-studio
python3 ./hybrid-studio/app.py
# Open http://localhost:7860/ for the daggr canvas, or http://localhost:7860/app/ for Gradio Blocks!
```

### Recipe C: Introspect Any Live HF Space Before Calling It

Never guess whether a Space expects `image`, `img`, or `input_image`, or whether its endpoint is `/infer`, `/run`, or `/predict`:

```bash
python3 "$SCRIPT" inspect-space black-forest-labs/FLUX.1-schnell
python3 "$SCRIPT" inspect-space not-lain/background-removal
```

### Recipe D: Deploy to Your Hugging Face Account (`HF_TOKEN`)

1. Ensure `HF_TOKEN` (with write permissions from [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)) is exported in your environment or stored in `~/.hermes/.env`:
   ```bash
   export HF_TOKEN=hf_your_token_here
   ```
2. Deploy with a single command:
   ```bash
   python3 "$SCRIPT" deploy ./sprite-workbench \
     --repo-id sprite-workbench \
     --hardware cpu-basic \
     --forward-hf-token
   ```
   - If `--repo-id` omits the username (`sprite-workbench`), the script automatically resolves your username via `huggingface_hub.whoami()` (`<your-username>/sprite-workbench`).
   - `--forward-hf-token` securely stores `HF_TOKEN` as a Space Secret so your deployed Space can call ZeroGPU Spaces or gated models.
   - Pass `--private` to create a private Space, or `--vendor ./my_shared_pkg` to bundle local helper modules into the Space upload.
