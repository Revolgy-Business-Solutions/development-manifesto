# Revolgy Development Manifesto

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

We all need to have a common way of working with the tools with which we, engineers, use to build the environments and software. 

- [Trunk Based Development]
	- ... is a branching strategy for Git.
	- Trunk Based Development has been chosen over the alternatives for its simplicity and compatibility with infrastructure code bases.
- [Code Review]
	- ... is the idea that all code that gets released is looked at by more than 1 person.
	- This is so that one person can't just release something on a whim.
- [CI/CD]
	- ... is the process that automatically runs tests and builds the software—or in our case, the infrastructure.
	- The motivation behind including CI/CD is that we have a unified environment for both "I" and "D" so that no one developer's setup can be the point of failure.
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

### Applied Trunk Based Development

#### Committing

![Diagram](./img/trunk_based_development.png)

When creating a repository, always set `trunk` as the default branch (in place of what otherwise would be `main` or historically `master`) and on your Git platform of choice, set the `trunk` branch to be protected so that not just anyone can push to it.

Clone the repository locally:

```sh
git clone "git@example.org:acme/tbd.git"
```

**ENSURE YOU HAVE THE TRUNK UPSTREAM SET**:

```sh
git branch -u origin/trunk trunk
```

When making changes and starting a new branch for them, make sure that your local `trunk` is in sync with the remote. Other people can make changes to the repository, and you would waste time by having to re-base your work afterwards.

```sh
# Fetch changes on the remotes.
git fetch
# Examine changes.
git log --all --graph --oneline

# Ensure you have the trunk branch checked out.
git switch trunk

# If you had some work (accidentally) committed to trunk that isn't on the
# remote, consider making a branch for it.
git switch -c branchname
# Or if you have some changes that aren't even committed yet, you may either
# commit them...
git commit -m 'Description of commit up to 80 chars'
# ... or stash them locally.
git stash

# If there weren't any history disruptions, you may simply run the following
# command. This will set your local trunk to origin/trunk.
git pull --ff-only

# If you had committed some work that you would like to abandon, you could
# always reset.
git reset --hard origin/trunk
```

You can theoretically also do all of that with just this command, however you will still have to handle the possible case of not having pushed your local changes.

```sh
git pull --rebase
```

Now, create your off-shoot branch.

```sh
git switch -c feature/foo
```

If you would like to skip synchronizing your local `trunk` with `origin`, you can also create the branch directly off it, but it is recommended to keep things in sync.

```sh
git switch -c feature/foo origin/trunk
```

Now, make your changes and commit them as you normally would.

```sh
git add ...
git commit -m '...'
```

If you have made several commits and would like to squash them or edit their commit messages, you can run the following command. This will open an editor (probably `nano`) where there will be instructions on what you can do with it.

```sh
git rebase -i COMMIT_SHA
# The -i stands for interactive.
# The COMMIT_SHA can be found by running git log --all --graph --oneline
# and finding the commit SHA where your branch split off, like 5f271293.
# You can also use a branch name in place of the SHA, but there will be cases
# where that commit no longer has a branch name on it.
```

If in the meantime someone made changes, fetch and re-base.

```sh
git fetch
# inspect changes
git rebase origin/trunk
```

And finally push.

```sh
git push origin feature/foo
```

#### Creating a Merge Request

Whenever you push to GitLab or any other Git platform, the output of the `push` command should show something like the following

```
remote: To create a merge request for feature/non-prod-env, visit:
remote:   [https://gitlab.com/Revolgy/branching-playground/-/merge_requests/new?merge_request[source_branch]=feature%2Fnon-prod-env](https://gitlab.com/Revolgy/branching-playground/-/merge_requests/new?merge_request%5Bsource_branch%5D=feature%2Fnon-prod-env)
```

Clicking on the link should take you to the MR creation form with the important fields (source branch, target branch) already filled in.

Read the relevant chapter for [Applied code review](#applied-code-review).

Finally, when the changes have been approved and the CI pipeline has finished successfully, the Merge Request is ready.

Click the "Merge" button and the Git platform will automatically add your changes to the target branch—`trunk`.

Ideally, if there is the option to do it, make the platform also delete its version of your branch. If the option isn't there or if you simply want to do it yourself for whatever reason, run the following command:

```sh
git push --delete origin feature/foo
```

You may also want to delete your local version of it too.

```sh
# If the changes have been merged and are currently in sync.
git branch -d feature/foo

# If the changes are already out of sync but you're sure you won't lose any
# data, you may force-delete the branch.
git branch -D feature/foo
```

#### Releasing

When a major version of a product is out, it is needed to cut the new release branch which will track the life of this major version until next release will come.

Each release should be branched off the current `trunk`. So first of all make sure that you have checked out the `trunk` branch and it is up to date with the upstream:

```sh
git switch trunk
git pull --rebase
```

Now create the new release branch and push it into upstream:

```sh
git switch -c release/1
git push -u origin release/1
```

Also you can tag the very first version of the new release. **This will be a repeated action for every new released version.**

```sh
git tag release-1.0
git push --tags
```

The current `release` branch should go along with the `trunk`. So `trunk` should be periodically merged into the current `release`.

```sh
git switch trunk
git pull --rebase
git switch release/1
git merge trunk
```

## Code review (Git, GitLab, GitHub)

### Definitions

- **merge request**: (or **pull request**) on most Git platforms, a merge request is a form of keeping track of changes that are proposed to be merged into a different branch, usually the main protected branch. Usually, it can be set so that only the "maintainer" or "owner" of the repository can accept the MR, and this person can also directly request that changes be made to the MR before acceptance through discussion.

### Abstract

The code review process aims to:

- Improve readability and maintainability.
- Prevent the introduction of defects.
- Leverage the experience of other contributors for each proposed change.
- Follow compliance with standards relative to the project (OWASP, PCI...)

Code review should be done for all contributions as they are to be merged into `trunk` (or any other permanent branch) from which releases are to be made.

Nitpicking irrelevant issues, such as white-space errors, grammar mistakes, etc., is not the goal. The goal is to educate each other and learn something from the review.

Merge Requests should be set so that they cannot be accepted by the same person who wrote the code.

Merge Requests should also contain changes to the CI/CD pipeline code so that it can effectively test itself, and reviewers should verify that this has happened.

Reviewers should be at least one maintainer or owner of the code base. If at any point there is doubt that such a number or the quality of reviewers is sufficient, more may be added at any time.

Developers should go over their MR themselves as if they were the reviewer *before* it is submitted for review. This always saves time of the whole group.

When writing the MR description, always provide it. Just a few sentences in your own words can help the others with understanding what the MR is even about.

### Applied code review

#### Developer's side

Read also [Creating a Merge Request](#creating-a-merge-request) for the practicalities.

When you push your changes into a new branch and create the Merge Request, the following requirements should already be set in place—if not, set them or ask the maintainer to set them.

- A minimum amount of accepted reviews from reviewers—at the very least just one.
- CI should pass as a complete success.

When a reviewer points out that something should be changed before acceptance, generally it will not be themselves implementing that change, as that would make them a contributor of the MR and therefore not someone who can do review.

Reviewers may go into the individual lines of code and comment on them, and the review will not go through until the conversations are resolved.

#### Reviewer’s side

- **"Does this code accomplish the author’s purpose?"** Every change should have a specific reason (new feature, refactor, bug-fix, etc). "Does the submitted code actually accomplish this purpose?"
- **Ask questions.** Functions and classes should exist for a reason. When the reason is not clear to the reviewer, this may be an indication that it should be explained more thoroughly.

- **How you would have solved the problem?** If it’s different, why is that? Does your (imaginary) code handle more (edge) cases? Is it shorter/easier/cleaner/faster/safer yet functionally equivalent? Is there some underlying pattern you spotted that isn’t captured by the current code?
- **Do you see any potential for useful abstractions?** Partially duplicated code often indicates that a more abstract or general piece of functionality can be extracted and then reused in different contexts.
- **Does the change follow standard patterns?** Established codebases often exhibit patterns around naming conventions, program logic decomposition, data type definitions, etc. It is usually desirable that changes are implemented in accordance with existing patterns.
- Check for **new dependencies**. If the MR added a dependency, does it make sense? Is it necesary?

- **Reading experience.** Did you grasp the concepts in a reasonable amount of time?
- Is the **code consistent with the project in terms of style**, API conventions, etc?

- **Does this change break backward compatibility?**
- **Does this code need (integration) tests?**
- **Was the documentation of the added part updated? Is the documentation explanatory?**

- Verify that API endpoints perform appropriate authorization and authentication consistent with the rest of the code base. Check for other common weaknesses, e.g., weak configuration, malicious user input, missing log events, etc. When in doubt, refer the CR to an application security expert.
- Last but not least, praise concise/readable/efficient/elegant code. Every review request must get a well-written description of why it was or wasn't approved. We do not blame each other, we do not judge each other, we help each other to learn. Everybody makes mistakes.

## Terraform structuring and best practices (Git, Terraform)

### Standard project structure

Terraform defines a [Standard Module Structure]. Please read it, practically all modules go by it and so should we.

Some excerpts from the Standard Module Structure below.

> **`main.tf`, `variables.tf`, `outputs.tf`**. These are the recommended filenames for a minimal module, even if they're empty. `main.tf` should be the primary entrypoint. For a simple module, this may be where all the resources are created. For a complex module, resource creation may be split into multiple files but any nested module calls should be in the main file. `variables.tf` and `outputs.tf` should contain the declarations for variables and outputs, respectively.

Terraform also automatically loads a number of variable definitions files if they are present:

- Files named exactly `terraform.tfvars` or `terraform.tfvars.json`.
- Any files with names ending in `.auto.tfvars` or `.auto.tfvars.json`.

Always include also a `versions.tf` file to set in ~~stone~~ file the versions of providers and of Terraform itself.

```hcl
# Figure out current versions by running `terraform version`

terraform {
  required_version = "~> 1.9.0"

  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.35.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "5.35.0"
    }
    null = {
      source  = "hashicorp/null"
      version = "3.2.2"
    }
    random = {
      source  = "hashicorp/random"
      version = "3.6.2"
    }
    time = {
      source  = "hashicorp/time"
      version = "0.11.2"
    }
  }
}
```

> **Variables and outputs should have descriptions.** All variables and outputs should have one or two sentence descriptions that explain their purpose. This is used for documentation.

> Nested modules. Nested modules should exist under the modules/ subdirectory. Any nested module with a README.md is considered usable by an external user. If a README doesn't exist, it is considered for internal use only. These are purely advisory; Terraform will not actively deny usage of internal modules. Nested modules should be used to split complex behavior into multiple small modules that advanced users can carefully pick and choose. For example, the Consul module has a nested module for creating the Cluster that is separate from the module to setup necessary IAM policies. This allows a user to bring in their own IAM policy choices.
> 
>If the root module includes calls to nested modules, they should use relative paths like ./modules/consul-cluster so that Terraform will consider them to be part of the same repository or package, rather than downloading them again separately.
>
>If a repository or package contains multiple nested modules, they should ideally be composable by the caller, rather than calling directly to each other and creating a deeply-nested tree of modules.

> **Examples**. Examples of using the module should exist under the `examples/` subdirectory at the root of the repository. Each example may have a README to explain the goal and usage of the example. Examples for submodules should also be placed in the root `examples/` directory.
> 
> Because examples will often be copied into other repositories for customization, any `module` blocks should have their `source` set to the address an external caller would use, not to a relative path.

### `README.md`

**Every** Terraform code base, regardless of whether it is a project or a module, should have a README.

Use the [`terraform-docs`][terraform-docs] command to generate the strictly technical parts of it. This process can (and should) be automated. [Link](https://github.com/terraform-docs/terraform-docs/blob/master/docs/user-guide/introduction.md)

```sh
terraform-docs markdown . > README.md
```

### Using Upstream Modules

Upstream modules (modules downloaded from the internet) **should not be referenced** directly.

When planning to use a upstream module, one should fetch the module and store it in either git repository for upstream modules or a new repository created specifically for the module, depending on poly-repo or mono-repo preferences. 

This eases version management.

### Including modules

Always include modules through Terraform itself. **Do not use Git submodules**, they are a pain to work with.

There is a lot of ways to include a module, depending on what your needs are.

```hcl
module "local" {
	source = "./modules/foo"
}
```

```hcl
module "git_over_https" {
	source = "git::https://example.com/foo.git?ref=v1.2.0"
}
```

```hcl
module "git_over_ssh" {
	source = "git::git@github.com:example/foo.git//path/within/repo"
}
```

### `.gitignore`

Always include a `.gitignore` file in a Terraform codebase, but **do not ignore the `.terraform.lock.hcl` file!** It's the equivalent of a NodeJS package lock, it makes the included modules have an exactly fixed version.

```
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files, which are likely to contain sensitive data, such as
# password, private keys, and other secrets. These should not be part of version 
# control as they are data points which are potentially sensitive and subject 
# to change depending on the environment.
*.tfvars
*.tfvars.json
!terraform.tfvars

# Ignore override files as they are usually used to override resources locally and so
# are not checked in
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Include override files you do wish to add to version control using negated pattern
# !example_override.tf

# Include tfplan files to ignore the plan output of command: terraform plan -out=tfplan
# example: *tfplan*

# Ignore CLI configuration files
.terraformrc
terraform.rc

```

### Remote state

Avoid use of specific non-standard magic glue solutions, such as `Makefile`s, `bash` scripts, etc.

Options for remote state:

- One environment = one account + shared account for state
- Multiple environments in one account differentiated either on VPC level or via tags etc...

### CI/CD

In CI/CD, always conform to the best practices for both performance and usability on your platform of choice.

Check every commit with fast and cheap tests, such as `terraform fmt`.

Check every commit *that is going into trunk or a release branch* with `terraform validate`.

Perform as many repeating tasks automatically, with the exception of `terraform apply` unless you're confident.

### Optimization for CI/CD performance

If possible, cache the `.terraform` directory in your CI/CD system, as it stores useful files, which Terraform will re-download when appropriate.

Do not use artifact storage for `.terraform`. Only use artifact storage for the plan file.

### Example `.gitlab-ci.yml`

**Warning: Update container versions with the file before use!**

**Note the environment variables in the file, they need to be filled in!**

This GitLab CI document deploys a Terraform codebase to 3 environments (as defined in GitLab), and it is set to validate each and every commit via the first few stages.

- `lint`
	- Also includes a check for `README.md` being up-to-date via `terraform-docs`.
	- Checks if the files are formatted, effectively ruling out some very bad commits from the get-go as the formatter needs valid code.
- `init`
	- This `init` doesn't actually interact with the backend, as it would lock it for some time, which would be suboptimal for high usage repositories. It is mainly for `validate`, there is another `terraform init` during `plan`
- `validate`
	- Runs a deeper validation of the code itself. It checks for things like wrong variable names, missing required arguments, etc.

Caching of `.terraform` is also already in place.

- `plan` - **Runs only in Merge Requests to GL environments!**
	- Runs a `terraform init` against the real backend.
	- Runs `terraform plan`.
	- Generates a report of the plan in a JSON format for easy viewing in GitLab.
- `apply` - **Runs only after apply.**
	- Manual step.
	- Runs `terraform apply` with the plan from before.

```yaml
---

stages:
  # Run quick checks if everything is formatted and updated
  - lint

  # Initialize Terraform working directory
  - init

  # Runs a deeper lint
  - validate

  # Creates a plan for deployment
  - plan

  # Applies deployment (has to be approved manually)
  - apply

variables:
  TF_PLAN_FILE: tf_plan_file
  JSON_PLAN_FILE: tf_plan.json
  TERRAFORM_TF_VARS_FILE: terraform.tfvars

image:
  name: hashicorp/terraform:1.7.3
  entrypoint:
    - '/usr/bin/env'
    - 'PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'

cache:
  paths:
    - .terraform
    - .terraform.lock.hcl

# Stages

# Checks if README.md is up-to-date.
# Update it by running `terraform-docs .` and committing README.md.
terraform-docs:
  image:
    name: quay.io/terraform-docs/terraform-docs:0.16.0
    entrypoint:
      - '/usr/bin/env'
  stage: lint
  dependencies: []
  needs: []
  script:
    - terraform-docs . --output-check

# Checks if Terraform code is formatted using `terraform fmt`.
terraform-fmt:
  stage: lint
  dependencies: []
  needs: []
  script:
    - terraform fmt -no-color -check

# Initializes working directory by "creating initial files, loading any remote
# state, downloading modules".
# But it won't be loading any remote state due to -backend=false
init:
  stage: init
  before_script:
    - terraform version
  script:
    - terraform init -no-color -backend=false -upgrade

# Validate the configuration files in a directory, referring only to the
# configuration and not accessing any remote services such as remote state,
# provider APIs, etc.
validate:
  stage: validate
  dependencies: ["init"]
  script:
    - terraform validate -no-color .

# Generates a speculative execution plan, showing what actions Terraform
# would take to apply the current configuration. This command will not
# actually perform the planned actions.
.plan:
  stage: plan
  dependencies: ["init"]
  before_script:
    - apk --no-cache add jq
  script:
    - terraform init -no-color -reconfigure -backend-config="bucket=$ENV_TF_BACKEND_BUCKET" -backend-config="prefix=$ENV_TF_BACKEND_PREFIX"
    - terraform plan -no-color -var-file=$TERRAFORM_TF_VARS_FILE -var-file=$ENV_TF_VARS_FILE -out=$TF_PLAN_FILE
    - terraform show -no-color --json $TF_PLAN_FILE | jq -r '([.resource_changes[]?.change.actions?]|flatten)|{"create":(map(select(.=="create"))|length),"update":(map(select(.=="update"))|length),"delete":(map(select(.=="delete"))|length)}' > $JSON_PLAN_FILE
  artifacts:
    reports:
      terraform: $JSON_PLAN_FILE
    paths:
      - $TF_PLAN_FILE
    expire_in: 1 week
  rules:
    - if: $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == $CI_ENVIRONMENT_NAME
      allow_failure: false
    - if: $CI_COMMIT_BRANCH == $CI_ENVIRONMENT_NAME
      allow_failure: false

plan-dev:
  extends: .plan
  environment: dev

plan-stg:
  extends: .plan
  environment: stg

plan-prd:
  extends: .plan
  environment: prd

# Creates or updates infrastructure according to Terraform configuration
# files in the current directory.
.apply:
  stage: apply
  script:
    - terraform apply -no-color -auto-approve -input=false $TF_PLAN_FILE
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_ENVIRONMENT_NAME
      when: manual
      allow_failure: false

apply-dev:
  extends: .apply
  dependencies: ["init", "plan-dev"]
  environment: dev

apply-stg:
  extends: .apply
  dependencies: ["init", "plan-stg"]
  environment: stg

apply-prd:
  extends: .apply
  dependencies: ["init", "plan-prd"]
  environment: prd
```

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
[Standard Module Structure]: https://developer.hashicorp.com/terraform/language/modules/develop/structure