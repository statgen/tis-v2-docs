# API Overview

## It's a REST API

The TOPMed Imputation Server exposes a REST API you can use directly to do tasks such as:

- Submit a new imputation job.
- See your current and past jobs.
- Get detailed information about a specific job.

You send HTTP requests to our API endpoints using any tool of your liking (CURL, Python with requests, the imputationbot...), and we respond with a JSON object.

The base URL for all calls to the API is:

```
BASE_URL = https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/
```

So if we say an endpoint is `/jobs`, the full URL you need to call is `https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs`.

With the exception of file download links, all endpoints return JSON payloads.

## Token-Based Authentication

The TOPMed Imputation Server uses bearer tokens for authentication. This means the server gives you a secret string, and you must submit that string with every query to prove your identity.

!!! warning "Keep your token safe!"
    Anyone who has your token can impersonate you when communicating with the server, so ensure you protect it as well as you would protect a password!

The token can be created and downloaded from your user profile (`<username> -> Profile`):

![Activate API](https://raw.githubusercontent.com/genepi/imputationserver-docker/master/images/api.png)

Some security measures are in place to protect your account:

- Tokens are valid for 30 days. Once your token expires, you'll need to create a new one.
- You can only view the token once, when you request it. Make sure to copy it!
- You can only have one active token at a time.

You can check the status of your token in the web interface. You can revoke your token and get a new one at any time.

## Accessing from the CLI: `curl` and `jq`

The full API specification (with examples using `curl` and Python) can be found in [the API reference page](api-reference.md). Here we summarize some basic examples using `curl` and `jq`.

### Listing Your Jobs

[curl](https://curl.se/) is an ubiquitous command-line utility used for interacting with REST APIs (among other things). To submit an HTTP GET request, you just write:

```sh
curl <url>
```

You can use the `-H` flag to set a request header. If you have saved your access token in an environment variable called `TIS_TOKEN`, you can list all your jobs by calling `GET /jobs`:

```sh
curl \
    -H "X-Auth-Token: ${TIS_TOKEN}" \
    'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs'
```

This command will print compact JSON to the terminal, but it might be an unreadable wall of text. You can parse it with [jq](https://jqlang.org/):

```sh
curl \
    -H "X-Auth-Token: ${TIS_TOKEN}" \
    'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs' \
    | jq
```

Then you'll see some pretty-printed output along these lines:

```json
{
    "count": 32,
    "page": 3,
    "pageSize": 15,
    "data": [
        {
            "id": "job-20250829-110739-457",
            "name": "My Job!",
            "state": 7,
            "positionInQueue": -1,
            "submittedOn": 1756480059559,
            "startTime": 1756480059670,
            "endTime": 1756480937680,
            "deletedOn": 1757087819500,
            "steps": [],
            "outputParams": [],
        },
        ...
    ]
}
```

`jq` offers filtering options, so for example if you only want to know the status of your jobs you can write

```sh
curl \
    -H "X-Auth-Token: ${TIS_TOKEN}" \
    'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs' \
    | jq '.data[] | { id, state }'
```

It will print something along the lines of

```json
{
    "id": "job-20250829-110739-457",
    "state": 7,
}
{
    "id": "job-20250829-110620-968",
    "state": 7,
}
...
```

### Geting Information About One Job

You can get more information about a specific job by calling `GET /jobs/<job-id>`. Following the previous example:

```sh
curl \
    -H "X-Auth-Token: ${TIS_TOKEN}" \
    'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs/job-20251016-102620-166' \
    | jq
```

Might produce a response like this:

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
        (log data from each imputation step)
    ],
    "outputParams": [
        (information about downloadable output files)
    ],
}

```

### Submitting a Job

To submit a job, you call `POST /jobs/submit/imputationserver2` and set several parameters in the payload, including *the whole file(s) you're uploading for imputation*. To transfer such large data volumes, you need to use the `multipart/form-data` encoding. Luckily, `curl` makes it quite easy:

```sh
curl \
    -H "X-Auth-Token: ${TIS_TOKEN}" \
    -F "refpanel=topmed-r3" \
    -F "build=hg19" \
    -F "population=all" \
    -F "files=@chr19.vcf.gz" \
    -F "files=@chr20.vcf.gz" \
    -F "files=@chr21.vcf.gz" \
    'https://imputation.biodatacatalyst.nhlbi.nih.gov/api/v2/jobs/submit/imputationserver2'
```

The `-F` flag denotes a form field for `multipart/form-data`, and the ampersand `@` tells `curl` to load the whole file as a binary stream. Once you submit, you will receive a message like this:

```json
{
  "success": true,
  "message": "Your job was successfully added to the job queue.",
  "id": "job-20251016-224029-866"
}
```

## Notes on Format

* Timestamps are milliseconds since the Unix Epoch (January 1st, 1970). If a timestamp has not been set (for example, the deletion time for a still-valid job), it's returned as `-1`.
* The job state is an enumeration, see the [API reference](api-reference.md) for details.
* There are many additional fields we do not document here. They are not considered useful for an end-user.
