# cic-storage

> Ez nem klasszikus repo. Ez AI-operált domain schema layer.
> Emberi belépő: ez a README. AI belépő: `ai/ONBOARDING.md`.

A `cic-storage` a `cic-primitives` **meta-séma rétegére** épülő domain-repó —
storage objektumokat ír le, a `cic-primitives` atomic/aggregate primitíváinak
kompozíciójaként.

**Fontos architekturális elv: ez a repó szándékosan provider-agnosztikus.**
A `cic-storage` azt írja le, MIT akarunk (kívánt tárolási állapot — bucket,
tier, titkosítás, hozzáférés), nem azt, hogy egy konkrét felhő (OCI/AWS/Azure/
on-prem) hogyan valósítja meg. A tényleges létrehozást/felügyeletet külön
`cic-module-<provider>` modulok végzik a `cic:provider` WASM ABI-n keresztül
(pl. `cic-module-oracle-cloud` — jelenleg az egyetlen implemented modul).
Provider-specifikus részletek (path-ok, HTTP-igék, SDK-anomáliák) szándékosan
NEM jelennek meg itt — azok a modul saját correspondence/binding rétegében élnek.

---

## Két szint

| Szint | Mit képvisel | Hol van | Eredet |
|---|---|---|---|
| **atomic primitive** | 8 irreducibilis atom — Shape, Role, Behavior, Contract, Address, Identity, Event, Access | `schemas/atomic/` | öröklött a `cic-primitives`-ból |
| **aggregate primitive** | Kompozíció sealed/defaulted/required slot-okkal | `schemas/aggregate/` | öröklött a `cic-primitives`-ból |
| **domain composition** | `StorageBucket` — provider-agnosztikus object-storage intent | `schemas/examples/storage-bucket.yaml` | **ennek a repónak a saját munkája** |

A domain objektum mindig következmény, soha nem kiindulópont — ez itt a
`StorageBucket`, ami a `ManagedEntity` aggregate-et alkalmazza a `cic:storage`
domain-ra, provider-független szinten.

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
| `StorageBucket` domain composition | **defined** | `schemas/examples/storage-bucket.yaml` — provider-agnosztikus, teljes surface-készlet (Config/State/Operation/Notification/Policy/Binding) |
| Provider modul kötés | **partial** | `cic-module-oracle-cloud` = implemented; aws/azure/onprem modulok = concept (lásd composition `derivation_chain.provider_mapping`) |
| KubernetesPod sablon-példa | **öröklött, nem storage-specifikus** | `schemas/examples/kubernetes-pod.yaml` — a `cic-primitives` demója, nem ennek a repónak a munkája |
| Signed release pipeline | **defined** | Vault Transit + ECDSA, a pipeline maga lefutott (lásd git tag-ek) |
| Production trust-chain | **not implemented** | CIC-Relay + CIC-Schemas feladata |

---

## Kapcsolódó repók

| Repo | Kapcsolat |
|---|---|
| `cic-primitives` | közvetlen upstream — atomic/aggregate primitívák + tooling, `git remote base` |
| `base-repo` | közvetett upstream (a `cic-primitives` saját `base@0.5.0` merge-én keresztül) |
| `cic-module-oracle-cloud` | provider modul — a `StorageBucket` intent egyik konkrét megvalósítója, `cic:provider` WASM ABI-n keresztül |
| `CIC-Relay` | runtime — a provider modulokat futtatja, amik a `storage-bucket.yaml` kompozíciót reconcile-olják |

---

## Release artifact — GHCR

The schema release is available as an OCI artifact in GitHub Container Registry.

**With ORAS:**

```bash
oras pull ghcr.io/centralinfracore/schema/cic-storage:v0.1.2-src2026
```

**With curl (no ORAS required):**

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
