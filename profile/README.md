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


## Change org name in software

Next I went through and replaced `testorg-original` with `testorg-rename` as before. All changes were done in branch `change-orgname`.

- [x] `hubUtils`: As `hubUtils` contains the utility functions for working with schema, it's best to change this package first. 
  In addition, there is a test in `hubUtils` for checking the validity of the config in test hubs in `inst/` which required `hubAdmin` functions. For those to pass, `hubAdmin` also needed to be updated, in particular function `check_config_schema_version` (and `validate_schema_version_property`) which returns the following error before replacing:
  ```
  Error in purrr::map(configs, ~validate_config(hub_path = hub_path, config = .x, : i In index: 1. Caused by error in `check_config_schema_version()`: x Invalid `schema_version` property. i Valid `schema_version` properties should start with "https://raw.githubusercontent.com/testorg-original/schemas/main/" and resolve to the schema file's raw contents on GitHub.
  ```
  Allowing for this check to also accept the old org-name as well as the new one should make this check back-compatible.
- [x] `hubAdmin`. When updating `hubAdmin` without having updated the schema, functions for creating config objects where using the old orgname. That's because it propagates the `id` of the schema property of the file schema actually used to create the config against. This sounds like reasonable behaviour. It cause problems for tests but that's because the find and replace action also replaces the orgname in the snapshots of test results. So while the functions where using schema and creating config objects with the old orgname, the snapshots had been updated with the new orgname. This would would either require re-snapshotting manually to allow for this or more targeted find and replace. Both are fiddlier than necessary and would only be required if the orgname was only changed in the latest schema version directory (instead of the whole schema repo, see below). It's why I'm proposing a full reset of the schema repo instead of just a single directory.
- [x] `hubData` same as above.
- [x] `hubValidations` tests passed except for tests on functions that check validity of hub `config` (e.g. `validate_pr`) where the same error emanating from `validate_schema_version_property` and reported above in `hubUtils` testing was observed (`"EXEC ERROR: Error in purrr::map(configs, ~validate_config(hub_path = hub_path, config = .x,  : \n  i In index: "| __truncated__`. Making `validate_schema_version_property` back-compatible would fix this issues also though.
- [ ] Validating submission file without change orgname in config files now fails with the familiar error:
  ```
      ✖ 2022-10-22-MOBS-GLEAM_FLUH.csv: EXEC ERROR: Error in purrr::map(configs,
  ~validate_config(hub_path = hub_path, config = .x, : ℹ In index: 1. Caused by error in
  `check_config_schema_version()` at hubAdmin/R/config-schema-utils.R:8:3: ✖ Invalid
  `schema_version` property. ℹ Valid `schema_version` properties should start with
  "https://raw.githubusercontent.com/testorg-rename/schemas/main/" and resolve to the schema
  file's raw contents on GitHub.
  ```
  
  Again, making `check_config_schema_version` back-compatible would fix this problem.

  **Once find and replace was performed, validation succeeded**. However, find and replace also fixed instances in GitHub Actions, and various READMEs in the hub which is overall a good idea.



## Change org name only in new schema version

Next I created a new version in the schema main branch and only changed the org name in the `id` properties of the latest version. Then I re-ran all above checks:
- [x] `hubUtils` all good
- [x] `hubAdmin`. Now, in many of the `create_*` family of functions tests which use the default latest schema, tests are failing and new snapshots with the new schema are required, which is fine and expected. Snapshots which point to older versions though for back-compatibilty are still also failing and will need to be set to the older org name (as discussed above).
- [x] `hubData`. All good.
- [x] `hubValidation` same as above.
- [x] Validating submission files: succeeds same as above (as org name has been changed)


## Make `hubAdmin` back-compatible

Next I allowed `check_config_schema_version` and `validate_schema_version_property` to accept both `testorg-original` and `testorg-rename` as valid organisations in the schema version URL check. This fixed the failing `validate_pr` tests in `hubValidations` and made the function back-compatible.

## Replace or instances of the old org in the schema repo

I tried this with the back-compatible version of `hubAdmin` and without.

- [x] `hubUtils` all good.
- [x] `hubData` all good. As above
- [x] `hubAdmin`. Now all tests pass and even using older versions of the schema with `create_*` functions now uses the updated org name.
- [x] `hubValidations` without `hubAdmin` back-compatibility this fails again because the schema_id in the ci-test hub is still pointing to the old org-name. With the back-compatible version all is good.
- [x]  Validating submission files: succeeds same as above (as org name has been changed)


# Summary

Link redirecting actually makes initial transition cause no problems with the current versions of software (so long as the orgname `Infectious-Disease-Modeling-Hubs` is still available).  Adding back-compatibility in the couple of functions that were explicitly using the old orgname will allow hubs with the old org in their `schema_id` to use newer versions of packages. This will give them breathing space to update schema ids at their convenience.

I would also suggest just changing the old org name to the new orgname through the entire schema repo. It makes all other updates (`hubAdmin` in particular) easier with a find and replace. It also means that creating config using older versions of the schema with `create_*` functions will also use the updated org name.

Finally, I suggest we fold all this into the release of the v3 schema.

## Actions 

### [hubverse devs] (to facilitate smooth transition)

- Add back-compatible version of `check_config_schema_version` and `validate_schema_version_property` in `hubAdmin`. to allow for both old and new orgnames to pass validation. hub admins to take their time updating
- Find and replace old orgname with new orgname throughout repos in the organisation
- It might also be a good idea to find and replace instances of `Infectious Disease Modeling Hubs` also.
- All remotes need to be changed in all repos too to silence the following git push warnings.
  ```
  Please use the new location:        
  remote:   https://github.com/testorg-rename/hubValidations.git        
  To https://github.com/testorg-original/hubValidations.git
  ```

### [hub admins] 

- At some point hub admins would be advised to do a find and replace to the new org name in their hubs.
- It might also be a good idea to find and replace instances of `Infectious Disease Modeling Hubs` also.
- Remotes need to be changed too to the correct org name.

