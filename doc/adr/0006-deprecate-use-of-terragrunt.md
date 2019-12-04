# ADR-004 Deprecate use of Terragrunt

Date: 28/11/2019

## Status

Accepted

## Context

Historically the Digital Studio has made use of
[Terragrunt](https://github.com/gruntwork-io/terragrunt) as a wrapper around
terraform. The predominant reason is because the terraform code sturcture has
been to split artchitecture up into seperate root modules and iterate over them
using several invokations of terraform, this seems to be the gruntworks pattern.
As these modules are invoked independantly of each other several configuration
settings need to be kept consistent between these modules, which means repeating
the same configuration in several files. Terragrunt's purpose is to abstract the
backend and provider configurations out of terraform and into some external
tfvars configuration that terragrunt reads and repeats it for you.

## Decision

Since the original release of terragrunt terraform has become much more mature.
The backend and provider configs can now be determined by cli flags so having
terragrunt is no longer necessary.

Use of terragrunt complicates our terraform code, as new engineers need to be
familiar with both terraform and terragrunt to be able to utilise our code.

In addition to the above reasons, it is also been seen that our specific use
of terragrunt has made extensive use of Bash scripts which have not been written
in a portable manner. This has lead to issues where terragrunt has not been
easily deployed from machines that do not meet the undocumented environment
expected.

## Consequences

Terragrunt will be deprecated. Future projects the DSO team pick up will no
longer make use of Terragrunt. When there is time, terragrunt may be removed
from existing projects. However this is not work that should be actively
pursued.

Instead of relying on Terragrunt to orchastrate terraform modules, we will
instead use a thin root module who's sole function is to call functional modules
with correct inputs.

There will be some issues around invokation of terraform as result of not using
terragrunt. There are several CLI options that terragrunt adds automagically to
terraform such as backend config and appropriate tfvar files. Without terragrunt
we will need to ensure that this is used appropriately when invoking terraform.

I propose that we include a wrapper script that adds these details in. However
the design of that should be linked with the discussion of cicd script logic
discussed in [adr-0005](./0005-Keep-build-and-release-automation-code-in-project-repos.md)
