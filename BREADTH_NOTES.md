# Build notes - Drug Discovery & Cheminformatics

Repo2RLEnv pr_diff (v0.9.6), Harbor-format output. Domain: drug-discovery.

## Funnel

| Stage | Count |
|---|---|
| Domain candidates (deduped) | 498 |
| After active+star+noise filter | 279 |
| Final target list (cap 200) | 200 |
| Repos with >=1 task | 152 |
| Repos with 0 tasks | 48 |
| Total tasks (<=3/repo) | 320 |

## Generation

- Shared task bank: each repo generated once (pr_diff, limit=3,
  max_files_per_pr=15, min_loc_changed=2), 4-way concurrency,
  rate-limit-aware with .done checkpoint resume.
- Per-domain consolidation: rank by reward_calibration.loc_changed desc,
  keep at most 3 tasks per repo. Zero-yield repos are NOT fabricated.
- Overlap with the 3 published cancer editions is allowed (user decision);
  domains are independent selections, not a partition of one repo set.

## Zero-yield repos (informational, 48)

- china-testing/python-api-tesing
- svaksha/Julia.jl
- wu-yc/LabClaw
- LeonChaoX/qinyan-academic-skills
- paperclipai/companies
- AgentTeam-TaichuAI/ScienceClaw
- nwchemgit/nwchem
- xjtulyc/MedgeClaw
- NVIDIA-BioNeMo/bionemo-agent-toolkit
- mordred-descriptor/mordred
- oddt/oddt
- MinkaiXu/GeoDiff
- K-Dense-AI/claude-skills-mcp
- vijaymasand/PyChem-Pro
- SpectrAI-Initiative/InnoClaw
- atomicarchitects/equiformer_v2
- OpenFreeEnergy/openfe
- weisscharlesj/SciCompforChemists
- dengjianyuan/Survey_AI_Drug_Discovery
- DeepWism/DeepWism-miRNA
- china-testing/python_cn_resouce
- schwallergroup/ai4chem_course
- atomicarchitects/equiformer
- MinkaiXu/GeoLDM
- Acellera/htmd
- Intelligent-Drug-Discovery-Lab/SurfDock
- isayev/ASE_ANI
- westpa/westpa
- AnshMittal1811/MachineLearning-AI
- zjunlp/MolGen
- pcko1/Deep-Drug-Coder
- qinheming/BIoClaw
- HFooladi/GNNs-For-Chemists
- theislab/chemCPA
- Shen-Lab/DeepAffinity
- blazerye/DrugAssist
- genular/pandora
- mcs07/CIRpy
- THUNLP-MT/dyMEAN
- jvalegre/aqme
- mims-harvard/PDGrapher
- xianglin226/Benchmarking-Single-Cell-Perturbation
- atomicarchitects/equiformer_v3
- LIYUESEN/druggpt
- cansyl/DEEPScreen
- jssweller/DrugHIVE
- benstaf/ChemGAN-challenge
- mooreneural/lacuna

## Caveats

- Static validation only; real grading needs an LLM judge key + Docker.
- inline_dockerfile mode skips image build (no Docker daemon needed to publish).
- Provider secrets in upstream diffs were redacted before public release.
