# ADR-003. Which branch holds what is reflected in Live?

Date: 4/12/2019

## Status

Accepted

## Context

DSO keeps both 'product' code and 'config' code in github. Currently this data tends to be stored all together in one repo.

Product code developers tend to signal to the release process that a release is ready by 'tagging' a commit and pushing the tag. Often the tag is created by the CI tooling itself so that a tagged commit can only ever be released if it has been through an assurance process and is therefore 'ready for deployment'.

Infrastructure teams working with 'configuration repos' tend to use branches or folders/files to represent which _released_ version of the infrastructure/ and/or application code is currently _deployed_ in which environment. This means that branches must be protected and the PR merge process is the means by which a decision to deploy is made.  

## Decision

We should manage the 'release process' of products and the 'deployment process' of services according to their own, differing requirements.

### Products

#### Versioning/Packaging

Tested, released product code such as a terraform project, a helm chart, or an npm package should be stored in either 

1) The appropriate write-only repository type
	- ie https://www.npmjs.com/ for nodejs packages
	- docker hub or ACR for docker images
	- etc

or 

2) In github as a specific tagged commit.
	
These two constructs are immutable (write-only), and so form a contract with their consumer that whatever testing has been performed against them has not been invalidated by hidden change. 

Development of product code should be done directly on master, so that everyone is integrating continuously.

#### References / Depencencies

*Released* products MUST specify their dependencies using immutable references, or must vendor dependencies in so that the resulting deploys are safe.

*Pre-release* products MAY refer to a dependency using a moving reference, but this MUST be converted to a static reference when moving into test.

### Service Configuration

#### Location

All infrastructure services we operate MUST store their configuration in github.

#### References / Depencencies

A service 'depends' on the products that incorporate it, therefore similar rules apply for service configuration code that apply to products:-

*Production and Test* service config MUST specify dependencies using immutable references.

*Development* service config MAY refer in text to a dependency using a moving reference ('latest' or 'master' etc), but this MUST be converted to a static reference when moving into test.

#### Signalling to the service deploy process that an infrastructure service version should be changed

Environment-specific static configuration data (e.g. tfvars) must be stored as text in the repo, in a folder or file which is named after the branch name.

The process for deploying a new version of this config, or to roll back to an old version MUST be implemented as merging a tested commit onto a protected branch that represents the environment. This merge will trigger automation to converge the system onto the correct state.

There are two processes by which this merge and deployment can be allowed to occur:-

1) A review, followed by an automated deployment;
2) An immediate deployment.

Which occurs depends on:-

1) whether the system is a 'live' system (a system is live if it supports customer production, test or dev workloads);
2) how comprehensive the testing and assurance processes for this system are; 
	where a system has a mature process of this kind, it is commonly called 'continuous deployment'.
3) the risk profile for the service;
4) customer approval requirements







1) begin a customer/peer review process which, after completion, will trigger automation to converge the system on the correct state, or
2) skip review and directly trigger automation to converge the system on the correct state.

##### Dynamic envrironments



The option that is chosen 



### Same repo

The above constraints can be implemented using the same repo.




## Consequences





When describing a *production or test* service, config-storing systems MUST NOT refer to a product version using a _moving_ reference, such as a branch name 
When describing a *development* service, config-storing systems MAY refer to a product version using a moving reference, but this MUST be converted to a static reference when moving into test.
