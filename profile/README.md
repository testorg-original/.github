# Testing org name transition for the hubverse

In this organistion I'll be testing the implications of the hubverse org name change on our hubs and software and confirming necessary actions by various stakeholders.

## Stage 1: Setting up test organisation

1. First I forked the hubverse schema repo  into the `testorg-original` organisation and replaced all original org name instances (`Infectious-Disease-Modeling-Hubs`) in the contents to the new org (`testorg-original`). 

2. I then did the same on branches of `hubUtils`, `hubAdmin`, `hubData` and `hubValidations`.
3. For `hubValidations` tests to pass, URLs in all branches of `ci-testhub-simple` also needed updating,
4. All tests across packages ran successfully apart from those testing against the S3 test bucket (which is expected as I had not changed the config the bucket hubs).

## Setting up test hub

To have a hub to perform tests on, I also forked `example-complex-forecast-hub` and again replaced all original org name instances (`Infectious-Disease-Modeling-Hubs`) in the contents to the new org (`testorg-original`). Initially I only did this on the main branch. 

I ran:

```r
hubValidations::validate_submission(
  hub_path = ".",
  file_path = "MOBS-GLEAM_FLUH/2022-10-22-MOBS-GLEAM_FLUH.csv",
  skip_submit_window_check = TRUE
)
```
to double check validations were working and the file did indeed validate successfully.

## Changing org name to `testorg`

Next I changed the org name to `testorg-rename` and ran through the validation and tests run previously. Need to ensure a fresh session everythim as the functions for fetching the schema are memoised.

### Check Hubs

- [x] Hub validation of file worked! locally.
- [x] PR would work also as `install_github("testorg-original/hubAdmin")` works.

### Check Sotware Tests

- [x] `hubUtils`
- [x] `hubAdmin`. However the functions for creating config schema ids was obviously still creating URLs to the old org name. It looks like they would work through redirects anyways though.
- [x] `hubData` same as before, only S3 tests that had not been originally updated failed. But I imagine, just like other hubs, if we had started with `testorg-original` these would also have passed.
- [x] `hubValidations`
