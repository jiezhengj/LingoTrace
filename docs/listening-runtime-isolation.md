# Listening Runtime Isolation

LingoTrace and ListenKit use separate Python 3.14 virtual environments. Homebrew Python 3.14 is only the bootstrap interpreter; normal transcription runs never install or upgrade packages.

## Ownership

| Project | Runtime | Direct dependencies | Responsibility |
| --- | --- | --- | --- |
| LingoTrace | `LingoTrace/.venv` symlink to `~/Library/Caches/LingoTrace/venvs/cpython-314` | `fugashi==1.5.2`, `unidic-lite==1.0.8` | Note rendering, tokenization, accent candidates |
| ListenKit | `ListenKit/.venv` | `faster-whisper==1.2.1` | Audio transcription and timestamped transcript artifacts |

The environments must not import each other's packages. LingoTrace must not import `faster_whisper`; ListenKit must not import `fugashi`.

## Boundary

The projects communicate only through:

- ListenKit CLI commands.
- Transcript JSON and Markdown artifacts.
- Slice manifests and export reports.

LingoTrace accepts transcript JSON with `schema_version: 1`. A payload without `schema_version` is treated as legacy v1. Explicit unknown versions are rejected before note generation.

## Initialization

Initialize each project explicitly from its own repository. Initialization is never triggered by a transcription command.

```bash
# LingoTrace repository
codex-skills/jp-listening-script-generator/scripts/init-listening-runtime.sh

# ListenKit repository
cli/init-faster-whisper.sh
```

The LingoTrace wrapper always uses `LingoTrace/.venv/bin/python`. Because the repository is stored under iCloud, `.venv` is a project-local symlink while the physical runtime stays in the local Cache directory. Loading the same `fugashi` native extension directly from the iCloud path was observed to hang for more than 100 seconds; the same extension loaded from a local path in about 0.4 seconds. `LINGOTRACE_LISTENING_PYTHON` is the preferred intentional override; `JP_LISTENING_PYTHON` remains a compatibility override. A missing or unhealthy runtime stops before transcription and prints the initialization command.

The external dictionary cache contains only static cross-version data such as `~/Library/Caches/jp-listening-dicts/accent_map.json`. Python packages belong in `LingoTrace/.venv`, not under the cache.

## Health Check

Run the read-only chain check from the LingoTrace repository:

```bash
codex-skills/jp-listening-script-generator/scripts/check-listening-chain.sh
```

It verifies:

- Both project runtimes use Python 3.14.
- LingoTrace loads the pinned dictionary packages and returns `公園⓪`, `散歩⓪`, and `し⓪` for the sample sentence.
- ListenKit imports `faster-whisper` within its bounded health check.
- Cross-project Python imports fail as required.
- ListenKit CLI entry points, JSON schema handling, `ffmpeg`, and `ffprobe` are available.

## Upgrades And Diagnosis

Change only direct dependency pins in each project's requirements file. Re-run that project's initializer, tests, and health checks before changing the other project. Do not copy `site-packages`, set cross-project `PYTHONPATH`, or install packages into the shared dictionary cache.

Package snapshots under `docs/` are diagnostic records of a verified environment. They are not installation inputs; requirements files remain the installation source of truth.

## Verification Record

On June 12, 2026:

- ListenKit's Python 3.14 runtime, `faster-whisper==1.2.1`, bounded import check, schema v1 producer/consumer behavior, and 71-test suite passed in PR #3.
- LingoTrace's 74-test suite passed after introducing the isolated-runtime contract.
- LingoTrace runtime installation and the `Unit3/attach/23.mp3` full-chain smoke test remain pending until the pinned dictionary packages finish installing into `LingoTrace/.venv`.
