# Product Specification

## Naming

| Component | Name |
|---|---|
| Project / system (this repository) | Khora |
| Hardware device | KhoraVault |
| Host service | `khorad` |
| Command-line client | `khora` |
| Mobile app | Khora app |

**Decision (2026-10-23):** renamed from `clawvault`. The old name tied the product to openclaw, while the product targets any agent or automated system. *Khôra* (Plato's "receptacle") describes a device that holds credentials without being their consumer.

**Why split the names:** the repository holds all three components (firmware, host service, mobile app), so the system needs a name distinct from the device. Bare `khora` collides with existing projects, so every installable or searchable artifact carries a qualified name (`khoravault`, `khorad`); bare `khora` names only the umbrella and the command-line client installed with `khorad`.

**Host service renamed (2026-10-23):** `khora-agent` became `khorad`, because "agent" already names the AI agents Khora serves. The `d` suffix follows the Unix convention for services (`sshd`, `dockerd`), and `khora` is its client, as `docker` is to `dockerd`.
