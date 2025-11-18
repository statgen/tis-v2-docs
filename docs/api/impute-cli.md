# The `impute` Command-Line Utility

The [Imputation Server Command-Line utility](https://github.com/statgen/tis-v2-cli) `impute` allows you to interact via CLI with imputation servers such as:

* [TOPMed Imputation Server](https://imputation.biodatacatalyst.nhlbi.nih.gov)
* [Michigan Imputation Server](https://imputationserver.sph.umich.edu)

It supports any imputation server running [Cloudgene 3](https://www.cloudgene.io/) and [Imputation Server 2](https://github.com/genepi/imputationserver2).

`impute` is maintained by the TOPMed Imputation Server team. See the [Contact Page](/docs/contact.md) for contact information.

## Examples

List all your jobs in the TOPMed Imputation Server:
```sh
./impute job list topmed
```

Get a specific job by its ID in the Michigan Imputation Server:
```sh
./impute job get michigan <job-ID>
```

Register a server called `mcps`:
```sh
./impute server register mcps <url>
```

Submit a job to the newly registered server:
```sh
./impute job submit mcps \
    --name 'My test!' \
    --refpanel topmed-r3 \
    --population all \
    --build hg19 \
    --file data/chr20.vcf.gz \
    --file data/chr21.vcf.gz
```

## Installation

1. Install [the `uv` Python package manager](https://docs.astral.sh/uv/)
2. Clone [the GitHub repo](https://github.com/statgen/tis-v2-cli).
3. Navigate to the project root and run:
```sh
uv sync
```

And you're good to go! You can see all available commands by running:
```sh
./impute --help
```

If you don't want to use `uv`, see the `pyproject.toml` file for Python dependencies.

## Access Tokens

`impute` expects the existence of a local folder `data/` containing tokens stored in plaintext, in a file named `<server>.token` (for example, `data/topmed.token`). If no such file is found, you will be interactively asked for a token.

**Please use caution when storing and using access tokens! Anyone using your token can impersonate you in the server.**

## CLI

`./impute` is the primary API interaction script. It has a lot of options, so using `--help` to find what you need is recommended.

Many commands require you to specify which server they apply to (e.g., `topmed` or `michigan`). By default, the script expects a text file `data/<server>.token` (e.g., `data/michigan.token`) containing a valid access token for the specified server; a different token file can be specified by passing `--token-file <path-to-token>`

* `./impute version` prints the utility's version.
* `./impute server` contains subcommands for managing available servers.
    * `./impute server show (name)` lists complete information about all registered servers (if `name` is skipped), or about the selected server (if `name` is provided).
    * `./impute server register <name> <url>` adds a server registry entry mapping the provided `name` (must be unique) to the provided `url`. The server is queried for basic information.
* `./impute job` contains subcommands for interacting with your jobs in a specific server.
    * `./impute job list <server>` lists all your jobs in the selected server, past and present.
    * `./impute job get <server> <job-id>` gives detailed information about a single job.
    * `./impute job submit <server> <params...>` submits a job with the provided parameters (some mandatory, some optional; check defaults!)
    * `./impute job cancel <server> <job-id>` cancels the selected job.
    * `./impute job restart <server> <job-id>` re-runs the selected job from scratch.
    * `./impute job download <server> <job-id>` downloads all files for the given job.

## Servers

By default, `impute` provides access to two servers:

* `topmed` connects you to the [TOPMed Imputation Server](https://imputation.biodatacatalyst.nhlbi.nih.gov)
* `michigan` connects you to the [Michigan Imputation Server](https://imputationserver.sph.umich.edu)

If no token file is found in the `data/` folder, when you try to communicate with a server you will be interactively asked to input an access token. You can get your token from the server website: press on your name, then "Profile", then "Create API Token".

You can register other servers like this:
```sh
./impute server register <name> <url>
```

You can list all your available servers like this:
```sh
./impute server show
```
