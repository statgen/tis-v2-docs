# API Reference

## Authentication

All HTTP calls to the API require authentication with a bearer token, sent in the request header `X-Auth-Token`. See the [API Overview](./api-overview.md) for details on obtaining a token.

The token you receive from the server is a JSON Web Token (JWT), as specified by the IETF standard [RFC 7519](https://tools.ietf.org/html/rfc7519). For a good introduction to the format, [see here](https://www.jwt.io/introduction#what-is-json-web-token).

## Data Models

### Job Info

This data type is returned by several API points. Depending on how much data is available, it can be a large, nested object.

```js
JobInfo = {
    id: str, // Unique ID for this job (form: "job-<timestamp>")
    name: str, // User-defined job name (optional; defaults to `id`)
    state: int, // See "State format" below
    positionInQueue: int, // -1 if not queueing, position in queue otherwise
    submittedOn: int, // Submission time (milliseconds since the Unix Epoch)
    startTime: int, // -1 if not started, otherwise start time (milliseconds since the Unix Epoch)
    endTime: int, // -1 if not done, otherwise end time (milliseconds since the Unix Epoch)
    deletedOn: int, // -1 if not deleted, otherwise deletion time (milliseconds since the Unix Epoch)
    steps: [StepInfo], // See StepInfo details below
    outputParams: [OutputInfo], // See OutputInfo details below
    ... // Additional fields
}
```

### State Format

The `state` field in `JobInfo` is an enumeration. These are its possible values:

```js
DEAD             = -1
WAITING          =  1
RUNNING          =  2
EXPORTING        =  3
SUCCESS          =  4
FAILED           =  5
CANCELED         =  6
RETIRED          =  7
SUCCESS_NOTIFIED =  8
FAIL_NOTIFIED    =  9
DELETED          = 10
```

### Step Logs

The `StepInfo` model corresponds to the `steps` list in `JobInfo`, and contains log information for each step in the imputation process (input validation, quality control, phasing, etc.)

```js
StepInfo = {
    name: str, // Which step is this?
    logMessages: [ // List of log messages produced by this step
        {
            message: str, // Log contents
            time: int, // When was this message produced? (milliseconds sine Unix Epoch)
            ... // Additional fields
        }
    ],
    ... // Additional fields
}
```

### Output Files

The `OutputInfo` model corresponds to the `outputParams` list in `JobInfo`, and contains grouped file information for the available downloads.

```js
OutputInfo = {
    name: str, // "output" or "cloudgene_logs"
    description: str, // Descriptive name of the output
    files: [ // List of specific file data
        {
            name: str, // File name
            size: str, // File size, as "<number> <units>"
            hash: str, // Required to get the download link
        }
    ],
    ... // Additional fields
}
```

## Endpoints

The following sections give detailed information and examples about the most common endpoints you will interact with.

Because the authentication token should be kept secure, in all the following examples we assume you haven't copy-pasted yours in a script. Instead, we assume you have set it as an enviornment variable called `TIS_TOKEN`.

### List Your Jobs

#### Request

`GET /jobs`

* URL parameters:
    * `page: int` (optional) &mdash; only retrieve the selected page (15 entries per page).

#### Response

```js
ListJobsResponse = {
    count: int, // Total number of jobs
    page: int, // Page number (1-indexed)
    pageSize: int, // Number of entries per page (15 if paginating, same as `count` otherwise)
    data: [JobInfo], // List of jobs (see Data Models)
    ... // Additional fields
}
```

#### Example

Request:

=== "CURL"
    ```sh
    $ curl \
        -H "X-Auth-Token: ${TIS_TOKEN}" \
        'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs?page=3'
    ```
=== "Python"
    ```python
    import os, json
    import requests

    BASE_URL = "https://imputationserver.sph.umich.edu/api/v2"
    AUTH_TOKEN = os.environ("TIS_TOKEN") # Reading from environment (don't store it in code!)

    # List the user's jobs
    response = requests.get(BASE_URL + "/jobs?page=3", headers={'X-Auth-Token' : AUTH_TOKEN })

    if response.ok:
        payload = response.json()
        print(json.dumps(payload, indent=4))
    ```

Response:

```json
{
    "count": 32,
    "page": 3,
    "pageSize": 15,
    "data": [
        {
            "id": "job-20250829-110739-457",
            "name": "My Job!",
            "state": 7, // RETIRED
            "positionInQueue": -1,
            "submittedOn": 1756480059559,
            "startTime": 1756480059670,
            "endTime": 1756480937680,
            "deletedOn": 1757087819500,
            "steps": [],
            "outputParams": [],
        },
        {
            "id": "job-20250829-110620-968",
            "name": "Another One Bites the Dust",
            "state": 7, // RETIRED
            "positionInQueue": -1,
            "submittedOn": 1756480059088,
            "startTime": 1756480059200,
            "endTime": 1756480979111,
            "deletedOn": 1757087819674,
            "steps": [],
            "outputParams": [],
        }
    ]
}
```

### Get Details for One Job

#### Request

`GET /jobs/<job-id>`



#### Response

`JobInfo` (see __Data Models__).

#### Example

Request:

=== "CURL"
    ```sh
    $ curl \
        -H "X-Auth-Token: ${TIS_TOKEN}" \
        'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs/job-20251016-102620-166'
    ```
=== "Python"
    ```python
    import os, json
    import requests

    BASE_URL = "https://imputationserver.sph.umich.edu/api/v2"
    AUTH_TOKEN = os.environ("TIS_TOKEN") # Reading from environment (don't store it in code!)
    JOB_ID = "job-20251016-102620-166"

    # Get details for job job-20251016-102620-166
    response = requests.get(BASE_URL + f"/jobs/{JOB_ID}", headers={'X-Auth-Token' : AUTH_TOKEN })

    if response.ok:
        payload = response.json()
        print(json.dumps(payload, indent=4))
    ```

Response:

```json
{
    "id": "job-20251016-102620-166",
    "name": "job-20251016-102620-166",
    "state": 4, // SUCCESS
    "positionInQueue": -1,
    "submittedOn": 1760624780293,
    "startTime": 1760624780394,
    "endTime": 1760626138133,
    "deletedOn": -1,
    "steps": [
        {
            "name": "Input Validation",
            "logMessages": [
                {
                    "message": "1 valid VCF file(s) found.\nSamples: 51\nChromosomes: 20\nSNPs: 7824\nChunks: 4\nDatatype: unphased\nBuild: hg19\nReference Panel: topmed-r3-prod (hg38)\nPopulation: all\nPhasing: eagle\nMode: imputation",
                    "time": 1760626138198,
                }
            ],
        },
        {
            "name": "Quality Control",
            "logMessages": [
                {
                    "message": "Uploaded data is hg19 and reference is hg38.",
                    "time": 1760626138212,
                },
                {
                    "message": "Lift Over",
                    "time": 1760626138216,
                },
                ...
            ],
        },
        ...
    ],
    "outputParams": [
        {
            "name": "output",
            "description": "Downloads",
            "files": [
                {
                    "name": "chr_20.zip",
                    "hash": "4acf1c84b465584f2c0c7cc4be74b5704eb1e93c3b5c088ef08b46f833b3ed14",
                    "size": "314 MB",
                },
                {
                    "name": "qc_report.txt",
                    "hash": "7207fe05a74a0686e568b5cb5d702bd28ee18cb40008a9a22658d78a2b3eeb19",
                    "size": "760 bytes",
                },
                ...
            ],
        },
        {
            "name": "cloudgene_logs",
            "description": "Logs",
            "files": [
                {
                    "name": "step1-nextflow.log",
                    "hash": "d9c28a3346d31f273dbb23305208bd8009cfa57d180bfac9d83139ed7ec69767",
                    "size": "54 KB",
                },
                ...
            ],
        }
    ],
}

```

### Submit a Job

#### Request

`POST /jobs/submit/imputationserver2`

This `POST` request needs to follow the `multipart/form-data` encoding to pass its inputs. It expects the following form parameters:

| Name       | Type                      | Required  | Default      | Description |
| ---------- | ------------------------- | --------- | ------------ | ----------- |
| files      | octet stream (input file) | mandatory | -            | VCF input file uploaded to the server. This argument can be repeated any number of times, once for each input file. |
| refpanel   | `topmed-r3`               | mandatory | -            | Reference panel (only TOPMed r3 available). |
| build      | `hg19` or `hg38`          | mandatory | -            | Build format of the input VCF files. |
| population | `off` or `all`            | mandatory | -            | Allele frequency check. |
| job-name   | string                    | optional  | (job ID)     | User-defined name for this job. |
| mode       | `imputation` or `qc_only` | optional  | `imputation` | Whether to only run QC or full imputation. |
| phasing    | `eagle` or `no_phasing`   | optional  | `eagle`      | Whether to phase with Eagle or skip phasing. |
| r2Filter   | float                     | optional  | `0`          | Equivalent to the "rsq Filter" option in the UI. |
| password   | string                    | optional  | (random)     | If provided, will be used as the password for all the ZIP files generated by this job. By default, a random secure password is generated and emailed to the user's address. |

#### Response

```js
SubmitJobResponse = {
    success: bool, // Was this request accepted for processing?
    message: str, // On failure: error reason. On success: generic message
    id: str, // The job ID for the newly submitted job
}
```

Since most input validation is done when the job gets processed, this call usually succeeds even when there are major issues such as missing form fields. Therefore, the only important aspect of the response is the returned job `id`.

#### Example

Request:

=== "CURL"
    ```sh
    $ curl \
        -H "X-Auth-Token: ${TIS_TOKEN}" \
        -F "refpanel=topmed-r3" \
        -F "build=hg19" \
        -F "population=all" \
        -F "files=@chr19.vcf.gz" \
        -F "files=@chr20.vcf.gz" \
        -F "files=@chr21.vcf.gz" \
        'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs/submit/imputationserver2'
    ```
=== "Python"
    ```python
    import os, json
    import requests

    BASE_URL = "https://imputationserver.sph.umich.edu/api/v2"
    AUTH_TOKEN = os.environ("TIS_TOKEN") # Reading from environment (don't store it in code!)

    url = BASE_URL + "/jobs"

    params = [
        # header        file            value                       mimetype
        ("refpanel"  , (None          , "topmed-r3")),
        ("build"     , (None          , "hg19"     )),
        ("population", (None          , "all"      )),
        ("files"     , ("chr19.vcf.gz", open("chr19.vcf.gz", "rb"), "application/octet-stream")),
        ("files"     , ("chr20.vcf.gz", open("chr20.vcf.gz", "rb"), "application/octet-stream")),
        ("files"     , ("chr21.vcf.gz", open("chr21.vcf.gz", "rb"), "application/octet-stream")),
    ]

    # Submit a job with VCF files chr19.vcf.gz, chr20.vcf.gz, chr21.vcf.gz
    response = requests.post(url, files=params)

    if response.ok:
        payload = response.json()
        print(json.dumps(payload, indent=4))
    ```

Response:

```json
{
  "success": true,
  "message": "Your job was successfully added to the job queue.",
  "id": "job-20251016-224029-866"
}
```

### Cancel a Job

#### Request

`GET /jobs/<job-id>/cancel`

#### Response

`JobInfo` (see __Data Models__).

#### Example

Request:

=== "CURL"
    ```sh
    $ curl \
        -H "X-Auth-Token: ${TIS_TOKEN}" \
        'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs/job-20251016-153844-544/cancel'
    ```
=== "Python"
    ```python
    import os, json
    import requests

    BASE_URL = "https://imputationserver.sph.umich.edu/api/v2"
    AUTH_TOKEN = os.environ("TIS_TOKEN") # Reading from environment (don't store it in code!)
    JOB_ID = "job-20251016-102620-166"

    # Get details for job job-20251016-102620-166
    response = requests.get(
        url=BASE_URL + f"/jobs/{JOB_ID}/cancel",
        headers={'X-Auth-Token' : AUTH_TOKEN },
    )

    if response.ok:
        payload = response.json()
        print(json.dumps(payload, indent=4))
    ```

Response:

```json
{
  "id": "job-20251016-153844-544",
  "name": "job-20251016-153844-544",
  "state": 6, // CANCELED
  "positionInQueue": 0,
  "submittedOn": 1760643524656,
  "startTime": 1760643524760,
  "endTime": 1760644443152,
  "deletedOn": -1,
  "steps": [
    {
      "name": "Input Validation",
      "logMessages": [
        {
          "message": "1 valid VCF file(s) found.\nSamples: 51\nChromosomes: 20\nSNPs: 7824\nChunks: 4\nDatatype: unphased\nBuild: hg19\nReference Panel: topmed-r3 (hg38)\nPopulation: all\nPhasing: eagle\nMode: imputation",
          "success": true,
        }
      ]
    },
    ...
  ],
  "outputParams": [
    {
      "name": "output",
      "description": "Downloads",
      "files": [
        {
          "name": "statistics/lift-over.txt",
          "hash": "2649d002420642a92575954678b91801b7a011900a848989d14611fc5251ff7a",
          "size": "0 bytes",
        },
        ...
      ],
    }
  ],
}
```

### Download a File

#### Request

`GET https://imputation.biodatacatalyst.nhlbi.nih.gov/share/results/<file-hash>/<file-name>`

Not under the same base as the other URLs!

You will need to retrieve the file hash and file name from `outputParams` as returned by `GET jobs/<job-id>`, see __Data Models__.

#### Response

This endpoint returns the file contents. Note that this can be a large binary blob! No JSON is produced.

#### Example

=== "CURL"
    ```sh
    $ curl \
        -H "X-Auth-Token: ${TIS_TOKEN}" \
        -L \
        'https://imputation.biodatacatalyst.nhlbi.nih.gov/share/results/<hash>/qc_report.txt' \
        > qc_report.txt
    ```
=== "Python"
    ```python
    import os, json
    import requests

    BASE_URL = "https://imputationserver.sph.umich.edu" # Note the shorter BASE_URL
    AUTH_TOKEN = os.environ("TIS_TOKEN") # Reading from environment (don't store it in code!)
    HASH = "..." # Put your file hash here (or better yet, write code that does that for you!)
    FILE = "qc_report.txt"

    url = BASE_URL + f"/share/results/{HASH}/{FILE}"
    headers = {'X-Auth-Token' : AUTH_TOKEN }

    # Use `stream=True` to iterate one chunk at a time, useful for big files!
    with requests.get(url, headers=headers, stream=True) as response:
        with open(FILE, "wb") as file:
            for chunk in response.iter_content(chunk_size=8192):
                file.write(chunk)
    ```

No response this time: the above call results in a local file `./qc_report.txt` containing the Quality Control report.
