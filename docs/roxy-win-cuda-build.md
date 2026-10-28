# Roxy Windows CUDA build (llama.cpp v0.8.0)

This is an **on-demand, fork-local workflow** that builds a matched Windows CUDA package from official upstream **v0.8.0** source. Nothing installs into Windows or any production server. It does not modify Roxy Control, Roxy Workbench, or Open WebUI.

Workflow: [.github/workflows/roxy-windows-cuda-v080.yml](../.github/workflows/roxy-windows-cuda-v080.yml)

## Versions and hardware

| Component | Pinned value |
| --- | --- |
| Source | ggml-org/llama.cpp tag v0.8.0 |
| Runner | GitHub-hosted windows-2022 |
| CUDA toolkit / runtime | 12.4 (on ephemeral GitHub runner only) |
| GPU compute capability | 8.6 / sm_86 (RTX 3080 Ti, RTX 3090, RTX 3060/3070) |
| CMake | Ninja Multi-Config, Release, CPU and CUDA backends |
| Programs | llama-server.exe, llama-cli.exe, plus dependent libraries |
| Distribution | One matched ZIP with NVIDIA runtime DLLs, manifest, SHA-256 checksums |

**Important:** GitHub-hosted Windows runners do not have an appropriate NVIDIA inference GPU. A successful CI build verifies compilation, packaging, DLL presence, and a startup/version check. It does **not** prove that inference, vision or GPU offload work on the actual computer.

## Build profiles

- **test** (default): Builds and embeds llama.cpp's browser chat UI.
- **production**: Does not build, download or embed browser UI assets. The HTTP inference API remains functional.

The --no-ui server argument also disables a present UI at runtime.

Each workflow invocation builds just **one** profile, keeping outputs unambiguous.

## How to run

1. Once this workflow is merged into the fork's **default branch**, open the repository's **Actions** tab.
2. Select **Roxy | llama.cpp v0.8.0 | Windows CUDA 12.4**.
3. Click **Run workflow**, choose **test** initially, then launch the build.
4. Download the named workflow artifact. Inside it is the **portable ZIP** and a matching checksum text file.
5. Extract the portable ZIP to a **new, isolated directory**, keeping executables and DLLs together.
6. Run llama-server.exe --version, then the usual model and vision tests.
7. Only after validation, run the workflow again with **production** selected.

**GitHub Actions requires manually triggered workflows to exist on the default branch.** You can review the files on a development branch, but merge before expecting the Run workflow button.

This workflow does not create a GitHub Release, change the default branch, install software locally or modify production paths.

## Quality and isolation checks

The workflow fails when:
- the checked-out official source is not at the required tag;
- the test UI fails to build;
- CMake or compilation fails;
- essential EXEs, backend DLLs, or CUDA runtime DLLs are missing;
- llama-server.exe --version fails to start; or
- package creation/upload fails.

BUILD-MANIFEST.json records the source commit SHA, build profile, CUDA version, architecture and workflow run ID. SHA256SUMS.txt records file hashes. The downloaded artifact also includes a checksum for the portable ZIP.

**The target PC does not need a second CUDA toolkit installation.** It does need a compatible NVIDIA driver. Do not mix binaries or DLLs from unrelated llama.cpp releases.

## Upgrade strategy

The first workflow is intentionally pinned to **v0.8.0** for reproducibility. After a working test build, we can add a stable-release selector, run acceptance tests and create validated releases. There is **no automatic installation or replacement of Roxy's production executables**.

## Expected artifact names

- roxy-llama-v0.8.0-win-x64-cuda12.4-sm86-test
- roxy-llama-v0.8.0-win-x64-cuda12.4-sm86-production

Each includes the portable ZIP and its SHA256 companion. No PATH, registry, Windows service, driver or application setting changes are made on the target computer.
