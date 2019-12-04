# ADR-005 Terraform code structure

Date: 28/11/2019

## Status

Accepted

## Context

DSO team should have a consistent and well defined method of structuring our
Terraform code. Existing code has been structured around utilising Terragrunt,
however in [adr-0005](./0006-deprecate-use-of-terragrunt.md) the DSO team has
decided not to use Terragrunt anymore. This requires we have a structure
optimised for using Terraform.

## Decision

DSO Terraform structure should be based upon a single root module which will
setup common Terraform details. Before calling one or more functional modules.

This extends the recommendations from Hashicorp regarding [creating
modules](https://www.terraform.io/docs/modules/index.html)

**root modules**

A root module has the following responsibilities:

  - configure Terraform state back end.

    This should define which backend to use, but expects to use a partial
    backend configuration to set some details via cli arguments detailed
    [here](https://www.terraform.io/docs/backends/config.html#partial-configuration)

  - provider configuration.

    Provider configuration becomes more complex as modules will generally
    utilise multiple providers. However as the majority of our estate runs in
    azure we can set this up in the root module, which then gets automagically
    inherited by the functional modules. If functional modules require any
    additional providers, such as template files, passwords etc, they will need
    to configure those themselves.

    This will result in some code duplication, however it should be minimal.

  - Functional module invocation.

    The root module is responsible for invoking any functional modules required
    to deploy the desired infrastructure.

  - determining appropriate inputs

    Terraform code should be capable of being deployed into many different
    environments with no code changes. This means that environment specific data
    needs to be passed to Terraform via the use of either tfvar files or cli
    arguments.

    These must be passed to the root module as part of the Terraform invokation
    and the root module is responsible for ensuring that appropriate inputs are
    passed to the functional modules. This includes passing outputs of one
    module into the inputs of another.

  **NOTE:** the root module should **NOT** be deploying any resources directs.
  It's sole purview is to orchestrate the use of functional modules whose role
  it is to deploy resources.

**functional Modules** A functional module has the following responsibilities:

  - deploy a discrete piece of reusable infrastructure

    A discrete peice of reusable infrastructure means a collection of resources
    that work togeather to provide a single peice of useful architecture. This
    architecture should be flexible enough to deploy many similar architectures.

    An example may help clear this up.

    Most of the DSO space is made up of a cluster of WebLogic servers speaking
    to a number of Oracle Databases, with some Engineering tools around the
    edges in an azure subscription.

    It might make sense to have a single functional module to setup and
    configure some common resources. For instance a base module may deploy a
    resource group and within that configure a vnet, an application subnet, a
    data subnet and an engineering tools subnet. Other potential resources could
    be network security groups for each of the subnets.

    With appropriate inputs to configure it, this functional module could then
    be used by all of the application stacks DSO are responsible for.

    All functional modules would also be required to expose relevant resources
    via outputs. This would allow other modules to attach network rules to the
    NSGs in the example above. This allows a relevant seperation of concerns.

    A functional module may make use of serveral other Terraform modules where
    that makes sense.

  - Rely more on modules inputs rather than remote state data.

    Using Terraform v0.12 we can pass whole resources as variables so this
    becomes much less tedious than in previous versions.

    Currently existing Terragrunt implementations do not utilise inputs to pass
    data between modules. Instead as each module is invoked seperately, due to
    the use of Terragrunt, there are a number of state files used, and existing
    modules make use of these. However this means each module that utilises the
    remote state of another module tightly binds the two modules togeather.
    As if the remote state changes, e.g. names of resources, then all modules
    using that remote state may break with no warning.

    If instead inputs are used, a module can require that there be subnet and
    NSG resources passed in as inputs, so it can deploy servers and network
    security rules. The module does not care how those resources are created,
    and it becomes the calling modules responsibility to ensure that the correct
    inputs are provided.

  - Accept the following required inputs:

    * namespace

**workspaces**

[Terraform workspaces](https://www.terraform.io/docs/state/workspaces.html) are
a useful tool to help separate code deployments using Terraform. Each Terraform
configuration has an associated back end configuration where the state file is
stored. Workspaces are an extension to this configuration allowing multiple
state files to be used for the same configuration. There is still only one
back end, however there are multiple state files, one for each workspace. This is
a convenient way of having multiple deployments without having to utilise entire
new back end configurations, as is the case with Terragrunt.

Currently we have separate state files based on which environment we deploy
to. This works by having multiple back end configurations. Instead each
subscription we deploy to should have a single back end configuration, and we
should make use of workspaces to deploy multiple environments into each
subscription. This means workspaces do not replace our existing state file
separation, instead they should augment it.

Within Terraform code we can access the name of the workspace using
`${terraform.workspace}` which can be used anywhere interpolations are allowed.
This should be used as part of resource name spacing. For this to work namespaces
should only contain lowercase characters and numbers.

For DSO development workspaces should be named for the git branch containing the
work to be tested. This will need to be transformed to meet the naming
convention above.

**variables and resource names**

For consistency all variables and resource names should be lowercase characters
only with underscores separating words. This is entirely arbitrary and other
naming conventions should be used if felt appropriate.

**NOTE** `resource name` here refers to the name Terraform uses to differentiate
different resources of the same type, not the name property that many resources
have.

**namespacing resources**

As we expect to deploy the same configuration multiple times into the same
subscription we will need to namespace the resources effectively. Each resource
being deployed with a name property should have a prefix used in that name to
differentiate it from the same resource deployed from another deployment.

The prefix should include details common to all resources within that Terraform
configuration. The prefix should be defined in the **root** module and passed
into all **functional** modules called.

A recommended default should be the workspace name followed by the product being
deployed E.G.

  ${terraform.workspace}nomis

**NOTE** only lowercase characters are used, and potentially numbers from the
workspace. This is because many resources have naming constraints that restrict
the use of symbols and uppercase characters. Therefore to have a consistent
prefix we should only use lowercase letters and numbers.

## Consequences

As a result of these changes our Terraform code should become more clearer and
better composed. Functional modules will be reused and should be written in a
fairly generic manner, while root modules will be written to use those
functional modules to deploy a specific piece of infrastructure. For instance
we may have a few functional modules, `azure_base_networking`,
`azure_weblogic_oracle_db_app`, `azure_engineering_tools` modules, which with
different inputs could be used by the following root modules, `azure_nomis`,
`azure_mis`, `azure_oasys`.

Having multiple workspaces in use will make development quicker and simpler by
not requiring that a whole new back end configuration setup is created for each
change being tested.
