# ADR-004 Deprecate use of Terragrunt

Date: 28/11/2019

## Status

Accepted

## Context

Historically the Digital Studio has made use of
[Terragrunt](https://github.com/gruntwork-io/terragrunt) as a wrapper around
Terraform. The predominant reason is because the Terraform code structure has
been to split architecture up into separate root modules and iterate over them
using several invocations of Terraform, this seems to be the Gruntworks pattern.
As these modules are invoked independently of each other several configuration
settings need to be kept consistent between these modules, which means repeating
the same configuration in several files. Terragrunt's purpose is to abstract the
back end and provider configurations out of Terraform and into some external
tfvars configuration that Terragrunt reads and repeats it for you.

## Decision

Since the original release of Terragrunt Terraform has become much more mature.
The back end and provider configs can now be determined by cli flags using what
Terraform refer to as a [partial
config](https://www.terraform.io/docs/backends/config.html#partial-configuration),
so having Terragrunt is no longer necessary.

Use of Terragrunt complicates our Terraform code, as new engineers need to be
familiar with both Terraform and Terragrunt to be able to utilise our code, and
there are far fewer resources available regarding Terragrunt, than Terraform

In addition to the above reasons, it is also been seen that our specific use
of Terragrunt has made extensive use of Bash scripts which have not been written
in a portable manner. This has lead to issues where Terragrunt has not been
easily deployed from machines that do not meet the undocumented environment
expected.

## Consequences

Terragrunt will be deprecated. Future projects the DSO team pick up will no
longer make use of Terragrunt. When there is time, Terragrunt may be removed
from existing projects. However this is not work that should be actively
pursued.

Instead of relying on Terragrunt to orchestrate Terraform modules, we will
instead use a thin root module who's sole function is to call functional modules
with correct inputs.

There will be some issues around invocation of Terraform as result of not using
Terragrunt. There are several CLI options that Terragrunt adds automagically to
Terraform such as backend config and appropriate tfvar files. Without Terragrunt
we will need to ensure that this is used appropriately when invoking Terraform.

I propose that we include a wrapper script that adds these details in. However
the design of that should be linked with the discussion of cicd script logic
discussed in [adr-0005](./0005-Keep-build-and-release-automation-code-in-project-repos.md)
