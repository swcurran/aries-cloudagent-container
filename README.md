[![img](https://img.shields.io/badge/Lifecycle-Stable-97ca00)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

# Docker images for the OpenWallet Foundation's ACA-Py Agent

This repository creates and pushes images of the [OpenWallet
Foundation](https://openwallet.foundation/)'s ACA-Py to the BC Gov Docker Hub
image repository. The image repository is located [on Docker
Hub](https://hub.docker.com/r/bcgovimages/aries-cloudagent/).

The images include implementations of ACA-Py agent :

-   [Hyperledger Aries CloudAgent Python (ACA-Py)](https://github.com/hyperledger/aries-cloudagent-python) - up to Release 1.0.1
-   [OpenWallet Foundation ACA_Py](https://github.com/openwallet-foundation/acapy) - Release 1.1.0 and after

# Repository structure

The repository is structured so that there is a folder for each implementation/language, and this contains the dockerfiles and resources required to build and test the output image.

Inside [make_image.py](make_image.py#L10) a data structure called `VERSIONS` describes the versions and base image for each implementation/language.

# Adding a new implementation/language

To add a new implementation/language:

1. create a new folder named after the new implementation in the root of the repository. For reference, the [python](python) folder can be copied and its contents updated to reflect the new implementation/language.
2. Update the `VERSIONS` object in [make_image.py](make_image.py#L10) to add the new implementation/language details.

## Image versioning

All the `aries-cloudagent` images will reside on the [Aries CloudAgent Docker Hub](https://hub.docker.com/r/bcgovimages/aries-cloudagent/), and will differentiate between each other by using specific tags. Even after the move of ACA-Py from [Hyperledger](https://hyperledger.org) (where it was called "Aries Cloud Agent - Python") to the [OpenWallet Foundation](https://openwallet.foundation/) (where it is called "ACA-Py"), the image name remains "aries-cloudagent" in the image repository.

By default, the tag for a new image will be composed by `$base_image` (minus `-$version`, if present), followed by `_$version`, where both values are from the `VERSIONS` JSON structure in [make_image.py](make_image.py#L10). For example:

- `$base_image`: `"ghcr.io/openwallet-foundation/acapy-agent:py3.12-1.2.0"`
- `$version`: `1.2.0`
- Tag: `py3.12_1.2.0`

Because of this naming convention, please make sure you use a tagged version for each base image rather than using `latest` or a more generic tag that is not as descriptive.

- Up until release 0.9.0 (and 1.0.0-rc3), the base image came from `von-images`
and used Python 3.6.
- After those versions and through version 1.0.1, the base
image comes from the released container image from ACA-Py CI/CD publication
process that pushes to Hyperledger's [GitHub Container Repository](ghcr.io)
based on Python 3.9.
- From Release 1.1.0, the base images from the released
container image from ACA-Py CI/CD publication process that pushes to OpenWallet
Foundations's [GitHub Container Repository](ghcr.io).

As such, `bcgovimages` images are now just republications of the same images
that are published from the ACA-Py CI/CD pipeline. We are retaining them because
of the large user base still using pulling ACA-Py from the `bcgovimages`
repository. We recommend those still using the `bcgovimages` repository update
their Dockerfiles to use the OpenWallet Foundations's [GitHub Container
Repository](ghcr.io) images.

:::note

The Python module name changed from "aries-cloudagent" to "acapy-agent" when the
project was moved from [Hyperledger](https://hyperledger.org) to the [OpenWallet
Foundation](https://openwallet.foundation/). Those using the images published
from here to the BC Gov Docker Hub organization must change the Python `import`
in their code to reflect that module name change. We recommend as well changing
the image source from being the BC Gov Docker Hub organization to the OpenWallet
Foundations's [GitHub Container Repository](ghcr.io), as listed above.

As a result of the module name change, images prior to 1.1.0 **cannot** be built
using the scripts in this repo. In theory, the logic could be updated to enable
that, but we don't think it is worth the effort. Add an issue if you need to be
able to do that.

:::

# Building the image locally

This project is setup to use **Visual Studio Code Remote - Containers**.  The devContainer includes everything you need to work on the project and build the images.  The container uses docker-from-docker so any images you build will show up in your host's docker repository.

## Pre-requisites

Using the devContainer:

- Visual Studio Code with the Remote - Containers extension installed.
- [Docker](https://www.docker.com/)

To develop and build the image locally you will need to install:

- [Python 3](https://www.python.org/)
- [Docker](https://www.docker.com/)

## Running the build

To build the image, open a terminal session at the root of this git repo and execute: `python make_image.py 0.3.0 python`,
replacing `0.3.0` with the ACA-Py version you need.

Additional parameters can be specified through command-line, for more information please type `python make_image.py --help` to display the command's usage page.

## Building an image from a GitHub source

In some cases you may want to build an image from a particular repository and branch.  You can do this by adding the `git_egg_ref` build-arg to the command line.  In this case the version number you specify will only be used to determine the relevant `base_image` and `acapy_reqs` to use for the build.

The format of the `git_egg_ref` argument is:
- `git+`**<git_repo_url>**[`@`**<git_ref>**]`#egg=`
  - Where:
    - **<git_repo_url>**: Is the url to the repo including the trailing `.git` extension.
    - **<git_ref>**: Optional.  The `git_ref` of the branch or tag to use.  If specified, the `<git_repo_url>` and `<git_ref>` must be separated by a `@`.

Example:
```
./make_image.py 0.7.3 python --build-arg git_egg_ref="git+https://github.com/WadeBarnes/aries-cloudagent-python.git@persistant_queue#egg="
```

## Adding and Publishing a New Image

To add a new version of ACA=Py to this repo, create a PR with the following changes:

- Decide on the base image to be used for the image, and the version of ACA-Py.
  - The image tag of the output will be a concatenation of the two, as [described above](#image-versioning).
- Edit the [make_image.py](make_image.py) file in this repo to add the new version.
- Create and test the build of the local image using the options `--no-cache` and `--test`
  - `python make_image.py 1.2.0 --no-cache --test python`
- If successful, push the image to the bcgovimages organization of Docker Hub using the `--push` option
  - `python make_image.py 1.2.0 --push python`
- If successful, verify the publishing of the image by checking [Docker Hub](https://hub.docker.com/r/bcgovimages/aries-cloudagent/tags)
  - Note that it make take a few minutes for the new tag to appear. Hit refresh periodically until the image is visible.

# Credits

The build scripts in this repository have taken inspiration from the ones used to build [von-image](https://github.com/PSPC-SPAC-buyandsell/von-image)
