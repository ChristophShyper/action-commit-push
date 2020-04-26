# GitHub Action for committing changes to a repository

**GitHub Action that will create a new commit and push it back to the repository.**

Dockerized as [christophshyper/action-commit-push](https://hub.docker.com/repository/docker/christophshyper/action-commit-push).

Features:
* Can add a custom prefix to commit message by setting `commit_prefix`.
* Can create a new branch when `target_branch` is set.  
* Can add a timestamp to a branch name, when `target_branch` is set and `add_timestamp` is `true`. Will create a branch named `${branch_name}/${add_timestamp}`. Great for cron-based updates.
* Good to combine with my other action [devops-infra/action-pull-request](https://github.com/devops-infra/action-pull-request).


## Badge swag
[![Master branch](https://github.com/devops-infra/action-commit-push/workflows/Master%20branch/badge.svg)](https://github.com/devops-infra/action-commit-push/actions?query=workflow%3A%22Master+branch%22)
[![Other branches](https://github.com/devops-infra/action-commit-push/workflows/Other%20branches/badge.svg)](https://github.com/devops-infra/action-commit-push/actions?query=workflow%3A%22Other+branches%22)
<br>
[
![GitHub repo](https://img.shields.io/badge/GitHub-devops--infra%2Faction--commit--push-blueviolet.svg?style=plastic&logo=github)
![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/devops-infra/action-commit-push?color=blueviolet&label=Code%20size&style=plastic&logo=github)
![GitHub last commit](https://img.shields.io/github/last-commit/devops-infra/action-commit-push?color=blueviolet&logo=github&style=plastic&label=Last%20commit)
![GitHub license](https://img.shields.io/github/license/devops-infra/action-commit-push?color=blueviolet&logo=github&style=plastic&label=License)
](https://github.com/devops-infra/action-commit-push "shields.io")
<br>
[
![DockerHub](https://img.shields.io/badge/DockerHub-christophshyper%2Faction--commit--push-blue.svg?style=plastic&logo=docker)
![Docker version](https://img.shields.io/docker/v/christophshyper/action-commit-push?color=blue&label=Version&logo=docker&style=plastic)
![Image size](https://img.shields.io/docker/image-size/christophshyper/action-commit-push/latest?label=Image%20size&style=plastic&logo=docker)
![Docker Pulls](https://img.shields.io/docker/pulls/christophshyper/action-commit-push?color=blue&label=Pulls&logo=docker&style=plastic)
](https://hub.docker.com/r/christophshyper/action-commit-push "shields.io")


## Reference

```yaml
    - name: Run the Action
      uses: devops-infra/action-commit-push@master
      with:
        github_token: "${{ secrets.GITHUB_TOKEN }}"
        add_timestamp: true
        commit_prefix: "[AUTO]"
        target_branch: update/version
```


Input Variable | Required | Default |Description
:--- | :---: | :---: | :---
github_token | Yes | `""` | Personal Access Token for GitHub for pushing the code.
add_timestamp | No | `false` | Whether to add the timestamp to a new branch name. Used when `target_branch` is set. Uses format `%Y-%m-%dT%H-%M-%SZ`.
commit_prefix | No | `[AUTO-COMMIT]` | Prefix added to commit message.
target_branch | No | *current branch* | Name of a new branch to push the code into. Creates branch if not existing.

Outputs | Description
:--- | :---
files_changed | List of changed files. As returned by `git diff --staged --name-status`.
branch_name | Name of the branch code was pushed into.


## Examples

Commit and push changes to currently checked out branch.
```yaml
name: Push changes
on:
  push
jobs:
  change-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Change something
        run: |
          find . -type f -name "*" -print0 | xargs -0 sed -i "s/foo/bar/g"
      - name: Commit and push changes
        uses: devops-infra/action-commit-push@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

Commit and push changes to a new branch and create pull request using my other action [devops-infra/action-pull-request](https://github.com/devops-infra/action-pull-request).
```yaml
name: Push changes
on:
  push
jobs:
  change-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Change something
        run: |
          find . -type f -name "*" -print0 | xargs -0 sed -i "s/foo/bar/g"
      - name: Commit and push changes
        uses: devops-infra/action-commit-push@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          commit_prefix: "[AUTO-COMMIT] foo/bar replace"
      - name: Create pull request
        uses: devops-infra/action-pull-request@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          body: "**Automated pull request**<br><br>Replaced foo/bar"
          title: ${{ github.event.commits[0].message }}
```
