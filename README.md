# Revolgy Development Manifesto

## About

TODO general description, we the people

TODO mention that content in quotes is optional reading

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

TODO describe why

- [Trunk Based Development]
- [Code Review]
- [CI/CD]

## Trunk Based Development (Git)

### Definitions

TODO requirements that reader knows basic git commands

### Abstract

TODO short version of TBD

### Applied TBD

TODO actual description of TBD

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