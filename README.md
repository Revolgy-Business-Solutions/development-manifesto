# Revolgy Development Manifesto

## About

TODO general description, we the people

TODO mention that content in quotes is optional reading

TODO mention that everything should be done diligently and responsibly.

### Technologies

We primarily use the following technologies:

- [Git]
- [Terraform]

**Git** is currently the unofficial standard [Version Control System][VCS]. [Git can be installed on almost any platform][Installing Git] and is supported by most development environments natively.

> Even though most Git repositories seem centralized, for example on [GitHub], Git itself is a distributed VCS.
> There is no major difference between the repository you might have on your PC, and the repository on your platform of choice—you could _technically_ host the repository on your PC for public viewing.
> Some more traditional workflows, such as the one of the creator of [Linux] (and also Git itself), [Linus Torvalds], use Git in such a fashion that Linus' own repository is considered to be the main repository of the kernel where he accepts incoming changes via email and only when he pushes it to [torvalds/linux.git] is it considered an officially accepted change of the "mainline" kernel. Other Kernel developers and maintainers have their own Linux repositories. [stable/linux.git] hosts the "stable" kernel. While it is technically a different repository, it has mostly the same commits with mostly the same history as the "mainline" kernel, and so you might have linux.git locally with both the "mainline" and the "stable" repositories as remotes.

**Terraform** is a [Infrastructure-as-Code][IaC] tool.

> Terraform is really a relatively simple tool which through its providers is capable of provisioning infrastructure on almost any cloud platform with an API.
> On each run of Terraform, it checks the entirety of the current state and code base, which only gets slower with each new resource, and Terraform doesn't have a built-in solution to this problem, other than that there is a simple recommendation to keep its code bases small and in stages.
> Terraform must be able to resolve the dependency graph as it's written in code, otherwise it won't run.

### Strategies

At Revolgy, we have elected to use the following procedures, processes, or strategies.

- [Trunk Based Development]
	- ... is a branching strategy for Git.
- [Code Review]
	- ... is the idea that all code that gets released is looked at by more than 1 person.
- [CI/CD]
	- ... is the process that automatically runs tests and builds the software—or in our case, the infrastructure.
	- **Note**: This guide assumes that the environment is set so that **EVERY CHANGE IS TESTED** before ultimately being used. 

## Trunk Based Development (Git)

### Definitions

This section requires that the reader know the basic usage of **Git**.

- **commit**: in Git, a commit is a collection of files in a tree, and the commit usually has a preceding commit.
- **branch**: in Git, a branch is a name for a specific commit and therefore its history as well. Branches can be updated by **push**ing, and pushing cannot change history—unless with `--force` on branches that aren't protected.
- **protected branch**: on most Git platforms, a protected branch is a branch that cannot be pushed to directly, or at least not by non-owners, and has to be merged to, usually with Code Review.
- **rebase**: in Git, a rebase is the act of taking a branch that split off from its parent branch at some point in the past, and re-committing on top of a different (usually newer) commit.
- **merge**: in Git, a merge is the act of joining two branches together. There are several strategies that Git can use based on the circumstances.
	- **fast-forward**: a fast-forward merge requires that the branch that is getting merged be based on top of the branch that its getting merged into. The fact that the off-branch is in such a position means that any merge conflicts (or rebase conflicts) have already been resolved. Then, it can simply be updated with all those new commits.
	- **merge commit**: a merge commit is a special type of commit that has two parent commits, effectively joining them. The use of a merge commit may sometimes lead to there being a merge conflict, which has to be resolved manually.
- **merge request**: (or **pull request**) on most Git platforms, a merge request is a form of keeping track of changes that are proposed to be merged into a different branch, usually the main protected branch. Usually, it can be set so that only the "maintainer" or "owner" of the repository can accept the MR, and this person can also directly request that changes be made to the MR before acceptance through discussion.

### Abstract

`trunk` is the main branch of the repository. It is protected and _shouldn't_ be pushed into directly under normal circumstances.

Every change is being developed in its own temporary branch created off `trunk`.

Changes are to be accepted only after CI and Code Review, and new test coverage must be part of the Merge Request if applicable.

`trunk` is an active rolling branch with the latest code which can (and should) be continuously deployed into a **testing** environment.

To cut a new release, a `release` branch is created and the release commit is **tag**ged with a version number. Stick to [Semantic Versioning].

Tagged releases can be then deployed by a CD pipeline.

`trunk` branch is periodically merged into the current `release` and when a new version of the current release should be issued, appropriate commit is being tagged with new version number.

If we run into a situation when we need to back-port something into an older release branch which is not in sync with `trunk` anymore, we `cherry-pick` a single commit from the `trunk` and tag a new version of this older release.

### Applied TBD

TODO actual description of TBD

![Diagram](./img/trunk_based_development.png)

## Code review (Git, GitLab, GitHub)

### Definitions

TODO explain PR/MR based on platform

### Abstract

TODO short version of code review

### Applied code review

TODO practical examples, templates

## Terraform structuring and best practices (Git, Terraform)

### Project structure options

TODO table of "projects", "modules"...

TODO gitignore

### Remote state

TODO best practices for remote state

### Writing modules

TODO standard structure of modules based on official recommendation

TODO methodology for what inputs and outputs there should be

TODO write description everywhere in outputs and inputs

### Using modules

TODO using local modules

TODO using remote modules

### CI/CD

TODO basic stuff

### Optimization for CI/CD performance

TODO stages with CI/CD not running that part of terraform to save time

[Git]: https://git-scm.com/
[Trunk Based Development]: https://trunkbaseddevelopment.com/
[Terraform]: https://www.terraform.io/
[IaC]: https://en.wikipedia.org/wiki/Infrastructure_as_code
[CI/CD]: https://about.gitlab.com/topics/ci-cd/
[Code Review]: https://about.gitlab.com/topics/version-control/what-is-code-review/
[VCS]: https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control
[Installing Git]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
[GitHub]: https://github.com/
[Linus Torvalds]: https://en.wikipedia.org/wiki/Linus_Torvalds
[torvalds/linux.git]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
[stable/linux.git]: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
[Linux]: https://kernel.org/
[Semantic Versioning]: https://semver.org/