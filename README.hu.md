# cic-storage

> Ez nem klasszikus repo. Ez AI-operált domain schema layer.
> Emberi belépő: ez a README. AI belépő: `ai/ONBOARDING.md`.

A `cic-storage` a `cic-primitives` **meta-séma rétegére** épülő domain-repó — storage
objektumokat lenne hivatva leírni, a `cic-primitives` atomic/aggregate
primitíváinak kompozíciójaként.

**Egyelőre nincs storage-specifikus domain composition megírva** — a
`schemas/examples/` alatt csak a `cic-primitives` sablon-demója
(`kubernetes-pod.yaml`) van, ami nem storage-specifikus.

---

## Két szint

| Szint | Mit képvisel | Hol van | Eredet |
|---|---|---|---|
| **atomic primitive** | 8 irreducibilis atom — Shape, Role, Behavior, Contract, Address, Identity, Event, Access | `schemas/atomic/` | öröklött a `cic-primitives`-ból |
| **aggregate primitive** | Kompozíció sealed/defaulted/required slot-okkal | `schemas/aggregate/` | öröklött a `cic-primitives`-ból |
| **domain composition** | konkrét storage objektum | — | **még nincs megírva** |

A domain objektum mindig következmény, soha nem kiindulópont — ennek a
repónak egyelőre nincs saját domain-kompozíciója, amiből ez következne.

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
| Storage-specifikus domain composition | **NOT IMPLEMENTED** | egyetlen saját domain composition sincs még megírva |
| KubernetesPod sablon-példa | **öröklött, nem storage-specifikus** | `schemas/examples/kubernetes-pod.yaml` — a `cic-primitives` demója |
| Signed release pipeline | **defined** | Vault Transit + ECDSA, a pipeline maga lefutott (lásd git tag-ek), de storage-specifikus tartalom nélkül |
| Production trust-chain | **not implemented** | CIC-Relay + CIC-Schemas feladata |

---

## Kapcsolódó repók

| Repo | Kapcsolat |
|---|---|
| `cic-primitives` | közvetlen upstream — atomic/aggregate primitívák + tooling, `git remote base` |
| `base-repo` | közvetett upstream (a `cic-primitives` saját `base@0.5.0` merge-én keresztül) |
| `CIC-Relay` | runtime — (még nincs mit futtatnia ebből a repóból) |

---

## Release artifact — GHCR

A séma release OCI artifactként érhető el a GitHub Container Registry-ben.

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
