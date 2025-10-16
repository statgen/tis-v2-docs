---
hide:
  - navigation
---

# Frequently Asked Questions (FAQ)

## What chromosomes are supported?

The TOPMed reference panel can be used to impute autosomes (1-22) and the X chromosome. See [here](imputation/reference-panel.md) for more details.

Please note that we do NOT support Y or MT chromosome processing.

## How many samples can I submit at once?

The Imputation Server only accepts jobs containing between 20 and 25,000 samples. If your job is too small (< 20 samples) or too large (> 25,000 samples), it will be rejected.

This limit exists to preserve quality of service for a wide audience. A workaround is to break large jobs into multiple chunks of 25,000 samples each. After completion, the results can be re-merged using [hds-util](https://github.com/statgen/hds-util) to combine the chunks and calculate the corrected R2.

If you have a use case that routinely requires smaller or larger jobs, please [contact us](contact.md) with details.

## I did not receive a password for my job, can you re-send it?

We can't.

The TOPMed Imputation Server creates a random password for each imputation job. For security reasons, this password is not stored on server-side at any time. If you didn't receive a password, please check the spam folder in your email. Please note that we are not able to re-send you the password. If you lose it, you will need to re-run your imputation job.

## I would like to impute WGS data, but the server says I have too many variants. What do I do?

The Imputation Server is meant to fill in the gaps between sites on a genotyping array (not WGS). Due to server resource requirements, imputing samples with many variants (more than 20,000 in any given 10Mb chunk) is not supported.

If you have WGS at 50x, then you would get little to no benefit from imputing on the server.

## How many jobs can I submit at once?

You can have up to three (3) jobs running at the same time. Please note that attempting to bypass this limit might result in a ban.

The TOPMed imputation server is a free resource, and usage limits allow us to provide service to a wide audience. Please do not attempt to bypass these limits by creating multiple accounts. We monitor usage, and reserve the right to terminate jobs or accounts that are in violation of our policies.

<!-- ## Unzip all files with a single command

You can unzip all result files at once using the command below:

`ls *.zip | xargs -P <nprocs> -n 1 unzip -P '<password>'`

-P <nprocs> specifies how many files to unzip in parallel (based on the number of CPU cores available).
-n 1 ensures that each unzip command processes one file at a time.
Replace <password> with the password provided for your result files. (Thanks to Charles-Alexandre for the suggestion) -->

## The unzip command is not working. What do I do?

Please check the following points:

1. When selecting AES256 encryption, please use [7-zip](https://www.7-zip.org/download.html) to unzip your files. For the default encryption option, all common `.zip` decompression programs should work.
2. If your password includes special characters (e.g. `\`), please put single or double quotes around the password when extracting it from the command line (e.g. `7z x -p"PASSWORD" chr_22.zip`).

## How long are my results available?

Your data is available for 7 days. If you need to extend the due date, please [let us know](contact.md) ahead of time. Once the data is deleted we cannot help.

## How many times can I download my files?

There is a limit of 50 downloads per file. Please [let us know](contact.md) if you need an extension.

## How can I improve the download speed?

[aria2](https://aria2.github.io/) tries to utilize your maximum download bandwidth. Remember to raise the k parameter significantly (-k, --min-split-size=SIZE). You will otherwise hit the Imputation Server download limit for each file (thanks to Anthony Marcketta for pointing this out).

## Can I download all results at once?

Yes! If you click the download icon in the **Results** tab, you will encounter `wget` and `curl` commands to easily download all the files from the same step at once. You can also use the `imputationbot` or your own scripts for parallel downloads.

## Is this service secure?

Due to small team size, it is difficult for us to complete detailed security questionnaires for every company or entity. However, as of May 2023, we have completed a rigorous external security review and received federal Authorization to Operate (ATO) from NHLBI/NIH. Please see our [security](imputation/data-security.md) documentation for some common information, or [contact us](contact.md) for specific questions.

## Why do my VCF 4.3 chromosome X phasing jobs keep failing?

Your chromosome X phasing jobs are likely failing because the server doesn’t currently support VCF version 4.3. The server attempts to rewrite chromosome X due to the PAR/non-PAR split using the htsjdk library, which isn’t fully compatible with VCF 4.3, especially for chromosome X data. To resolve this issue, try changing the VCF header to version 4.2. The server supports this version and should allow your phasing jobs to complete successfully.
