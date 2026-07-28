---
title: Release
has_children: false
nav_order: 7
---

# Release

The OrecchietTetris codebase produces two release artefacts:

- Source distribution (`OrecchietTetris-<version>.tar.gz`), which contains the source code and all the files required to build the project.
- Wheel distribution (`OrecchietTetris-<version>-py3-none-any.whl`), which is the pre-built package that can be installed directly by users.

Both artefacts are generated through the Poetry build system using the following command:

```bash
poetry build
```

The generated files are stored inside the `dist/` directory.

The artefacts are released on:

- PyPI, which provides the official distribution channel for Python packages.
- TestPyPI, which is used to validate the release process before publishing to the official repository.
- GitHub Releases, which allows users to download a specific version of the project together with the release notes.

In addition, the project's `CHANGELOG.md` file is updated to document the changes introduced in each release.

The release process is automated through a CI/CD pipeline; such workflow will be further explained in the [CI/CD](../08-cicd/) section.

## Choice of the license

Both the source code and the released artefacts are distributed under the MIT License. This license was chosen because it is simple, permissive, and widely adopted within the open-source community. It allows anyone to use, modify, distribute, and even commercially reuse the project while requiring only the preservation of the original copyright notice. As an educational project, OrecchietTetris benefits from a license that maximizes accessibility, reuse, and collaboration while imposing minimal restrictions on future users and contributors.

## Choice of the versioning schema

OrecchietTetris adopts Semantic Versioning (SemVer), using the format:

```text
MAJOR.MINOR.PATCH
```

Semantic Versioning clearly communicates the nature of the changes introduced in each release:

- `MAJOR` version: incremented when incompatible or breaking changes are introduced.
- `MINOR` version: incremented when new functionality is added in a backward-compatible manner.
- `PATCH` version: incremented when bugs are fixed or minor improvements are made without introducing new features.

Since all release artefacts are generated from the same codebase, they always share the same version number.

### Creating a New Release

As stated earlier, the CI/CD workflow is in charge of: updating the application version and the changelog, releasing the new version and creating the tag of the new version. An update of the application is released everytime a commit is pushed or a branch is merged to the branch `master` and the changes committed are so that an upgrade of the version is necessary. The last condition is also automated by the application of the Conventional Commit specification, as described in [Development](../04-development/) section; in this way, the level of the version change depends on the type of the commit. The version is upgraded accordingly to the highest level of change brought by the commits pushed:

- a `fix` commit leads to a `PATCH` level update;
- a `feat` commit leads to a `MINOR` level update;
- a `BREAKING CHANGE` commit leads to a `MAJOR` level update.

Any other type of commit will not update the version at any level.
