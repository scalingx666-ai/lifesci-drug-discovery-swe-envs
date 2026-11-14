# Issue

**Title:** fix(training): always keep the newest checkpoint

## Description

## Problem

`fit` and `fit_from_iterator` build their orbax `CheckpointManager` with `best_fn` (the validation loss), `best_mode='min'` and `max_to_keep` (default 3). With these options orbax-checkpoint 0.12.4 uses `BestN` as its only preservation policy. It keeps the N lowest-loss checkpoints and has no rule that keeps the newest one, so a new checkpoint whose loss is not among the best N is deleted as soon as it is written.

Both restart paths read `latest_step()`, so a restart goes back to the newest checkpoint that survived. The TFDS path then starts at that checkpoint's next epoch, and the training after it is lost without a warning. A run whose validation loss has levelled off and is noisy hits this often.

Toy reproduction on main: the validation loss rises with every step, `max_to_keep=2`, eval every step, 4 steps per epoch.

| | `fit_from_iterator` | `fit` |
|---|---|---|
| after one epoch | `ckpt_1`, `ckpt_2` (`ckpt_4` deleted) | same |
| after a resume with one more epoch | still `ckpt_1`, `ckpt_2`; latest step 2 | same |

The resumed epoch trained 4 more steps, and none of them was kept.

## Fix

- `checkpoint_utils.make_training_checkpoint_options` builds the training options for both trainers. Its policy is `AnyPreservationPolicy([BestN(...), LatestN(1)])`: the same N best plus the newest. `max_to_keep` (default 3) sets N. Other `CheckpointManagerOptions` pass through; orbax itself rejects `keep_period` and `keep_time_interval` next to a preservation policy, and no caller in the repo passes them. The training options and the inference loader share one ranking (`_RANK_BY_LOSS`).
- Inference: the newest checkpoint may no longer be one of the best, so `load_params_from_checkpoint` now loads the checkpoint with the lowest saved validation loss (`best_step()`). When no checkpoint saved a loss (params-only checkpoints), it loads the latest one. Before, it loaded the newest of the N retained best, which is usually the best but not always. `load_model_from_workdir` had its own copy of the latest-step loader; its checkpoint branch now calls `load_params_from_checkpoint`.
- A new `step=` argument lets both restart paths load the latest step: the `fit()` restart, and the TFDS restart from a checkpoint without training state.

**Behaviour change:** for a training workdir without `params.pkl`, `eval`, `md` and fine-tuning now load the lowest-loss checkpoint instead of the newest of the best 3. Shipped models use `params.pkl` and are not affected.

## Tests

The tests use a toy model with patched training and validation steps, in which the validation loss rises with every step, so the newest checkpoint is never among the best:
- `test_checkpoint_retention.py::test_fit_keeps_and_resumes_from_the_newest_checkpoint`: `fit` keeps `[1, 2, 4]`, inference loads the lowest-loss step 1, and a restart continues from step 4 to step 8 (w = -0.8), not from step 2.
- `test_tfds_pipeline_logic.py::test_fit_from_iterator_restart_matches_uninterrupted_updates` now runs with `max_to_keep=1`, so for `fit_from_iterator` (both `eval_every` cases from #68) the restart must resume from the newest checkpoint, not the best, and still match an uninterrupted run bit for bit.
- `test_params_only_checkpoint_has_no_training_state` gains a second save without a loss; inference then loads the latest.

Runs, on CPU (MeluXina), with this branch merged into main after #68:
- `tests/mlff_tests/`: **204 passed**.
- Control, #68 without this PR: the `max_to_keep=1` restart test fails in both cases (`assert 1 == 4` and `assert 3 == 2`), because orbax pruned the newest checkpoint.
- Earlier run on the original commit: `test_cli_train`, `test_cli_finetune`, `test_finetuning_strategies`, `test_periodic_training`, `test_model_registry`, `test_legacy_model_autodetect` and `test_ci_lanes`: 183 passed, 3 skipped (slow, `--full` only).

## Not changed (follow-ups)

- A non-finite validation loss still skips the whole save, so the newest step after a NaN eval is not kept.
- With the two-stage loss, `best_step()` compares losses across the stage switch.
- `fit()` saves only the deployable params, so its restart is a warm start rather than a full resume.

Main (with #68, final TFDS checkpoint) is merged into this branch.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `48cac8eb4f48`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.