# VaultShare

A lightweight file-sharing platform inspired by Google Drive.
Users can upload files, organize them into folders, share them with other users
at different permission levels, and generate time-limited public links.

## Repositories

| Repository | Description |
|---|---|
| [vaultshare-infra](https://github.com/secjur-vault-share/vaultshare-infra) | Docker Compose orchestration — **start here** |
| [vaultshare-api](https://github.com/secjur-vault-share/vaultshare-api) | Django 6 + DRF backend |
| [vaultshare-frontend](https://github.com/secjur-vault-share/vaultshare-frontend) | Vue 3 + TypeScript frontend |

## Quick Start

```bash
git clone https://github.com/secjur-vault-share/vaultshare-infra
cd vaultshare-infra
cp .env.example .env
make up
make seed
```

Open [http://localhost:3000](http://localhost:3000) and log in with `demo@vaultshare.io` / `demo1234`.

## Architecture

```
vaultshare-infra/
  docker-compose.yml   # api · frontend · postgres · redis · celery

vaultshare-api/        # Django modular monolith
  apps/accounts/       # User model, JWT auth
  apps/storage/        # Files, folders, versioning, storage abstraction
  apps/sharing/        # Permissions, shares, public links
  apps/audit/          # Append-only audit log

vaultshare-frontend/   # Vue 3 · TypeScript · Pinia · Tailwind
  pages/               # File browser, shared view, public link viewer
```

## Key Design Decisions

- **Storage abstraction** — a `StorageBackend` interface decouples business logic from
  the filesystem. Swapping to S3 or Azure Blob requires only a new backend class.
- **Append-only audit log** — `AuditLog` entries are immutable at the model level.
  No update or delete path exists, by design.
- **Permission model** — three levels (Viewer, Editor, Owner) resolved at the service
  layer, never in views. Folder permissions are inherited by their contents.
- **File versioning** — uploading a file with the same name into the same folder creates
  a new version rather than overwriting. Any previous version remains downloadable.
