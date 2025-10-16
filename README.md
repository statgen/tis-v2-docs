# TOPMed Imputation Server Docs (Nextflow Edition)

New TOPMed Imputation Server documentation for the Nextflow edition (Cloudgene 3 / Imputationserver 2).

Following on the steps of [genepi/michigan-imputationserver](https://github.com/genepi/michigan-imputationserver), we moved the docs to a consolidated [MkDocs](https://www.mkdocs.org/) system. Prettier and ad-free!

## Tooling

This project uses `uv` for dependency management. To install dependencies:
```sh
uv sync
```

To run the docs server locally:
```sh
uv run mkdocs server
```

To deploy the docs, run:
```sh
uv run mkdocs gh-deploy
```
