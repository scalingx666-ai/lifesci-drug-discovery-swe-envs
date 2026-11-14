# Issue

**Title:** local_web 新增 3D 蛋白结构查看器

## Description

## 主要变更

  ### 🧬 3D 蛋白结构查看器（新增）
  - 内置 3Dmol 渲染器，本地托管（`npm install 3dmol` + `/vendor/3Dmol-min.js`），国内网络可用
  - 点击聊天里 agent 返回的 `.pdb` / `.cif` / `.mmcif` / `.mol2` / `.sdf` / `.xyz` → 右侧抽屉直接 3D 展示
  - 支持 6 种风格（卡通 / 棒状 / 球状 / 线条 / 球棍 / 表面）× 5 种配色（彩虹 / 按链 / B 因子 / 残基 / 单色）
  - 自旋 / 深色背景 / 重置 / 截图 / 链残基原子统计；中英文工具条实时切换

  ### 🐛 其他修复
  - system prompt限制编造PDB，下载失败必须明确告知
  - 文件 URL 大小写不敏感回退（`1crn.pdb` 与 `1CRN.pdb` 都能打开）
  - 工作区布局优化，预览体验更舒服

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `0373cee30324`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.