# cic-storage — Claude kontextus

## Mi ez a rendszer

A `cic-storage` a CentralInfraCore **storage domain schema repo** — a cic-primitives leszármazottja.
Egységes `StorageResource` séma leírja a block volume életciklusát (create, attach, detach, resize,
delete) — hypervisor disk, SAN/iSCSI LUN és cloud block storage egyaránt.

**Scope:** csak block volume. Object storage (S3/GCS/Blob) és NFS/NAS service réteg — NEM ide tartozik.
Snapshot az erőforráson végzett művelet, nem önálló menedzselt objektum.

Részletes architektúra: `ai/SYSTEM_CONTEXT.md`
Tervezési döntések: `ai/DECISIONS.md`
Kötelező szabályok: `ai/MAINTENANCE_CONTRACT.md`

---

## Boot sequence — minden session elején

1. `mcp__cic-graph__kb_status` — KB elérhető és friss?
2. `ai/DECISIONS.md` — D-001 ismerete kötelező mielőtt bármit mondasz
3. `ai/SYSTEM_CONTEXT.md` — teljes storage domain kontextus
4. `ai/MAINTENANCE_CONTRACT.md` — mit szabad, mit nem

---

## Háromszintű státusz — minden állításhoz kötelező

| Státusz | Jelentés |
|---|---|
| **defined** | YAML séma létezik, `make validate` zöld |
| **draft** | Design megvan, séma még nincs |
| **concept** | Megbeszélt, formálisan nem rögzítve |

---

## Aktuális séma állapot

| Elem | Státusz | Megjegyzés |
|---|---|---|
| `storage-resource.yaml` | **draft** | unified block volume — tervezés alatt |
| `storage-adapter.yaml` | **concept** | egységes adapter contract |

---

## Kritikus döntés (D-001)

**D-001 — StorageResource: csak block volume, egységes séma**
Object storage, NFS/NAS, snapshot — service réteg, NEM ide tartozik.
A paradigma (hypervisor/san/cloud) az address és adapter rétegben van.
Address: `{backend}/{provider}/{location}/{id}`

---

## Kompozíciós lánc

```
base-repo
    └──► cic-primitives (primitives/@v0.1.2)
              └──► cic-storage (ez a repo)
```

---

## Mérce

```bash
make validate          # ha nem zöld, semmi sem kész
make release VERSION=  # signed artifact (Vault szükséges)
```
