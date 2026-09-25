# Develop Notes

## Current Status

### Feature Development

Aim at **Stage 1**: implement a backend adapter for `desilike`, and support local PNG inference with single tracer.

Features in plan:

- [ ] *multi-tracer* support (by desilike)
- [ ] support for *other backends* (native, cobaya, etc.)
- [ ] support for *other PNG types* (equilateral, orthogonal, cosmological collider, etc.)
- [ ] support for joint inference with *other cosmological parameters*
- [ ] support for joint inference with likelihoods from *other datasets* (CMB, LSS, etc.)

### AI 协作

与 ChatGPT 有以下对话：
1. #sleep repo design: repo structure, public API, backend adapter, etc. 内容沉淀入 [docs/design_notes](./design_notes.md)
2. #active desilike 调研及 backend 设计： desilike implement 已整理入文档 [experiments/desilike_lpng/notes](./desilike_lpng/notes.md), stage1 路线图待整理入文档 [docs/dev_notes](./dev_notes.md)
