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

To create a new release of OrecchietTetris, the following procedure should be followed:

1. Create a dedicated release branch if necessary:

```bash
git checkout -b release/X.Y.Z
```

2. Update the project version according to the Semantic Versioning rules.

3. Update the `CHANGELOG.md` file with the changes included in the new release.

4. Commit and merge the changes into the main branch:

```bash
git add .
git commit -m "Prepare release X.Y.Z"
git push
```

5. Create an annotated Git tag:

```bash
git tag -a X.Y.Z -m "Release X.Y.Z"
```

6. Push the tag to GitHub:

```bash
git push origin X.Y.Z
```

7. The GitHub Actions deployment workflow is automatically triggered, generating the release artefacts, publishing them to the configured repositories, and creating the corresponding GitHub Release.

