---
title: "Branch Descriptions"
category: "contributing"
displayOrder: 0
version: "v2"
---

This page gives an overview on the most important branches of the Duality repository.

# master

The main development branch of Duality, where bugs are fixed and new features introduced. For bigger changes, development will be done in a temporary branch from the latest master version in order to keep it operational at all times.

# release

Whenever a new binary release is done, the master branch is merged into the release branch. That way, the source code used for compiling the last release remains easily accessible without manually plowing through commits. Also, continuous integration services can watch release in order to automatically create build artifacts such as the main installer .zip file and NuGet packages.

# develop-3.0

Ongoing development on the [next major version step](https://github.com/AdamsLair/duality/milestone/6) for Duality. Due to the amount and impact of breaking changes in this branch, there are some [special restrictions](https://github.com/AdamsLair/duality/wiki/Development-Cycle) applying.

# 1.x

A stale branch that archives the last Duality source version before the major version step to v2.x. Future version steps will likely spawn their own archive branches.