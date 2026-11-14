# Issue

**Title:** Use `bmm` in `Contracter` to avoid materializing an `[edges, u, i, j, k]` tensor

## Description

Summary generated with the help of Claude!

1. Test fix:
There was an issue with the `device` parametrization in `tests/nn/test_contract_basic.py`: `["cpu"] + ["cuda"] if torch.cuda.is_available() else []` parses as `(["cpu"] + ["cuda"]) if ... else []`, so since 29585e4 (March 2025) all 248 of its tests, including the comparison against e3nn's `TensorProduct`, have been skipped on CPU-only runners. With the parentheses they run on CPU again (184 now pass on `develop`); GPU runners are unaffected.

2. `Contracter` acceleration:
`Contracter._contract_conv` (path-channel coupling, non-diagonal `w3j`) computes

```python
# zuij, uijk -> zuk
out = torch.sum(outer.unsqueeze(-1) * ww3j, (2, 3))
```

which materializes the full `[edges, u, i, j, k]` product before summing: 1 MiB per edge in float32 for `l_max = 3` (`i = j = k = 16`) with 64 channels. For Allegro-OAM-L on a 1,728-atom NaCl cell (135,108 edges) that is a 132 GiB allocation, which runs out of memory on an 80 GB A100. This PR replaces it with

```python
out = torch.einsum("zuij,uijk->zuk", outer, ww3j)
```

which does the same contraction as a permute + `bmm`, without the 5-D intermediate.

This code runs whenever the kernels (cuEq/Triton) don't: `Contracter` itself (CPU, or GPU without kernels), `CuEquivarianceContracter` off CUDA, and `TritonContracter` off CUDA/XPU or in training mode.

### Results

Allegro-OAM-{S,M,L} foundation models, `float32`, energy + forces per call / peak memory.

**GPU:** A100-SXM4-80GB, torch 2.11.0+cu130, TF32 off (S on an A100 MIG 3g.20gb slice); ms (median of 10 calls) / peak `torch.cuda.max_memory_allocated` in GiB:

| Model | Mode | Structure | original | einsum |
|---|---|---|---|---|
| S | eager | NaCl-1728 | 313 / 11.9 | 136 / 5.0 |
| S | AOTI | NaCl-1728 | 203 / 2.0 | 117 / 2.1 |
| M | eager | NaCl-64 | 31.7 / 0.8 | 35.4 / 0.5 |
| M | eager | NaCl-1728 | 368 / 19.4 | 193 / 10.9 |
| M | eager | Cu-1372 | 697 / 37.5 | 357 / 21.0 |
| M | AOTI | NaCl-64 | 15.1 / 0.2 | 11.6 / 0.2 |
| M | AOTI | NaCl-1728 | 256 / 4.4 | 174 / 4.4 |
| M | AOTI | Cu-1372 | 476 / 8.5 | 317 / 8.5 |
| L | eager | NaCl-64 | 138 / 7.1 | 43.0 / 2.6 |
| L | eager | NaCl-1728 | OOM (131.94 GiB alloc) | 902 / 66.6 |
| L | eager | Cu-1372 | OOM (179.53 GiB alloc) | OOM (~91 Gb?) |
| L | AOTI | NaCl-64 | 46.0 / 0.8 | 30.1 / 0.8 |
| L | AOTI | NaCl-1728 | 1109 / 19.2 | 677 / 19.6 |
| L | AOTI | Cu-1372 | 1512 / 26.0 | 924 / 26.7 |

**CPU:** Apple M5 Pro (64 GB), torch 2.10.0; ms (median of 20 calls) / peak RSS in GiB. edges at 64 / 216 atoms are 1,654 / 5,512 (S), 2,048 / 6,912 (M) and 5,004 / 16,784 (L):

| Model | Mode | Structure | original, 6 threads | einsum, 6 threads | original, 1 thread | einsum, 1 thread |
|---|---|---|---|---|---|---|
| S | eager | NaCl-64 | 104 / 1.2 | 61 / 0.8 | 246 | 92 |
| S | eager | NaCl-216 | 305 / 2.3 | 152 / 1.3 | 890 | 297 |
| M | eager | NaCl-64 | 266 / 1.7 | 155 / 1.2 | 641 | 249 |
| M | eager | NaCl-216 | 826 / 3.6 | 471 / 2.5 | 2348 | 837 |
| L | eager | NaCl-64 | 2224 / 8.4 | 614 / 3.5 | 6164 | 1252 |
| L | eager | NaCl-216 | not run (RAM) | 2361 / 10.2 | not run (RAM) | 5194 |
| S | AOTI | NaCl-64 | 58 / 0.7 | 36 / 0.6 | 177 | 60 |
| S | AOTI | NaCl-216 | 204 / 1.0 | 154 / 0.9 | 635 | 239 |
| M | AOTI | NaCl-64 | 178 / 0.9 | 141 / 0.9 | 427 | 189 |
| M | AOTI | NaCl-216 | 567 / 1.3 | 361 / 1.2 | 1380 | 651 |
| L | AOTI | NaCl-64 | 1832 / 1.5 | 583 / 1.5 | 4180 | 870 |
| L | AOTI | NaCl-216 | 6338 / 3.5 | 2232 / 3.7 | 13489 | 2942 |

- **Eager:** 1.9–3.2x faster on GPU with 1.6–2.7x less memory, and L now fits at 1,728 atoms (the 1,372-atom Cu cell still doesn't). On CPU 1.7–3.6x faster (2.6–4.9x on one thread) with 1.3–2.4x lower peak RSS.
- **AOTI:** inductor already fuses the product into the sum, so memory is about the same, but the einsum is still 1.3–1.7x faster on GPU and 1.3–3.1x on CPU (2.1–4.8x on one thread).
- On GPU the cuEquivariance contracter is still faster (AOTI, NaCl-1728: M 125 ms / 3.6 GiB, L 481 ms / 12.8 GiB), so this mostly helps where neither kernel is used: CPU, GPU inference without them, and GPU training without cuEquivariance

### Accuracy and tests

- Against the original: |ΔE| ≤ 8.2e-6 eV and max |ΔF| ≤ 4.8e-7 eV/Å on GPU (NaCl-64; eager, AOTI and TorchScript), and |ΔE| ≤ 7.0e-6 eV and max |ΔF| ≤ 1.2e-6 eV/Å for every original/einsum pair in the CPU table.
- On a single `Contracter` (`l_max` 2 and 3, one or many paths, float64, eager and TorchScript), outputs, gradients w.r.t. inputs and weights, and second derivatives (as used in force training) match the original to ≤ 5.3e-16 relative error, and `gradcheck`/`gradgradcheck` pass. Training wasn't run end to end; this second-derivative check is op-level.
- `pytest tests/` passes on CPU (macOS, torch 2.10.0), including the AOTInductor ASE tests.

## Additional Details from Claude
- **Two operands on purpose.** With three operands, e.g. `einsum("zui,zuj,uijk->zuk", x1, x2, ww3j)`, `torch.einsum` asks `opt_einsum` for a contraction path using the concrete sizes, which bakes the edge count into `make_fx` traces, so `nequip-compile`'s graph check fails. Checked with `nequip.utils.fx._nequip_make_fx` at 1,654 and 1,480 edges: the current sum and this einsum trace identically, while the three-operand einsum's traces differ in the hard-coded edge count (and match with `opt_einsum` disabled). It wouldn't save memory anyway, as it still forms the `[edges, u, i, j]` outer product first.
- The output is now a permuted (non-contiguous) view of the `bmm` result; everything downstream in allegro uses `reshape` or is stride-aware.
- The `w3j_is_ij_diagonal` branch has the same pattern but only materializes `[edges, u, i, k]`, with `k = 1` in the scalar-output last layer, so it's unchanged.

### Reduced precision

The contraction is now a matmul, so the opt-in reduced-precision modes for float32 models apply to it:
- **TF32** (A100): AOTI on NaCl-1728 goes from 174 to 94 ms (M) and from 677 to 286 ms (L), but the energy shift relative to TF32 off grows to 5e-3 eV (M) and 8e-3 eV (L) per 64 atoms (about 0.1 meV/atom), against 0.8–1.8e-3 eV for the original code.
- **`bf16-mixed`:** `bmm` is on autocast's lower-precision list (the old `mul`/`sum` were not), so this contraction now runs in bf16 and returns `bfloat16`. On a single `Contracter` (`l_max = 3`, CPU autocast), its output error against float64 is 1.3–1.6x larger (e.g. 2.9e-3 → 4.2e-3).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `8bd074c87645`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.