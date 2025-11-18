# The `imputationbot` Command-Line Utility

The `imputationbot` is a CLI utility to interact with imputation servers running [Cloudgene 3](https://www.cloudgene.io/) and [Imputation Server 2](https://github.com/genepi/imputationserver2), such as the [TOPMed Imputation Server](https://imputation.biodatacatalyst.nhlbi.nih.gov) and the [Michigan Imputation Server](https://imputationserver.sph.umich.edu).

`imputationbot` is maintained by Lukas Forer. See the [Contact Page](/docs/contact.md) for contact information.

## Installation

1. [Install Java](https://adoptium.net/temurin) (8 or higher).
2. Install `curl`.
3. Navigate to the folder where you want to install the app.
4. Run the following command:
   ```sh
   curl -sL imputationbot.now.sh/v2 | bash
   ```
   (this command downloads an installation script and immediately runs it).
5. Check if the `imputationbot` got installed correctly:
   ```sh
   imputationbot version
   ```

## Using `imputationbot`

### Register a Server

You can register a server by calling
```sh
imputationbot add-instance
```

This will start an interactive process that will ask you for the server URL and an API token. You can get your token from the server website: press on your name, then "Profile", then "Create API Token".

### Submit a Job

You can submit a job by calling the `impute` subcommand. For example:
```sh
imputationbot impute --files /path/to/your.vcf.gz --refpanel 1000g-phase-3-v5 --population eur
```

Note that you don't explicitly select the server. `imputationbot` automatically selects the first registered server that contains a refpanel with the given name.

### Monitor Jobs

List all your jobs and their status:
```sh
imputationbot jobs
```

Get more details about one job:
```sh
imputationbot jobs <job-id>
```

### Download Results

To download all output files for a given job, run:
```sh
imputationbot download <job-id>
```

If the job is not done, `imputationbot` waits until the job is finished and then downloads the files.

You can provide the decryption password to decompress the downloaded ZIP files directly:
```sh
imputationbot download <job-id> --password <my-password>
```
