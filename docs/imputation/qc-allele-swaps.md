# Changes in Allele Swaps Handling in Imputation Server 2.0

With the update to the latest Minimac4 version, we have modified the default handling of allele swaps in Michigan Imputation Server 2. 

## Overview of Changes

In previous versions of the server, swapped reference and alternate alleles were automatically corrected, with only ambiguous SNPs being filtered out. To improve genotype quality and avoid errors from incorrectly genotyped variants, the handling of allele swaps has now been updated.

## Key Changes

- **Previous Behavior**: Allele swaps were automatically corrected, with only ambiguous SNPs being filtered out.
- **New Behavior**: With the update, the server now enforces stricter quality control. The system allows up to 100 allele swaps during processing. If more than 100 allele swaps are detected, the quality control process will fail, stopping the imputation process to prevent poor genotype quality.

## Impact on Data Submissions

This update may cause failures in submissions that were previously accepted but now exceed the 100-allele swap threshold.

## Recommendations

To avoid imputation quality issues and ensure your data meets the new allele swaps threshold:

- We recommend using tools such as Will Rayner’s tool to check for and correct allele switches in your dataset.

- This will ensure that issues such as **strand or allele flips** are resolved before submission, helping your data meet the new quality standards.


Please follow these steps to verify your data before submitting it to Imputation Server 2.