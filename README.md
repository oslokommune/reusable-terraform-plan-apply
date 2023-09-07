# reusable-terraform-plan-apply

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-%23FE5196?logo=conventionalcommits&logoColor=white)](https://conventionalcommits.org)

This repository contains a [reusable workflow for running Terraform](https://docs.github.com/en/actions/using-workflows/reusing-workflows):

> Rather than copying and pasting from one workflow to another, you can make workflows reusable. You and anyone with access to the reusable workflow can then call the reusable workflow from another workflow.

<p align="center">
◊
</p>

<p align="center">
  <img width="50%" src="https://user-images.githubusercontent.com/1691190/210278921-9023355d-e703-4a1b-a1d8-0afd95bc412b.png">
</p>


## Documentation

[Visit `km.oslo.systems`](https://km.oslo.systems/setup/ci-cd/deploy-image/) for setup guidance.

## Why use this workflow?

- Easier to start with than hand-building all the GitHub Actions into a single workflow.
- Gives you inputs so you can reuse the workflows across many repositories and only needing the full workflow stored in a central repository.
- A lot of effort have been put into writing clear and concise steps and step summaries that are easy to follow.
- The workflows are written with great attention to detail. Edge cases are handled gracefully.
- The majority of linting and dependency management is handled for you.
- [Security best practices](https://km.oslo.systems/security.html#github-actions) are followed.
- Collaborate with [Team Kjøremiljø](https://github.com/orgs/oslokommune/teams/kjoremiljo/members)!
- [Share improvements and ideas](https://github.com/oslokommune/reusable-workflows/discussions)!


## Versioning

[Conventional Commits](https://www.conventionalcommits.org) are used together with [`svu` (Semantic Version Util)](https://github.com/caarlos0/svu) to provide [Semantic Versioning](https://semver.org/) for the repository as a whole.
