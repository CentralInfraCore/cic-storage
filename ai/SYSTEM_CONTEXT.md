# System Context — cic-storage (AI számára)

Olvasd el mielőtt bármit módosítasz.

---

## Mi ez a rendszer?

A `cic-storage` a CentralInfraCore **storage domain schema repo**.
Egységes `StorageResource` ManagedEntity specializáció — block volume lifecycle.

**Scope döntés (D-001):**
- ✓ Block volume: hypervisor disk, SAN/iSCSI LUN, cloud block storage
- ✗ Object storage (S3/GCS/Blob) — service réteg, fogyasztod nem menedzseled
- ✗ NFS/NAS — service fogyasztás, a compute mountolja
- ✗ Snapshot mint önálló resource — a `StorageResource`-on végzett művelet

---

## Kompozíciós lánc

```
base-repo
  └─[remote: base]─► cic-primitives (@v0.1.2)
                          └─[remote: base]─► cic-storage (ez a repo)
```

---

## StorageResource — tervezési alap

**Egységes séma, capability mechanizmussal (D-001):**

```
StorageResource
  identity:             kind=StorageResource, namespace=cic:storage
  config_surface:       size_gb, filesystem, access_mode, encryption, tags
  state_surface:        attach_state, attached_to, usage_bytes, health
  operation_surface:    attach, detach, resize, snapshot, delete
  notification_surface: attach-state-changed, threshold-exceeded, health-degraded
  binding_surface:      {backend}/{provider}/{location}/{id}
```

**backend értékek:**
- `hypervisor` — hypervisor-managed disk (qcow2, raw, vmdk)
- `san` — SAN/iSCSI LUN (fizikai tárolórendszer)
- `cloud` — cloud block storage (EBS, Persistent Disk, Azure Disk)

**Capability példák:**
- `filesystem` — formázott filesystem (ext4, xfs, ntfs)
- `encryption` — at-rest encryption
- `snapshot` — pont-in-time snapshot művelet
- `resize_online` — online resize (leállítás nélkül)
- `multi_attach` — több compute node-hoz egyszerre csatolható

---

## Kapcsolat cic-compute-val

A `ComputeResource.disk_gb` csak a boot disk. Minden további volume
`StorageResource` — a compute csatolja (`attach` operation).

---

## Jelenlegi állapot (2026-05-06)

| Elem | Státusz |
|---|---|
| git bootstrap + primitives/@v0.1.2 merge | **defined** |
| project.yaml + dependency.yaml | **defined** |
| `schemas/domain/storage-resource.yaml` | **draft** |
| `schemas/adapters/storage-adapter.yaml` | **concept** |
| `make validate` zöld | **pending** |
| első signed release | **concept** |
