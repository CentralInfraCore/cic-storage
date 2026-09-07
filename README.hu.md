# cic-storage

> Ez nem klasszikus repo. Ez AI-operált domain schema layer.
> Emberi belépő: ez a README. AI belépő: `ai/ONBOARDING.md`.

A `cic-storage` a `cic-primitives` **meta-séma rétegére** épülő domain-repó —
storage objektumokat ír le, a `cic-primitives` atomic/aggregate primitíváinak
kompozíciójaként.

> **⚠ Ágak közötti eltérés (2026-09-07 óta ismert, még nem oldott meg):**
> ez a README a **`devel`** ág állapotát írja le. A `devel`-en **nincs**
> ténylegesen egyeztetett, kanonikus storage domain composition — az alábbi
> `StorageBucket` egy kísérleti, provider-agnosztikus tervezési vázlat, amit
> egy lezárt (nem mergelt) PR kísért, és amit egyeztetni kell egy MÁR LÉTEZŐ,
> valódi, korábban elkészült munkával: `StorageResource` + `StorageAdapter`,
> ami a `storage/main` és `storage/releases/v0.1.0` ágakon él (GHCR-en is
> publikálva `v0.1.2-src2026` néven — lásd lent), de **sosem lett mergelve
> a `devel`-re**. Ezt a repót korábban (PR #2, `625eba7`) tévesen "nincs
> storage-specifikus domain compositionja" állapotúnak dokumentáltuk — ez a
> `devel` ágra igaz volt, de a repó egészére nem. Lásd `ai/DECISIONS.md` D-015/D-016.

---

## Két szint

| Szint | Mit képvisel | Hol van | Eredet |
|---|---|---|---|
| **atomic primitive** | 8 irreducibilis atom — Shape, Role, Behavior, Contract, Address, Identity, Event, Access | `schemas/atomic/` | öröklött a `cic-primitives`-ból |
| **aggregate primitive** | Kompozíció sealed/defaulted/required slot-okkal | `schemas/aggregate/` | öröklött a `cic-primitives`-ból |
| **domain composition (`devel`, kísérleti)** | `StorageBucket` — provider-agnosztikus object-storage intent tervezési vázlat | `schemas/examples/storage-bucket.yaml` | ennek a repónak a `devel` ágán készült, PR lezárva, NEM végleges |
| **domain composition (valódi, más ágon)** | `StorageResource` + `StorageAdapter` — platform-agnosztikus block volume (hypervisor/SAN/cloud) | `storage/main`, `storage/releases/v0.1.0` (origin) | korábbi, kész munka — **nincs a `devel`-en** |

A domain objektum mindig következmény, soha nem kiindulópont. Jelenleg KÉT
külön tervezet létezik erre a domainre, más-más ágon, egymással nem
egyeztetve — ez a repó jelenlegi legfontosabb nyitott kérdése.

---

## Gyors start

```bash
make validate    # séma validáció — ha ez nem zöld, semmi sem kész
make release     # signed artifact (Vault szükséges)
```

---

## AI belépési pontok

| Fájl | Mire való |
|---|---|
| `ai/ONBOARDING.md` | Boot protokoll — minden session elején |
| `ai/MAINTENANCE_CONTRACT.md` | Mit szabad, mit nem, mikor kell döntés |
| `ai/SYSTEM_CONTEXT.md` | Teljes architekturális kontextus |
| `ai/PROMPTMAP.yaml` | Task queue — mi a következő konkrét lépés |
| `ai/DECISIONS.md` | Döntési history — miért úgy van ahogy van |

---

## Aktuális állapot

| Réteg | Státusz | Megjegyzés |
|---|---|---|
| Örökölt atomic/aggregate primitívák | **defined** | `schemas/atomic/`, `schemas/aggregate/` — a `cic-primitives`-ból, `base` remote-on át |
| `StorageBucket` domain composition (`devel`) | **draft, nem egyeztetett** | `schemas/examples/storage-bucket.yaml` — kísérleti, provider-agnosztikus vázlat, PR #3 lezárva mergelés nélkül |
| `StorageResource`+`StorageAdapter` (valódi, más ágon) | **defined, de nincs a `devel`-en** | `storage/main` / `storage/releases/v0.1.0` — block volume, hypervisor/SAN/cloud, capability-alapú konformancia, GHCR-en publikálva |
| KubernetesPod sablon-példa | **öröklött, nem storage-specifikus** | `schemas/examples/kubernetes-pod.yaml` — a `cic-primitives` demója, nem ennek a repónak a munkája |
| Signed release pipeline | **defined** | Vault Transit + ECDSA, a pipeline maga lefutott (lásd git tag-ek) |
| Production trust-chain | **not implemented** | CIC-Relay + CIC-Schemas feladata |

---

## Kapcsolódó repók

| Repo | Kapcsolat |
|---|---|
| `cic-primitives` | közvetlen upstream — atomic/aggregate primitívák + tooling, `git remote base` |
| `base-repo` | közvetett upstream (a `cic-primitives` saját `base@0.5.0` merge-én keresztül) |
| `cic-compute` | cross-domain referencia célpontja — `StorageResource.attached_to → cic:compute:ComputeResource` |
| `cic-module-oracle-cloud` | provider modul — egy lehetséges konkrét megvalósítója akár a `StorageBucket`, akár a `StorageResource` intentnek, `cic:provider` WASM ABI-n keresztül |
| `CIC-Relay` | runtime — a provider modulokat/adaptereket futtatja, amik a storage compositiont reconcile-olják |

---

## Release artifact — GHCR

A séma release OCI artifactként érhető el a GitHub Container Registry-ben.
**Ez a `StorageResource`+`StorageAdapter` release** (a `storage/releases/v0.1.0`
vonalból) — nem a `devel` ágon lévő `StorageBucket` vázlat, aminek még nincs
saját GHCR release-e.

**ORAS-szal:**

```bash
oras pull ghcr.io/centralinfracore/schema/cic-storage:v0.1.2-src2026
```

**curl-lel (ORAS nélkül):**

```bash
REPO="centralinfracore/schema/cic-storage"; TAG="v0.1.2-src2026"; \
TOKEN=$(curl -fsSL "https://ghcr.io/token?scope=repository:${REPO}:pull" | jq -r .token); \
DIGEST=$(curl -fsSL \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Accept: application/vnd.oci.image.manifest.v1+json" \
  "https://ghcr.io/v2/${REPO}/manifests/${TAG}" | jq -r '.layers[0].digest'); \
curl -fL -H "Authorization: Bearer ${TOKEN}" \
  "https://ghcr.io/v2/${REPO}/blobs/${DIGEST}" \
  -o cic-storage-v0.1.2.yaml
```
