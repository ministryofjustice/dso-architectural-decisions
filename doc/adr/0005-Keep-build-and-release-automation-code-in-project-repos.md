# ADR-0005. Keep build and release automation code in project repos & automation libraries, rather than CI-tool DSLs

Date: 29/11/2019

## Status

Proposed

## Context

As part of infrastructure modernisation, the DSO team must select a CI task runner and strategy for using it. The field of CI/CD is moving quickly, and an over-riding industry standard has not settled. New developments in the field arrive regularly, so we must be careful to construct our automation in such a way that we can easily move between automation systems.

## Decision

In order to be able to trigger our automation from different systems, and from our local development workstations, team should keep their automation with the project code they are deploying.

### Folder Structure, CI config and script locations

This is documented in 0006-Standard-repo-folder-structure.md (in progress).

### Versioning

If CI automation scripting grows in complexity then it should be broken out into tested, versioned docker images, as these can be imported and used easily from any CI agent or dev machine.

### Secrets

Secrets required to perform the automation should be pulled from Azure keyvault (using ```az keyvault secret``` or similar). Developers should have access to pull secrets for development/test deployments, but only the CI tool's service principal should be able to pull production secrets.

### Job persistence/long running actions

Where we have jobs that take long periods of time, scripts should be split up and the minimum amount of DSL config should be created to orchestrate the execution of said components.

## Consequences

We'll have CI-tool agnostic build/test/release/deploy automation;
Some automation may take longer to create where DSL features already exist.
