---
title: "Branch Descriptions"
category: "development"
displayOrder: 0
version: "v3"
---

This page gives an overview on the most important branches of the Duality repository.

* TOC
{:toc}

# master

The main development branch of Duality, where bugs are fixed and new features introduced. For bigger changes, development will be done in a feature branch from the latest master version in order to keep it operational at all times.

# release

Whenever a new binary release is done, the master branch is merged into the release branch. That way, the source code used for compiling the last release remains easily accessible without manually plowing through commits. Also, continuous integration services can watch release in order to automatically create build artifacts such as the main installer .zip file and NuGet packages.

# 2.x, 1.x

Stale branches that archive the last Duality source version before a major version step. Future version steps will likely spawn their own archive branches.