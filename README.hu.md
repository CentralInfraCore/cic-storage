# cic-primitives

> Ez nem klasszikus repo. Ez AI-operált primitive schema layer.
> Emberi belépő: ez a README. AI belépő: `ai/ONBOARDING.md`.

A CIC **meta-séma rétege** — az a szint, amelyből minden domain objektum
(switch interface, kubernetes pod, service, database, policy) schema-szinten levezethető.

Nem domain modell. Nem IaC tool. Nem YANG leíró.

---

## Két szint

| Szint | Mit képvisel | Hol van |
|---|---|---|
| **atomic primitive** | 8 irreducibilis atom — Shape, Role, Behavior, Contract, Address, Identity, Event, Access | `schemas/atomic/` |
| **aggregate primitive** | Kompozíció sealed/defaulted/required slot-okkal | `schemas/aggregate/` |

A domain objektum mindig következmény, soha nem kiindulópont.

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
| 8 atomic primitive YAML | **defined** | Shape · Role · Behavior · Contract · Address · Identity · Event · Access |
| 5 aggregate primitive YAML | **defined** | ManagedEntity, ConfigSurface, StateSurface, OperationSurface, PolicySurface |
| Primitive meta-schema validáció | **defined** | `schemas/index.yaml` + `compiler.py` — `make validate` zöld |
| sealed/required slot enforcement | **defined** | domain specializáció kompatibilitás ellenőrzött |
| KubernetesPod domain példa | **defined** | `schemas/examples/kubernetes-pod.yaml` |
| Signed release pipeline | **defined** | Vault Transit + ECDSA, `primitives/@v0.1.3` kiadva |
| defaulted slot merge szemantika | **draft** | replace/deep_merge/append/union — D-008, első domain override-nál dől el |
| build provenance | **draft** | `build_hash == source_hash` — külön build lépés még nincs |
| LifecycleSurface / CapabilitySurface / NotificationSurface | **concept** | Relay execution modell előfeltétel |
| ExecutionSurface aggregate | **concept** | D-009, Relay modell előfeltétel |
| Schema/API/runtime kódgenerálás | **not implemented** | a `semantic_mapping` mezők irányt adnak, de generator nincs |
| Teljes szemantikai típusellenőrzés | **not implemented** | a jelenlegi validator formai, nem szemantikai |
| Production trust-chain | **not implemented** | CIC-Relay + CIC-Schemas feladata |

---

## Kapcsolódó repók

| Repo | Kapcsolat |
|---|---|
| `base-repo` | upstream tooling (Makefile, CI, compiler) — `git merge base@0.5.0` |
| `CIC-Relay` | runtime — primitívekből épülő sémákat futtatja |
| domain repók | leszármazottak — `cic-primitives` a base-jük |

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
