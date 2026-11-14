# QX RAG

A Python project scaffold for a RAG application. The current app exposes a
FastAPI application and registers an Inngest function. Qdrant is being added
as the vector database; the initial storage helper in `vector_db.py` is still
in progress and is not ready to use yet.

## Requirements

- Python 3.12 (the version is recorded in `.python-version`)
- [uv](https://docs.astral.sh/uv/)
- Git

## Set up locally

From the project directory, install the locked dependencies:

```powershell
uv sync --locked
```

`uv` creates the `.venv` environment if needed. You can run commands through
that environment with `uv run`, or activate it in PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

The project dependencies include FastAPI, Uvicorn, Inngest, LlamaIndex,
Qdrant Client, OpenAI, Streamlit, and `python-dotenv`. The exact resolved
versions are recorded in `uv.lock`.

To add a dependency, use `uv add` with its package name (for example,
`uv add httpx`) rather than editing `pyproject.toml` by hand. Commit both
`pyproject.toml` and `uv.lock`.

## Run the API

Start the development server from the project root:

```powershell
uv run uvicorn main:app --reload
```

Uvicorn's app argument uses `module:attribute` format: `main:app` refers to
the `app` object in `main.py`. The interactive API documentation is available
at <http://127.0.0.1:8000/docs>.

The app loads environment variables from a root-level `.env` file. The file
is ignored by Git; do not commit credentials or other secrets.

## Run Qdrant locally

Install and start Docker Desktop, then create a persistent Docker volume and
start the Qdrant container:

```powershell
docker volume create qdrant_data
docker run -d --name qdrantRagQX -p 6333:6333 -p 6334:6334 -v qdrant_data:/qdrant/storage qdrant/qdrant
```

Port `6333` serves the HTTP API and dashboard; port `6334` serves gRPC. Check
that Qdrant is running and responding:

```powershell
docker ps --filter "name=qdrantRagQX"
Invoke-RestMethod http://localhost:6333
```

The local connection URL used by the storage helper is
`http://localhost:6333`. To stop and later restart the container:

```powershell
docker stop qdrantRagQX
docker start qdrantRagQX
```

The `qdrant_data` volume keeps Qdrant data when the container stops. If Docker
cannot download the image and reports a DNS or HTTPS proxy error, fix Docker
Desktop's network/proxy configuration and retry the `docker run` command.

## Git workflow

- `dev` is the integration branch for completed features and fixes.
- Create feature or bug-fix branches from `dev`, for example:

  ```powershell
  git switch dev
  git pull --ff-only origin dev
  git switch -c feature/short-description
  ```

- Commit and push your work on that branch:

  ```powershell
  git add main.py
  git commit -m "Describe the change"
  git push -u origin feature/short-description
  ```

  Replace `main.py` with the path of the file or files you changed.

- Open a pull request from the feature or fix branch into `dev`.
- Promote tested work by opening a pull request from `dev` into `main`.
- Keep `main` protected in the GitHub repository settings so changes go
  through pull requests rather than direct pushes.

The GitHub default branch controls which branch GitHub opens by default; it
does not, by itself, protect a branch.

## GitHub Actions

The workflow in `.github/workflows/ci.yml` runs on pushes to `dev` and `main`
and on pull requests targeting those branches. It installs the locked
dependencies with `uv sync --locked` and checks that the current dependencies
can be imported. Tests and deployment are not configured yet.