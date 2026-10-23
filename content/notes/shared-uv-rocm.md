---
title: uv
date: 2026-10-23
summary: my usage of uv and env
ai: true
---

# Shared uv cache and ROCm environment

## Current layout

- `UV_CACHE_DIR=/mnt/extra/uv/cache` and `UV_LINK_MODE=symlink` are set in `~/.config/zsh/dev.zsh`.
- The uv cache is on `/mnt/extra`, on the same filesystem as the shared environment.
- The shared environment is `/mnt/extra/venvs/rocm-py314`.
- It uses Python 3.14.8, PyTorch `2.12.0+rocm10.0.0`, HIP `7.15.26333`, and AMD Radeon 880M.
- `~/Projects/ncm/.venv` and `~/Projects/ARnDlm/.venv` are symbolic links to `/mnt/extra/venvs/rocm-py314`.
- `~/Projects/Archive/minimind/.venv` remains independent. It uses the same ROCm PyTorch version but Transformers `5.18.0`; the shared environment uses Transformers `4.57.0`.

## Package link modes

Most packages in the shared environment use symbolic links to package files in the uv cache. This avoids duplicate package payloads.

The ROCm binary stack uses hard links instead: `torch`, `triton`, and the AMD/ROCm runtime packages. Symlink installation made GPU operations fail with `device kernel image is invalid`. Hard links keep one copy of each file because the uv cache and shared environment reside on the same filesystem.

The project-level `.venv` entries are symbolic links to the shared environment. They are not hard links.

## Usage and cautions

Load the settings in an existing zsh session with:

```bash
source ~/.config/zsh/dev.zsh
```

Use the project environment path to run code:

```bash
cd ~/Projects/ncm
.venv/bin/python evaluate.py --config config/eval/sudoku-easy.yaml --checkpoint-dir artifacts/sudoku/easy --data-dir data/sudoku/easy --n 10 --gpus 0
```

NCM and ARnDlm share one environment. Installing, removing, or upgrading a package there affects both projects. Do not run separate project dependency syncs against this shared environment without reviewing the package changes.

Do not run `uv cache clean` on `/mnt/extra/uv/cache`. The symbolic links in the shared environment depend on cached package files. When reinstalling PyTorch or ROCm binary packages, pass `--link-mode hardlink`; symbolic links caused GPU kernel failures.

## Verification

```bash
uv cache dir
readlink ~/Projects/ncm/.venv
readlink ~/Projects/ARnDlm/.venv
.venv/bin/python -c 'import torch; print(torch.__version__, torch.version.hip, torch.cuda.get_device_name(0))'
```

The shared NCM environment passed a 10-example Sudoku easy evaluation: 10/10 solved, 100% cell accuracy, and 100% valid grids.
