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
