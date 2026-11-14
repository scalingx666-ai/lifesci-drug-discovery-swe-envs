# Issue

**Title:** Add opt-in KOKKOS acceleration for LAMMPS

## Description

LAMMPS builds that provide KOKKOS/CUDA without the legacy GPU package cannot currently use RadonPy's GPU execution path. This change adds an explicit KOKKOS option while keeping the existing CPU and legacy GPU paths as the default.

## Changes

- Add KOKKOS package/backend detection and optional `kokkos` arguments to `LAMMPS.exec()` and `LAMMPS.run()`.
- Allow presets to opt in through `RadonPy_LAMMPS_KOKKOS=on`, or direct callers through `LAMMPS(kokkos="on")`. The default is `off`; `auto` is also available when explicitly selected.
- Generate `-k on ... -sf kk -pk kokkos newton on neigh half gpu/aware off`. GPU-aware MPI is not required.
- Preserve the existing `mpi=-1` and `omp=-1` automatic-count conventions. For KOKKOS builds without OpenMP, automatic threading uses a single thread; explicit requests for unsupported backends fail clearly.
- Use CPU force evaluation for SP energy-tally reruns and centroid heat-flux calculations that are incompatible with the KOKKOS force path. Each fallback is logged.

The change is limited to `radonpy/sim/lammps.py`, `radonpy/sim/preset/sp.py`, and `radonpy/sim/preset/tc.py`. Existing force-field parameters, timesteps, thermal-conductivity input expressions, MPI launcher settings, and AutoMD scripts are unchanged. Documentation, test files, and site-specific scripts are not included.

## Usage

With a suitable KOKKOS/CUDA-enabled LAMMPS executable, set this before importing RadonPy:

```bash
export RadonPy_LAMMPS_KOKKOS=on
export LAMMPS_EXEC=/path/to/lmp
```

Continue to pass `gpu=1` to the preset's execution method. Direct `LAMMPS` callers can instead specify `kokkos="on"` on the constructor or execution method. Leaving the option unset retains the previous accelerator selection, even when the LAMMPS executable includes KOKKOS.

## Validation

- 95 local tests passed: the existing non-engine tests and additional compatibility checks kept outside this PR.
- Default launch commands and OpenMP thread settings match the unmodified develop version across 56 combinations of package availability and execution options.
- Verified automatic MPI/OpenMP counts, the existing `const.mpi_cmd` override, legacy `exec()` subclass signatures, KOKKOS backend detection, and CPU fallback selection.
- Thermal-conductivity input generation matches develop when the random seed is held fixed.
- No LAMMPS executable is currently available in the local validation environment, so native MD tests and GPU tests were not rerun for this final revision. The preceding implementation was tested on NVIDIA GB200; those results do not replace runtime validation of this revision.

---

## 日本語

## 概要

本変更では、従来のCPU計算およびGPUパッケージによる実行挙動をデフォルトとして維持しつつ、明示的な指定によってKOKKOSを利用可能にします。

### 変更内容

* **KOKKOS対応と引数の追加**:
* KOKKOSパッケージおよびCUDA/OpenMPのビルド・対応状況を検出し、`LAMMPS.exec()`および`LAMMPS.run()`にオプション引数`kokkos`を追加しました。

* **有効化の方法**:
* プリセット実行時は環境変数`RadonPy_LAMMPS_KOKKOS=on`、クラス直接利用時は`LAMMPS(kokkos="on")`などで有効化できます（デフォルト: `off`、`auto`の明示指定も可能）。

* **起動コマンドの制御**:
* KOKKOS有効時は、LAMMPS起動引数に`-k on ... -sf kk -pk kokkos newton on neigh half gpu/aware off`を付与します（CUDA-aware MPIは必須としません）。

* **プロセス・スレッド設定とバリデーション**:
* `mpi=-1`および`omp=-1`によるプロセス数・スレッド数の自動設定を維持します。OpenMP非対応のKOKKOS環境では自動設定時に1スレッドを割り当てます。また、非対応の実行方式が明示指定された場合は理由を明示したエラーを送出します。

* **非対応計算でのフォールバック**:
* KOKKOSの力計算と互換性のない「溶解度パラメータ（SP）用のエネルギー再集計計算」および「重心応力を用いた熱流束計算」では、自動的にCPUの力計算へと切り替え、その旨をログに出力します。

**変更対象ファイル**:
`radonpy/sim/lammps.py` / `radonpy/sim/preset/sp.py` / `radonpy/sim/preset/tc.py` の3ファイルです。
※既存の力場パラメータ、時間刻み、熱伝導計算の入力式、MPI起動設定、AutoMDスクリプトに変更はありません。ドキュメント、テストコード、計算機固有スクリプトは本PRには含めていません。

### 使用方法

KOKKOS/CUDA対応のLAMMPSバイナリを用意し、RadonPyのインポート前に環境変数を設定します。

```bash
export RadonPy_LAMMPS_KOKKOS=on
export LAMMPS_EXEC=/path/to/lmp

```

* **プリセット利用時**: 従来どおり`gpu=1`を指定して実行します。
* **直接呼び出し時**: コンストラクタまたは実行メソッドに`kokkos="on"`を指定します。
* **未指定時**: LAMMPSバイナリにKOKKOSが含まれている場合でも、従来の実行方式（CPUまたはGPUパッケージ）が維持されます。

### 検証内容

* **ローカルテスト**: MDエンジンを起動しない単体テストおよび互換性確認を含め、計95件のテスト通過を確認しました。
* **既存挙動の互換性**: パッケージの有無と実行オプションの組み合わせ（計56通り）において、デフォルトの起動コマンドおよびOpenMPスレッド設定が変更前の`develop`ブランチと完全に一致することを確認しました。
* **機能検証**: MPI/OpenMP数の自動設定、既存の`const.mpi_cmd`の上書き挙動、従来型引数を持つサブクラスの`exec()`呼び出し、KOKKOS実行方式の検出、CPUへのフォールバック動作を確認しました。
* **入力生成の一致**: 乱数シード固定時、熱伝導計算の入力ファイル生成結果が`develop`ブランチと同一であることを確認しました。

---

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `d0d12c8207ba`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.