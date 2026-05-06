# LLM Rules — cic-storage

- Minden állításhoz jelöld meg: **defined** / **draft** / **concept**
- `schemas/atomic/` és `schemas/aggregate/` upstream — ne módosítsd
- Csak `schemas/domain/` és `schemas/adapters/` az írható terület
- Object storage, NFS/NAS, önálló snapshot resource: D-001 alapján TILTOTT
- Minden döntést rögzíts `ai/DECISIONS.md`-ben D-NNN formátumban
- `make validate` — ha nem zöld, javíts, ne kerüld meg
