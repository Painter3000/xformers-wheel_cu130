# xformers-wheel-cu130

Custom **xformers** Wheel für CUDA 13.0 + Blackwell GPU (sm_120).

Optimiert für **HuggingFace ZeroGPU** und **PSHuman** auf Blackwell Hardware.

---

## 🎯 Specs

| Komponente    | Version                      |
| ------------- | ---------------------------- |
| **Python**    | 3.10                         |
| **PyTorch**   | 2.11.0 + cu130               |
| **CUDA**      | 13.0                         |
| **GPU Arch**  | **12.0 (Blackwell sm_120)**  |
| **xformers**  | main (konfigurierbar)        |

---

## 🚀 Workflow

Der GitHub Actions Workflow (`build_xformers_cu130.yml`) baut das Wheel automatisch.

**Manuell triggern:**

1. GitHub → **Actions** → *Build xformers Wheel*
2. **Run workflow** → Optional: xformers ref eingeben (Branch / Tag / Commit)
3. **Run** → 30–60 Min. warten

**Release:** Nach erfolgreichem Build wird ein GitHub Release mit dem Wheel erstellt.

---

## 📦 Installation

### ⚠️ Wichtig: `--no-deps`

xformers muss mit `--no-deps` installiert werden, damit pip nicht automatisch eine andere torch-Version nachzieht und das cu130-Setup beschädigt:

```bash
pip install xformers-*.whl --no-deps
```

### In requirements.txt (nach dem torch-Block)

```
# xformers – cu130-Build, sm_120
https://huggingface.co/spaces/painter3000/PSHuman/resolve/main/Wheels/xformers-XXX-cp310-cp310-linux_x86_64.whl
```

> Wenn xformers korrekt installiert ist, verwendet PSHuman automatisch
> `xformers.ops.memory_efficient_attention` statt des bmm-Fallbacks in
> `attn_processors.py` – bessere Qualität und geringerer VRAM-Verbrauch.

---

## ⚙️ Build-Details

- ✅ `--no-build-isolation` → exakt torch 2.11.0, kein Downgrade
- ✅ `--index-url cu130` → CUDA-Wheel, nicht CPU
- ✅ `FORCE_CUDA=1` → Kompilierung auch ohne physische GPU
- ✅ `sm_120` → Blackwell RTX PRO 6000 unterstützt
- ✅ `CUDA_HOME=/usr/local/cuda` → nvcc wird zuverlässig gefunden
- ✅ `PATH` enthält `/usr/local/cuda/bin` → gilt für alle Build-Steps
- ✅ Submodule werden mitgezogen (Triton etc.)
- ✅ Verifikations-Step: Build schlägt fehl wenn Wheel `py3-none-any` ist
- ✅ Docker Image: `nvidia/cuda:13.0.1-devel-ubuntu22.04`

---

## 🔗 Verwendung in PSHuman

Nach Installation in `requirements.txt` aktiviert sich xformers automatisch
in `attn_processors.py`:

```python
# attn_processors.py – Hilfsfunktion
def _memory_efficient_attention(query, key, value, attn_bias=None):
    if xformers is not None:
        return xformers.ops.memory_efficient_attention(...)  # ← aktiv mit Wheel
    # bmm-Fallback wenn kein Wheel
```

---

## 🗂️ Verwandte Wheels (PSHuman Blackwell Stack)

| Paket | Repo |
| --- | --- |
| pytorch3d | [pytorch3d-Wheel](https://github.com/Painter3000/pytorch3d-Wheel) |
| torch_scatter | [pytorch3d-Wheel](https://github.com/Painter3000/pytorch3d-Wheel) |
| nvdiffrast | [pytorch3d-Wheel](https://github.com/Painter3000/pytorch3d-Wheel) |
| kaolin | [kaolin-Wheel](https://github.com/Painter3000/kaolin-Wheel) |
| **xformers** | dieses Repo |

---

## 📋 Vergleich: FaceLift vs. PSHuman xformers

| | FaceLift (cu128) | PSHuman (cu130) |
| --- | --- | --- |
| torch | 2.8.0+cu128 | 2.11.0+cu130 |
| CUDA | 12.8 | 13.0 |
| Docker | cuda:12.8.0-devel | cuda:13.0.1-devel |
| Repo | `calculate_splat_gemini` | `PSHuman` |
| Zweck | Zero123++ Multi-View | PSHuman Multi-View |

---

**Status:** ⏳ Build ausstehend
