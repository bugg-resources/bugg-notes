# buggd

buggd is a Python package

It includes: 
* buggd, the recording daemon
* soundcardctl - a utility for manually controlling the soundcard, handy for dev, etc.
* modemctl - a utility for manually controlling the modem, handy for dev, etc.

# How to release

1) Set the version number (see below)
2) Build the package by running build_package.sh in the repo
3) cont..

## Version numbering

buggd using Semantic Versioning  (semver.org)

The version number is set in the pyproject.toml file.
From there, it gets automatically included into the Debian package build, including the changelog file.

Versions should be manually tagged in the git repo, in annotated form. See the first tag in the repo for an example. 

# BuggOS

BuggOS is a custom build of Raspberry Pi OS Lite.

It is built using pi-gen, driven by a Github Actions workflow.

## How to release
1) Decide what version of RPi OS to build from, by adkusting the "ref:" field in the pi-gen section of the .github/workflow file. Currently we're on "ref: 2024-03-12-raspios-bookworm-arm64". It has to be the arm64 branch.
2) Get an Ubuntu (/Debian?) machine (preferably aarch64 for fast build, or maybe kvm emulated ARM)
3) Check out the buggOS repo
4) Run `apt build-dep .` in the repo root to install the build system dependencies
5) cont
6) 