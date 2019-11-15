# ADR-0004. Git Branching Strategy

Date: 12/11/2019

## Status

Accepted

## Context

A team utilising git repositories should have a consistent set of branching
strategies. This will enable encoding relevant information into a branch name
for improved efficiency. Each component of the branch name will then be able to
be utilised in other areas of automation.

## Decision

Each branch name will be broken down into 3 components. An optional category,
issue-id reference and a brief summary. The approved format
is shown below.

  `<category>/<issue-id>-<summary>`
  OR
  `<issue-id>-<summary>`

each component has a set of rules it must adhere to.

  If present _**category**_ MUST be one of _**feature**_, _**bugfix**_, _**security**_ or _**personal**_.

  each _**category**_ describes the type of work to be found in that branch.

  - _**feature**_ branches introduce some new functionality to the repository.
  - _**bugfix**_ branches are fixing some issue with existing code.
  - _**security**_ branches are resolving a security issue with the code
  - _**personal**_ branches are for work that is independant of a specific ticket.

    Determination of which category to use is fairly arbitrary and engineers
    judgement should be used. There is likely to be some overlap, so just use
    whichever fits best or no _**category**_. The main exception is for the  _**personal**_ category
    which is to be used for experimenting with different approaches, but with
    no view to deployment. work from a personal branch would then be pulled
    into a branch in one of the standard categories.

  _**issue-id**_ is to be used to namespace environments created using
  the branch. This should only be a default and could be overridden by an
  explicit decision. For branches in _**feature**_, _**bugfix**_ and _**security**_
  categories the _**issue-id**_ MUST be the jira ticket id.
  This allows details about the work contained in the branch to be quickly and
  easily correlated with work in Jira.

  For branches in the _**personal**_ category the _**issue-id**_ is not relevant
  so instead a similar format but using a username as the jira project should
  be used. e.g. pmccabe-1

  _**summary**_ should be a consise description of the work contained in the branch.
  This should consist only of lowercase letters numbers and dashes to
  seperate words. This should not be a replication of the summary line in the
  jira ticket. but should be more consise.

  #### Examples:

  - feature/JIRA-34-create-new-weblogic-servers
  - bugfix/JIRA-35-enable-selinux
  - security/JIRA-36-update-to-latest-rhel-version
  - personal/PMCCABE-1-experiment-with-vmextension-script-deployments
  - JIRA-37-setup-some-secrets

Each branch should be created from `origin/master` then worked on and tested.
When ready a Pull Request should be generated to merge back into `origin/master`

## Consequences

With a standard approach to branching we can use automated scripts to extract
information as part of CI/CD pipelines. For example the
_**issue-id**_ can be used to uniquely namepace deployments
based on the branch name.

Each branch _**may**_ have it's own deployed environment to use for testing the
developed code. There will be a 1 to 1 relationship between deployed
environments and repository branches. The only exception to this 1 to 1
relationship will be the _**master**_ branch. The _**master**_ branch may be used to
back any environment that needs to be live like.

We can also, at a glance, see what jira tickets are being developed. this will
ease lookup issues if team members are away and work needs to be handed off.


