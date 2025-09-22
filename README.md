# InvenTree project ADR

InvenTree project wide docs and tools.
This covers both ADRs for [InvenTree/InvenTree](https://github.com/inventree/) as well as other systems in the [InvenTree org](https://github.com/inventree/).

## Tech stack

- pre-commit (config and in CI) to ensure common formatting and spellchecking
- uv for managing environments, dependencies and packaging
- sphinx and furo for rendering the docs - very similar to mkdocs-material

## Structure

- `.github` - contains the CI/CD configs
- `docs/adr` - contains the ADRs for the InvenTree project

## Local development

Ensure uv is installed:

```bash
pip install uv
```

Then run the following command to create a virtual environment and install the dependencies:

```bash
uv sync
```

Local versions of the docs can be built using:

```bash
uv run  sphinx-build -M html docs build
```
