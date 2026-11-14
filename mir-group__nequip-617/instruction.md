# Issue

**Title:** Freeze parameters of models loaded by the ASE and TorchSim integrations in eager mode

## Description

`_from_saved_model`, used by `NequIPCalculator` and `NequIPTorchSimCalc` to load checkpoints and packages in `eager` mode, puts the model in eval mode but leaves its parameters with `requires_grad=True`. Other inference paths (TorchScript, AOTInductor and LAMMPS ML-IAP) already freeze them with `prepare_model_for_compile`. With trainable parameters, autograd keeps the tensors needed for parameter gradients while computing forces, although inference never computes those gradients. This PR adds `model.requires_grad_(False)` after `model.eval()` to instead freeze the parameters, which significantly reduces memory demand.

Peak RSS in eager CPU inference on a rattled 216-atom NaCl cell; Apple M5 Pro, 6 threads, torch 2.10:

| Model | before | after |
|---|---|---|
| Allegro-OAM-M | 3.65 GiB | 2.99 GiB |
| Allegro-OAM-M, with einsum allegro | 2.37 GiB | 1.76 GiB |
| Allegro-OAM-L, with einsum allegro | 10.22 GiB | 6.12 GiB |
| NequIP-OAM-S | 0.52 GiB | 0.52 GiB |
| NequIP-OAM-M | 1.06 GiB | 0.98 GiB |

'einsum allegro' is the `Contracter` einsum change in  For Allegro, the saving is one `[edges, channels, i, j]` tensor per layer, kept for the `Contracter` weight gradient: 256 KiB per edge for Allegro-OAM-L, about 4 GiB here. 

NequIP models save less, since their large per-edge tensors are needed for the forces anyway, and only the radial MLP's hidden activations and some node-level tensors are dropped: peak mem is 3% (NequIP-OAM-M) and 5% (NequIP-OAM-L) lower on 216 atoms, and 5.5% and 6.5% lower on a 1,000-atom cell:
| Model | before | after |
|---|---|---|
| NequIP-OAM-M | 2.55 GiB | 2.41 GiB (−5.5%) |
| NequIP-OAM-L | 6.19 GiB | 5.79 GiB (−6.5%) |

Tested:
- Energy, forces, stress and second derivatives are identical frozen vs unfrozen for Allegro-OAM-S, NequIP-OAM-S and freshly built models.

## Types of changes
<!-- What types of changes does your code introduce? Put an `x` in all the boxes that apply: -->
- [x] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds or improves functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Documentation improvement (updates to user guides, docstrings, or developer docs)

## Checklist:
<!-- Put an `x` in all the boxes that apply. If you're unsure about any of
     these, don't hesitate to ask. We're here to help! -->
- [x] I have read ["Contributing to NequIP"](../docs/dev/contributing.md).
- [x] My code follows the code style of this project and has been formatted using `black`.
- [x] All new and existing tests passed, including on GPU (if relevant).
- [ ] I have added tests that cover my changes (if relevant).
- [x] I have updated `CHANGELOG.md`.
- [ ] I have updated the documentation (if relevant).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `9f8c38d0f18c`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.