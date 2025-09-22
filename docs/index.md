
# InvenTree Project ADR Documentation

These pages contain general documentation regarding the direction and (technical) governance of the InvenTree project. They target the general project, not only InvenTree/InvenTree (which contains the backend for the inventory management system).

InvenTree consists of many components, working together to enable inventory management and releated tasks, with a focus on electronics (manufacturing). Decisions regarding the architecture of the project are documented as {term}`ADR` in the [docs/adr](adr/index.md) directory.

```{toctree}
:caption: Table of Contents
:maxdepth: 2
self
adr/index
security.md
```

## Ecosystem

Due to its plugin interface, InvenTree is being extended by multiple parties.
These pages govern all components created, maintained and releases by the InvenTree organization. Other actors in the ecosystem are **encouraged** to participate in the creation and lifecycle-ing of {term}`ADR`s, but these pages do not govern them.

There is **no penalty** mechanism for not following these guidelines in principle. Non-compliant components might however, not work and might not receive attention.

## Glossary

```{glossary}
ADR
 Architectural Decision Record
```
