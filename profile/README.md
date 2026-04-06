# VaultShare

**VaultShare** is a robust, lightweight file-sharing platform inspired by Google Drive, built as a modular multi-service architecture. It was originally developed as a solution for the SECJUR engineering challenge.

> [!IMPORTANT]
> **AI-Native Development**: This entire ecosystem was developed in approximately **10 hours** using cutting-edge **AI Coding Agents** (such as Google Antigravity and Claude Code). It serves as a benchmark for high-velocity engineering where architectural decisions, implementation, and testing are orchestrated by autonomous AI agents under human guidance.

Users can securely upload files, manage complex folder hierarchies, and share resources with granular, inherited permission levels (Viewer, Editor, Owner).

## Implementation Status

- [x] **Phase 1 — Infrastructure Foundation**: Docker Compose stack, PostgreSQL, Redis, Celery, structlog, linting from day one
- [x] **Phase 2 — Accounts & Storage Abstraction**: JWT auth, custom User model, `StorageBackend` ABC with `LocalDiskBackend` (full) and `S3Backend` (stub)
- [x] **Phase 3 — Core Files & Audit Trail**: Upload, download, soft delete, append-only `AuditLog` with model-level immutability, first acceptance tests
- [x] **Phase 4 — Sharing & Permissions**: File and folder sharing with three permission levels (Viewer, Editor, Owner), permission inheritance, share revocation
- [x] **Phase 5 — File Versioning**: Version listing and per-version download endpoints. Same-name upload in shared folders creates new versions for Editors.
- [ ] **Phase 6 — Folders & ZIP Download**: Folder CRUD, nested hierarchy navigation, streaming ZIP via `stream-zip`
- [ ] **Phase 7 — Public Links & Celery Cleanup**: Time-limited shareable links, expiry enforcement, scheduled cleanup task
- [ ] **Phase 8 — Frontend**: Vue 3 + TypeScript SPA (scaffolded, not yet implemented)
- [ ] **Phase 9 — Bonus & Polish**: Full-text search, storage quotas, real-time notifications, final AI code review

## 🛠 Tech Stack

- **Backend**: Django 6, Django REST Framework (Modular Monolith architecture)
- **Database**: PostgreSQL (main), Redis (caching and Celery broker)
- **Async Tasks**: Celery & Celery Beat (background processing, cleanup, scheduled tasks)
- **Infrastructure**: Docker & Docker Compose (orchestration), Makefile (automation)
- **Frontend**: Vue 3, TypeScript, Tailwind CSS (scaffolded)
- **Security**: JWT Authentication (symmetric signing), append-only Audit Logs

| Repository | Description | Status |
|---|---|---|
| [vaultshare-infra](https://github.com/secjur-vault-share/vaultshare-infra) | Docker Compose orchestration & Makefile — **start here** | Complete |
| [vaultshare-api](https://github.com/secjur-vault-share/vaultshare-api) | Django 6 + DRF backend (modular monolith) | Core features implemented |
| [vaultshare-frontend](https://github.com/secjur-vault-share/vaultshare-frontend) | Vue 3 + TypeScript frontend | Scaffolded only |
| [vaultshare-docs](https://github.com/secjur-vault-share/vaultshare-docs) | System architecture, design decisions, and implementation history | Complete |

## Quick Start

```bash
git clone https://github.com/secjur-vault-share/vaultshare-infra
cd vaultshare-infra
cp .env.example .env
make up
make seed
```

Open [http://localhost:3000](http://localhost:3000) and log in with `demo@vaultshare.io` / `demo1234`.

Run the acceptance suite:

```bash
make verify
```

## Architecture

```
vaultshare-infra/
  docker-compose.yml       # api · frontend · postgres · redis · celery · celery-beat

vaultshare-api/            # Django modular monolith (service layer pattern)
  apps/accounts/           # Custom User model, JWT auth (email-first)
  apps/storage/            # Files, folders, file versions, storage backend abstraction
  apps/sharing/            # FileShare, FolderShare, permission resolution
  apps/audit/              # Append-only audit log (immutable at model level)

vaultshare-frontend/       # Vue 3 · TypeScript · Vite (scaffold)
```

## 🤖 AI-Driven Development

This project serves as a case study for the **1:100 Engineering Ratio**—where a single human orchestrator and high-capability **AI Coding Agents** can deliver what typically takes a team weeks.

- **Tools**: Developed with [Google Antigravity](https://google.com/antigravity) and [Claude Code](https://claude.ai/code).
- **Process**: From zero to a working modular monolith with 90%+ test coverage in ~10 hours.
- **Verification**: Autonomous test-driven development (TDD) using a custom acceptance suite to ensure behavioral parity across releases.

- **Storage abstraction** — a `StorageBackend` ABC decouples business logic from the
  filesystem. `LocalDiskBackend` is fully implemented; `S3Backend` is a stub mapping
  each method to its `boto3` equivalent, demonstrating that the interface is genuinely
  swappable.
- **Append-only audit log** — `AuditLog.save()`, `.delete()`, and QuerySet `.update()`
  all raise at the model level. Immutability is architectural, not procedural.
- **Permission model** — three levels (Viewer, Editor, Owner) resolved via
  `PermissionService`. "Most permissive wins" when a user has both a direct file share
  and an inherited folder share. Matches Google Drive behaviour; documented trade-off.
- **Soft delete** — `deleted_at` timestamp only (no redundant `is_deleted` boolean).
  Partial unique constraint on `(owner, folder, name)` prevents race conditions on
  concurrent uploads.
- **Version-aware schema from day one** — `File` and `FileVersion` are separate models
  even before versioning endpoints are active, avoiding a mid-project schema migration.

See [DESIGN_DECISIONS.md](https://github.com/secjur-vault-share/vaultshare-docs/blob/main/DESIGN_DECISIONS.md) for the full reasoning trail.

## What I'd Do Next

- **Phase 6 — Folder tree & ZIP**: Implement `FolderService.walk_tree()` as a generator
  and stream ZIP responses via `stream-zip`. Refactor from adjacency list to materialized
  paths (`django-treebeard`) if folder depth > 10 becomes a performance bottleneck.
- **Phase 7 — Public links**: Add `PublicLink` model with `secrets.token_urlsafe(32)`,
  expiry enforcement, and a Celery Beat task for cleanup. Add
  `check_creator_permission` in `PublicLinkService.access` to handle cascading
  invalidation when a link creator's access is later revoked.
- **Phase 8 — Frontend**: Build the Vue 3 SPA with Pinia state management, Tailwind
  styling, and an Axios interceptor for JWT refresh. Cover: file browser, sharing modal,
  breadcrumb navigation, public link viewer.
- **Production hardening**: Replace `SECRET_KEY` with injected secret, add rate limiting,
  configure CORS for specific origins, set up CI/CD pipeline with `make check` as gate.
