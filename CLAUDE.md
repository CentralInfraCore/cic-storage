# CIC Storage — Claude kontextus

## Branch szabály — KÖTELEZŐ

**Érdemi fejlesztés kizárólag a `devel` ágon történhet.**

- `main` — csak merge fogad (devel → main), közvetlen commit tilos
- `storage/releases/v*` — kizárólag release tag célra
- `devel` — ez az aktív fejlesztési ág

Ha nem `devel`-en vagyunk: figyelmeztetés, és átváltás `devel`-re mielőtt bármilyen
schema, kód vagy dokumentáció változtatás történik.

## ⚠ Ágak közötti eltérés — KÖTELEZŐ elolvasni válasz előtt

A `devel`-en (ez a fájl is itt él) **nincs** egyeztetett, kanonikus storage
domain composition. `schemas/examples/storage-bucket.yaml` (`StorageBucket`)
egy kísérleti, provider-agnosztikus vázlat innen a `devel`-ről, lezárt (nem
mergelt) PR-rel — NEM tekinthető véglegesnek.

Eközben létezik egy **valódi, korábban elkészült munka** — `StorageResource`
+ `StorageAdapter` (block volume, hypervisor/SAN/cloud, capability-alapú
konformancia) — a `storage/main` és `storage/releases/v0.1.0` ágakon
(origin, GHCR-en publikálva `v0.1.2-src2026` néven), ami **sosem lett
mergelve a `devel`-re**. A PR #2 (`625eba7`) korábban tévesen "nincs
storage-specifikus domain composition" állapotot dokumentált — csak a
`devel`-re volt igaz, a repó egészére nem. Lásd `ai/DECISIONS.md` D-015/D-016.

Amíg ez nincs egyeztetve: ne állítsd, hogy a `StorageBucket` a repó
"a" domain compositionja — mondd ki mindkét tervezetet és az egyeztetetlen
állapotot.

## Mi ez a rendszer

A `cic-storage` egy **domain-repó** — a `cic-primitives` meta-séma rétegére épülve
storage objektumokat ír le, schema-szinten, **szándékosan provider-agnosztikus
módon**: a kívánt tárolási állapotot (bucket, tier, titkosítás, hozzáférés)
mondja ki, nem azt, hogy egy adott felhő (OCI/AWS/Azure/on-prem) hogyan
valósítja meg. A tényleges létrehozás/felügyelet külön `cic-module-<provider>`
modulok feladata a `cic:provider` WASM ABI-n keresztül.

A `schemas/atomic/`+`schemas/aggregate/` alatti fájlok **öröklöttek** a
`cic-primitives`-ból (a `base` remote-on át) — ez a repó nem definiálja őket.
A `devel` ág saját (nem végleges) munkája: `schemas/examples/storage-bucket.yaml`
(`StorageBucket`) — lásd fent a figyelmeztetést. A `kubernetes-pod.yaml` a
`cic-primitives` öröklött sablon-demója, nem ennek a repónak a munkája.

A primitívek azok az **irreducibilis szemantikai atomok és kompozícióik**, amelyekből
bármilyen menedzselt objektum strukturált, validálható, verziózott YAML sémává fordítható
— ezt a réteget a `cic-primitives` adja, nem ez a repó.

Részletes architektúra: `ai/SYSTEM_CONTEXT.md`
Következő konkrét feladatok: `ai/PROMPTMAP.yaml`
Tervezési döntések háttere: `ai/DECISIONS.md`

---

## Boot sequence — minden session elején

Mielőtt szakmai kérdésre válaszolsz, végezd el ezt a sorrendet:

1. `mcp__cic-graph__kb_status` — tudásbázis elérhető és friss?
2. Olvasd el: `ai/SYSTEM_CONTEXT.md`
3. Státusz térkép: mi **defined**, mi **draft**, mi **concept**
4. Bridge térkép: hol nincs még séma-szintű megfelelő a fogalomnak

Amíg ez a négy pont nincs meg, ne tegyél tényállításokat a primitive modell állapotáról.

---

## Háromszintű státusz — minden állításhoz kötelező

| Státusz | Jelentés |
|---|---|
| **defined** | YAML séma létezik, `make validate` zöld |
| **draft** | Design megvan írásban, séma még nincs |
| **concept** | Megbeszélt, de formálisan még nincs rögzítve |

## Scaffold térkép (aktuális)

| Elem | Státusz | Megjegyzés |
|---|---|---|
| git repo bootstrap | **defined** | `git merge base@0.5.0` a `cic-primitives`-on át (nem közvetlen) |
| `dependency.yaml` | **defined** | `base@0.5.0` composition lock (örökölt) |
| `project.yaml` | **defined** | `x-cic.repo_type: domain` |
| `schemas/` struktúra | **defined** | atomic/ + aggregate/ (örökölt) + examples/storage-bucket.yaml (`devel`, kísérleti) |
| atomic/aggregate réteg | **öröklött** | Shape, Role, Behavior, Contract, Address, Identity, Event, Access + surface-aggregate-ek |
| `StorageBucket` domain composition (`devel`) | **draft, nem egyeztetett** | provider-agnosztikus vázlat, `make validate` zöld, PR #3 lezárva mergelés nélkül |
| `StorageResource`+`StorageAdapter` (valódi, más ágon) | **defined, de nincs a `devel`-en** | `storage/main` / `storage/releases/v0.1.0`, GHCR-en publikálva |
| Provider modul kötés | **partial** | `cic-module-oracle-cloud` implemented; aws/azure/onprem concept |
| `make validate` zöld | **defined** | Docker-alapú tooling, Vault nélkül is fut |
| signed release pipeline | **defined** | lefutott (lásd git tag-ek) |

---

## A két szint

```
atomic primitive   = irreducibilis szemantikai atom
                     Shape · Role · Behavior · Contract · Address · Identity · Event · Access
                   → ezekből schema fragment generálható

aggregate primitive = szemantikai kompozíció sealed/defaulted/required slot-okkal
                   → ezek adják a használható tervezési egységeket
                   → aggregate-ből indulunk, nem atomból
```

Az objektum mindig következmény, soha nem kiindulópont.

---

## A kompozíciós mechanizmus

**Git remote = öröklődési lánc.** Nem YAML override rules.

```
base-repo (upstream sablon)
    │  remote: base → git merge base@0.5.0
    └──► cic-primitives
              │  remote: base → git merge base@0.5.0
              └──► cic-storage  (ez a repo)  ·  cic-network, cic-storage, stb. (testvér domain repók)
```

A fájlstruktúra IS az interface contract. A merge konfliktus = séma sértés.

---

## Bridge térkép — hol szakad meg a lánc

```
concept/Shape atom        ──?──  schemas/atomic/shape.yaml
concept/ManagedEntity     ──?──  schemas/aggregate/managed-entity.yaml
concept/git-composition   ──?──  git remote + base@0.5.0 merge
design/project.yaml       ──?──  compiler tooling (repo_type döntés)
```

Ha egy kérdés ilyen pontra mutat: ne mondd, hogy "nincs" — mondd, hogy
**"a fogalom documented, de a séma-szintű megfelelője még nem létezik"**.

---

## Graph-first reasoning (MCP)

MCP kérdéseknél ne `search_query → snippet → válasz` sorrendben dolgozz.

Helyette:
1. Fogalom azonosítás → induló node-ok (`search_nodes`, `find_nodes`)
2. 1–2 hop szomszédok (`neighbors`, `guided_path`)
3. Státusz ellenőrzés (defined/draft/concept)
4. Bridge ellenőrzés (van-e séma-fájl megfelelő)
5. Csak ebből válasz

---

## Reasoning mód

Válasz előtt azonosítsd:

- **immersion**: fogalmak, relációk, a primitive modell logikájának befogadása — ne javasolj implementációt
- **design**: séma struktúra, slot definíciók, kompozíciós szabályok tervezése
- **implementation**: konkrét YAML, séma fájl, Makefile változás

Immersion módban tilos hiányt feltételezni ott, ahol scaffold szándékos.

---

## Kapcsolódó repók

| Repo | Remote | Mit ad |
|---|---|---|
| `cic-primitives` | `base` | atomic/aggregate primitívák, tooling, signing hook, CI, Makefile, mk/infra.mk |
| `base-repo` | közvetett (a `cic-primitives` saját `base` remote-ja) | eredeti tooling-sablon |
| `cic-module-oracle-cloud` | — | provider modul, a `StorageBucket` intent egyik konkrét megvalósítója |
| `CIC-Relay` | — | a runtime, ami a provider modulokat futtatja a `storage-bucket.yaml` kompozíció ellen |

---

## Mérce

```bash
make validate          # séma validáció — ha ez nem zöld, semmi sem kész
make release VERSION=  # signed artifact
```

Lezárási kritérium minden primitive-re:
1. Ebből hogyan lesz séma (YAML)?
2. Ebből hogyan lesz API (RESTCONF / OpenAPI)?
3. Ebből hogyan lesz runtime viselkedés?

Ha mind a három megválaszolható → lezárt.
