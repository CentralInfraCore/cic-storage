# Tervezési döntések — cic-storage

---

## D-001 — StorageResource: csak block volume, egységes séma (2026-05-06)

**Döntés:** A cic-storage scope-ja kizárólag block volume lifecycle. Object storage
(S3/GCS/Blob), NFS/NAS, és VolumeSnapshot mint önálló resource NEM tartozik ide.
Egyetlen `StorageResource` séma — a paradigma (hypervisor/san/cloud) az address és
adapter rétegben van, nem a config_surface-ben.

**Miért:**
- Object storage: service API endpoint, nem provisionálható infrastruktúra resource
- NFS/NAS: a compute fogyasztja (mount), maga a NAS nem CIC-menedzselt
- Snapshot: a `StorageResource`-on végzett művelet, nem önálló menedzselt objektum
- Egységes séma: compute D-010 és network D-001 tapasztalata — capability mechanizmus
  elegendő a paradigma szétválasztáshoz

**Következmény:** Nem lesz `block-volume.yaml`, `object-bucket.yaml`, `volume-snapshot.yaml`
külön. Egyetlen forrás: `schemas/domain/storage-resource.yaml`.
Snapshot az `operation_surface`-ben él, nem külön resource.

---

## D-002 — attach_state enum (2026-05-06)

**Döntés:** Az `attach_state` enum: `attached, detached, attaching, detaching, error, unknown`

**Miért:** A volume életciklusa attach/detach körül forog — tranziens állapotok
(`attaching`/`detaching`) valósak minden platformon.
- `attached`: volume csatolva, I/O lehetséges
- `detached`: szabad, csatolható
- `attaching`/`detaching`: tranziens, adapter normalizálja
- `error`: I/O hiba, adapter nem tudja kezelni
- `unknown`: adapter nem tudja meghatározni

---

## D-003 — address séma: backend/provider/location/id (2026-05-06)

**Döntés:** A StorageResource address konzisztens a többi domain-nel:
`{backend}/{provider}/{location}/{id}`

Példák:
- `hypervisor/proxmox-01/local-lvm/vm-100-disk-0`
- `san/netapp-01/aggr1/lun-42`
- `cloud/aws/eu-central-1/vol-0abc123`

**Miért:** Konzisztens addressing modell — a Relay routing logikája domain-agnosztikus.
