# Product Specification

## Naming

| Component | Name |
|---|---|
| Project / system (this repository) | Khora |
| Hardware device | KhoraVault |
| Host service | `khora-agent` |
| Mobile app | Khora app |

**Decision (2026-10-23):** renamed from `clawvault`. The old name tied the product to openclaw, while the product targets any agent or automated system. *Khôra* (Plato's "receptacle") describes a device that holds credentials without being their consumer.

**Why split the names:** the repository holds all three components (firmware, host agent, mobile app), so the system needs a name distinct from the device. Bare `khora` collides with existing projects, so every installable or searchable artifact carries a qualified name (`khoravault`, `khora-agent`); `khora` is only the umbrella.
