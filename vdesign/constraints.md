# Technical and Business Constraints

Security invariants and the threat model live in `docs/architecture_design.md`; credential handling rules in `docs/credential_model.md`.

## Operational & Architectural Constraints
- **Universal Agent Interface**: `khorad` exposes a language-agnostic local interface (Unix domain socket / named pipe / CLI) so any agent runtime can integrate without custom hardware drivers.
- **Minimal Runtime Dependencies**: `khorad` ships as a small, self-contained daemon. Target platforms are a pending decision in `plan.md`.
- **Single Source of Truth**: Protocol schemas and interface contracts defined in `docs/` or a shared contracts directory govern all three implementations (hardware firmware, host service, mobile app).
