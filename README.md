- [🌲 Trunk-based development](#-trunk-based-development)
  - [Overview](#overview)
  - [Trunk-based development flow](#trunk-based-development-flow)
- [👀 Code review](#-code-review)
- [👷 Terraform best practices](#-terraform-best-practices)

Revolgy Delivery team way of work. We create own our guidelines. We change them if they do not fit our work anymore.

## 🌲 Trunk-based development
`Trunk` is a main rolling branch. Releases are cut from the `trunk` branch. There are also several approaches on how to cut releases with trunk-based workflow: e.g. [from the `trunk` directly](https://trunkbaseddevelopment.com/release-from-trunk/) or [from separate release branch](https://trunkbaseddevelopment.com/branch-for-release/).

### Overview

- There are no `feature` or `hotfix` branches, only one continuous branch exists and it is `trunk`. Every change is being developed in its own temporary branch created off `trunk`.
- Any change cannot be directly merged without testing, tests executed and the test coverage must be part of MR/PR.
- The developer or engineer does not have to pick a correct branch to start his/her work: for each feature, fix, whatever we create a temporary branch with a descriptive name (e.g. `feature:mTLS_support` or `chore:dependencies_cleanup`) deleted after the tests and successful merge approval.
- `trunk` is an active rolling branch with the latest code which can (and should) be continuously deployed into the **testing** environment.
- When we cut a new release, a `release` branch is created and the release commit is being tagged with a version number. Stick to [semantic versioning](https://semver.org/). We follow this logic in versioning {MAJOR**.**MINOR**.**PATCH}.
  - Semantic versioning rules:
      1. MAJOR version when you make significant release changes,
      2. MINOR version when you add functionality in a backward-compatible manner. In Revolgy context this is new feature addition like a new monitoring service eg. Splunk in Terraform.
      3. PATCH version when you make backward-compatible bug fixes. Here we count also hotfixes.
- Tagged releases can be then deployed by CI/CD pipeline.
- `trunk` branch is periodically merged into the current `release` and when a new version of the current release should be issued, appropriate commit is being tagged with new version number.
- if we run into a situation when we need to backport (or sync-back) something into an older release branch which is not in sync with `trunk` anymore, we cherry-pick a single commit from the `trunk` and tag a new version of this older release.

Related patterns of behavior for code management and branching:

- Peer review is mandatory and should be done thoroughly and responsibly
- Each MR should be continuously tested
- squash and merge all the MRs, again with reasonable commit message.
  - except when syncing `trunk` into `release` (to preserve history for potential cherry-picking)
- MR should contain reasonable description. See the template:

```markdown
	This MR contains a feature set regarding the new API update for WAF v2.0 with new rules for CoolClient.io.
> ⚠️ NOTE: IMPORTANT SHORT MESSAGE - DELETES, OPS NOTE ETC.

## Merge type
| Status  | Type  | Env Vars Change | Traking issue |
| :---: | :---: | :---: | :--: | :--: |
| New/Update| Feature/Hotfix/Release | Yes/No | [Link](<ticket link here>) |


## Description
Bla bla bla I am going to the spa.

## Feature / Hotfix list included
- feature A
- feature B
- feature C
- Added new rule pack for Cross-site request forgery attack
- Update for existing ACL

## Deployment notes
Notes regarding deployment of the contained body of work. These should note any new dependencies, new scripts, etc.

**New environment variables:**
- env var : env var Staging variable for Rules

**New scripts:**
- script : script details

**New dependencies:**
- dependency : dependency details
```
### Trunk-based development flow

The diagram above shows a complete life cycle of a project managed with trunk based branching flow. Let's take a look at each step in detail. For instance, we are working with project [https://gitlab.com/Revolgy/branching-playground](https://gitlab.com/Revolgy/branching-playground)

1. Implement change (no matter if it is a feature or fix)
    - If you haven't cloned the repository yet, let's do that: `git clone[git@gitlab.com](mailto:git@gitlab.com):Revolgy/branching-playground.git`
    and then set default upstream to `trunk`: `git branch -u origin/trunk trunk`
    - Now when you have the repository cloned and default upstream is set to trunk, create your branch: `git checkout -b feature/non-prod-env` and implement the change you've planned.
    - After you are happy with the implementation, commit the change with a meaningful comment: `git commit -m "First environment added"`
    - And push it into upstream: `git push origin feature/non-prod-env`
    - In the output of git command you'll get a hyperlink for creating a  merge request:

        ```markdown
        remote:
        remote: To create a merge request for feature/non-prod-env, visit:
        remote:   [https://gitlab.com/Revolgy/branching-playground/-/merge_requests/new?merge_request[source_branch]=feature%2Fnon-prod-env](https://gitlab.com/Revolgy/branching-playground/-/merge_requests/new?merge_request%5Bsource_branch%5D=feature%2Fnon-prod-env)
        remote:
        ```

    - When creating merge request please fill in Description thoroughly. Also chose relevant reviewers and option to squash commits when MR is accepted.
    - After merge request is accepted, your change will be in the `trunk` branch
2. The second step in our diagram is initializing a new Release branch. **When a major version of a product is out, it is needed to cut the new Release branch which will track the life of this major version until next Release will come**.
    - Each release should be branched off the current trunk. So first of all make sure that you have checked out the `trunk` branch and it is up to date with the upstream:

        ```markdown
        git checkout trunk
        git pull -r
        ```

    - Now create the new Release branch and push it into upstream:

        ```markdown
        git checkout -b release-1
        git push origin release-1
        ```

    - Also you can tag the very first version of the new release. This will be a repeated action for every new released version (Step 4 in the diagram above).

        ```markdown
        git tag release-1.0
        git push origin release-1.0
        ```

        You can also just push tags instead of specifying `release-1.0`: `git push --tags`

3. The current `release` branch should go along with the `trunk`.  So `trunk` should be periodically merged into the current `release`.
    - Make sure that you've got synced `trunk` branch:

        ```markdown
        git checkout trunk
        git pull -r
        ```

    - Now checkout current `release` branch and merge `trunk` into it.

        ```markdown
        git checkout release-1
        git merge trunk
        ```

4. Cut new Release by tagging relevant commit in the current `release` branch, This step has been done in step 2 while creating the first release. Any other release is no different. Just checkout the branch and tag a commit.
Let's say you want to sync up `trunk` with current `release-1` and create a new release which will be `1.2`

    ```markdown
    git checkout release-1
    git merge trunk
    git tag release-1.2
    git push --tags
    ```

5. The last step in the diagram is backporting a change from the current release to the older one. This option will be used very seldom. For instance, if we fix a critical bug in the current release, but older release which we still support happens to have the same bug.
In such a case, we can not just merge `trunk` into the older `release` branch because it will lead to merging the whole history and our older `release` will become equal to `trunk`. That is not what we want to achieve. So we should use [git-cherry-picking](https://git-scm.com/docs/git-cherry-pick) to merge a single commit into an older `release`.
    - Get sha hash of the commit you want to back-port. It can be found in Gitlab of by looking through `git log` output. E.g.:

        ```markdown
        commit 9c8b813348bd34d6b974c929dac338d55344b5a5 (HEAD -> release-2, tag: release-2.1, origin/trunk, origin/HEAD, trunk)
        Author: Anton Vorobiev <anv@revolgy.com>
        Date:   Mon Dec 28 09:48:44 2020 +0100

            Resize staging environment to match production
        ```

        So in this case commit hash we are interested in is `9c8b813348bd34d6b974c929dac338d55344b5a5`

    - Now simply checkout the branch you want to merge the change and invoke the `git cherry-pick` command. Let's say our current release branch is `release-2` and you want to merge the change into `release-1`.

        ```markdown
        git checkout release-1
        git pull -r
        git cherry-pick 9c8b813348bd34d6b974c929dac338d55344b5a5
        git push origin release-1
        ```

        Now after invoking `git log` you will see commit `9c8b813348bd34d6b974c929dac338d55344b5a5` on top of `release-1` branch without any other history from `trunk` or newer release branch.

        New version with that fix can be now released by procedure described in step 4.

## 👀 Code review
TBD

## 👷 Terraform best practices
TBD
