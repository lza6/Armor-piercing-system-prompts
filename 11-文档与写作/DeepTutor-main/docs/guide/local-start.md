# Local Start

DeepTutor now has a single local onboarding flow.

## Recommended flow

1. Clone the repository.
2. Activate a Python 3.11+ environment.
3. Install from source:

```bash
pip install -e .
pip install ".[server]"
pip install ".[math-animator]"   # optional
```

4. Run:

```bash
python scripts/start_tour.py
```

The tour runs directly in the terminal and asks whether you want `Web` or `CLI` guidance.

Both flows end by writing a clean project-root `.env`. Runtime behavior config is loaded only from:

- `data/user/settings/main.yaml`
- `data/user/settings/agents.yaml`
- `data/user/settings/interface.json`

## What the tour does

- chooses the frontend and backend ports DeepTutor will keep using
- installs the selected dependency profile
- collects LLM, embedding, and optional search settings
- lets you test LLM and embedding before finalizing
- writes `.env` and clears any stale temporary `data/user/settings/env.json`

## Start the app after onboarding

```bash
python scripts/start_web.py
```

This launcher reads the project-root `.env`, starts the FastAPI backend, and injects `NEXT_PUBLIC_API_BASE` into the frontend process automatically.

## CLI-only usage

If you only want the terminal workflow, choose `CLI` inside `start_tour.py`. After that, common commands remain:

```bash
deeptutor chat
deeptutor kb list
deeptutor serve --port 8001
```

## Notes

- `config/` is no longer used at runtime.
- `web/.env.local` is no longer part of the supported local setup flow.
- TTS settings are no longer part of the official configuration surface.
