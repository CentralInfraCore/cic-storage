# AI Maintenance Contract — cic-storage

---

## Mit szabad

- `schemas/domain/storage-resource.yaml` módosítása, ha `make validate` zöld marad
- `schemas/adapters/storage-adapter.yaml` létrehozása/módosítása
- `ai/DECISIONS.md` bővítése új döntéssel (D-NNN formátum, dátummal)
- `ai/PROMPTMAP.yaml` státusz frissítése

## Mit nem szabad

- `schemas/atomic/` és `schemas/aggregate/` módosítása — upstream (cic-primitives)
- `schemas/index.yaml` módosítása — upstream (kivéve: AdapterContract kind hozzáadása)
- `tools/`, `mk/`, `Makefile` módosítása — upstream (base-repo via primitives)
- Object storage, NFS/NAS, önálló snapshot resource hozzáadása — D-001 tiltja
- `make validate` megkerülése

## Release folyamat

```bash
git checkout -b storage/releases/vX.Y.Z
export VAULT_ADDR="https://127.0.0.1:18200"
export VAULT_TOKEN=$(cat $XDG_RUNTIME_DIR/vault/sign-token)
export VAULT_SKIP_VERIFY=1
make release
git tag "storage/@vX.Y.Z"
git tag "cic-storage@X.Y.Z"
```

## Mérce

`make validate` — ha nem zöld, semmi sem kész.

Lezárási kritérium az adapter contract-ra:
1. Az address mező egyezik a storage-resource.yaml address kulcsával?
2. Az observe output mappal a state_surface-re?
3. Az operations capability listája szinkronban van a binding_surface-szel?
