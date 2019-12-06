# ADR - Which branch holds what is reflected in Live?

Date: 4/12/2019

## Status

Accepted

## Context

DSO keeps both 'product' code and 'config' code in github. Currently this all tends to be stored together in one repo.

Product code developers traditionally signal to the release process that a release is _ready for release_ by 'tagging' a commit and pushing the tag. Often the tag is created by the CI tooling itself so that a tagged commit can only ever be released if it has been through an assurance process and is therefore _ready for deployment_.

Infrastructure teams working with 'configuration repos' tend to use branches or folders/files to represent which _released_ version of the infrastructure and/or application code is currently _deployed_ in which environment. This means that these live branches must be protected. The PR merge process is the means by which a decision to deploy is made.

## Decision

We should manage the 'release process' of products and the 'deployment process' of services according to their own, differing requirements, using industry standard automation processes to protect service while increasing development velocity and ease of release & deploy.

### Product code

#### Versioning/Packaging

Tested, released product code such as a terraform project, a helm chart, or an npm package should be stored in either:

1) The appropriate write-only repository type
- ie https://www.npmjs.com/ for nodejs packages
- docker hub or ACR for docker images
- etc

or

2) In github as a specific tagged commit.

These two constructs are immutable (write-only), and so form a contract with their consumer that whatever testing has been performed against them has not been invalidated by hidden change.

Development of product code should be done directly on master, so that everyone is integrating continuously.

#### References / Dependencies

*Released* products MUST specify their dependencies using immutable references, or must vendor dependencies in so that the resulting deploys are safe.

*Pre-release* products MAY refer to a dependency using a moving reference, but this MUST be converted to a static reference when moving into test.

### Service Configuration Code

#### Location

**All infrastructure services we operate MUST store their configuration in github.**

#### References / Dependencies

A service 'depends' upon the products that incorporate it, therefore the rules that apply to service configuration code are similar to the ones that apply to products:-

*Production and Test* service config MUST specify dependencies using immutable references.

*Development* service config MAY refer in text to a dependency using a moving reference ('latest' or 'master' etc), but this MUST be converted to a static reference when moving into test.

#### Signalling to the service deploy process that an infrastructure service version should be changed

Environment-specific static configuration data (e.g. tfvars) must be stored as text in the repo, in a folder or file which is named after the branch name.

The process for deploying a new version of this config to a static environment (or to roll back to an old version) MUST be implemented as merging a tested commit onto a protected branch that represents the environment. This merge will trigger automation to converge the system onto the correct state.

There are two processes by which this merge and deployment can be allowed to occur:-

1) A review, followed by an automated deployment;
2) An immediate automated deployment.

Which occurs depends on:-

1) Whether the system is a 'live' system (a system is live if it supports a customer's production, test or dev workloads);
2) How comprehensive the testing and assurance processes for this commit have been. Where a system has a mature process of this kind, it is commonly called 'continuous deployment'.
3) The risk profile for the service;
4) Customer approval requirements.

A determination of which should be made by team consensus.

#### Branch builds

If an infrastructure service is under active development by more than one developer, the CI tooling SHOULD respond to branch activity as follows:-

```
If a branch is pushed named INFRA-*
	if an environment does not exist in the azure devtest environment
		deploy a new instance of the service
	else
		update the service with the new code
	notify the developer who made the last commit where the environment is

If a branch is deleted named INFRA-*
	destroy the environment
```

This obviously requires that infrastructure products are written to support the easy deployment of multiple service instances.

### Same repo?

The above constraints can be implemented where a product and service share the same repo. CI tooling MUST ensure that only specific tagged commits that have received appropriate testing may be merged into live service branches.

## Consequences

Positive:
- Developers will not interfere with each other's work
- It will be clear what version of a service is deployed
- It will be easy to roll service instances back

Negative:
- Automation libraries to achieve this must be created
- Deploying will be slower to start with, and annoying because we're used to deploying things with developer accounts
