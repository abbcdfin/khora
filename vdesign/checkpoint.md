# Checkpoint

## Current State (2026-10-23)
- Renamed the project from `clawvault` to Khora (system) and KhoraVault (hardware device) in `README.org` and `docs/`; naming rationale in `docs/product_specification.md`.
- Renamed the GitHub repository to `abbcdfin/khora`; `origin` points at it and `main` is pushed.
- Initialized Vibe project structure (`vdesign/`, `docs/`) and drafted `docs/architecture_design.md`, including the threat model.
- Roadmap holds no accepted phase: all work is unnumbered candidates in `plan.md`, with a thin end-to-end slice proposed first.
- The two decisions blocking the thin slice were laid out with recommendations, awaiting the product owner's verdict:
  - Host agent language: Rust recommended (`zeroize` for secrets in memory; protocol code shareable with Rust firmware) over Go.
  - Agent IPC encoding: newline-delimited JSON-RPC 2.0 recommended (any runtime integrates with a socket and a JSON parser) over protobuf.

## Next Steps
- Rename the local directory `~/playground/khoravault` to `~/playground/khora`.
- Get the verdict on the two blocking decisions and record it in `docs/`.
- Hold the Decision Check Point on the thin slice; if accepted, number it as Phase 1 and open `active_plan.md`.
