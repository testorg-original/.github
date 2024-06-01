# Testing org name transition for the hubverse

In this organistion I'll be testing the implications of the hubverse org name change on our hubs and software and confirming necessary actions by various stakeholders.

## Stage 1: Setting up test organistion

1. First I cloned the hubverse schema repo and replace all org name instances in the contents to the new org (`testorg-original`). 

2. I then did the same on branches of `hubUtils`, `hubAdmin`, `hubData` and `hubValidations`. All tests across packages ran successfully apart from those testing against the S3 test bucket (which is expected as I had not changed the config the bucket hubs).
