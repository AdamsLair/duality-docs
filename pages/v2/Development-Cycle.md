---
title: "Development Cycle"
category: "contributing"
displayOrder: 0
version: "v2"
---

This page will give you an overview on the development cycle of Duality, including when (and for what) to create branches and when to merge them in which way. For more information about specific existing branches, please refer to the [Branch Descriptions](Branch-Descriptions.md) page.

# Ongoing Development

Bugfixes and minor features are developed in the `master` branch. **Breaking changes are not allowed** and the head of the branch is required to always represent a complete, running, non-WiP version of Duality. Until a feature or bugfix is complete, do not push to `master`, as this would put the head into a WiP state. If a task is too big to be completed in a single run, your options are to either work locally or create a feature branch to work on. 

Occasionally, the `master` branch serves as the basis for new binary package releases.

## Deploying a Binary Release

Binary deployment is not usually a task you will face - except if you're a Duality core developer. Here's the basic process outline.

### Updating Package Specs

In order to deploy a new package release, update all `AssemblyInfo.cs` and `.nuspec` files to their new [semantic version](http://semver.org/) number. Most of the time, this can be done automatically using the [Update Package Versions script](https://github.com/AdamsLair/duality/tree/master/Build/Scripts), which will lead you through an interactive process of reviewing changes since the last package update, updating version numbers and finally creating a new git commit with the package update. However, if you updated a dependency that is not itself part of the main repository, make sure to **update the .nuspec dependency entry** as well - prior to executing the above script. Failing to do so will break the binary release of Duality for both updaters and new users.

### Merging into `Release`

After updating the package specs, the next step is to merge `master` into `release` and pushing both branches. The continuous integration service AppVeyor will automatically build both latest commits and run unit tests. If they turn green and you've spent enough time testing, you're ready for the deployment.

### Deploying Packages

To deploy a new set of Duality packages and release them into the wild, [log into AppVeyor](https://ci.appveyor.com/projects) using your GitHub account. When prompted, select `AdamsLairBot` as a user role. _Note: In order to be able to do this, you'll have to be authorized._ Open `Environments` and select `Duality NuGet Package Release`. Click `New Deployment` and select the latest `release` build. You should recognize it as your latest merge commit. A console view will appear that shows the deployment progress - if it finishes without errors, your work is done here.

As an additional counter-measure to broken binary releases, it is always a good idea to check if the binary install still works as expected. First, open the [NuGet page](https://www.nuget.org/packages/AdamsLair.Duality) of the packages you just deployed and wait until the "not indexed yet" notice disappears. When that happened, download Duality and let it install. Select one or two sample packages, install them as well and play around with them to see if all still works properly. There have been cases where that kind of test revealed serious problems (incorrect dependencies, wrong data inside package, ...) that could fortunately be addressed quickly. Do your testing!

# Pull Requests

If you're planning to do a pull request and are not a Duality core developer, please also read [How to Contribute](how-to-contribute.md). In general, **Pull Requests are always done in their own feature branch**. This is to ensure that they remain isolated and can be dealt with individually, rather than polluting a contributor's `master` branch and potentially leaking across different Pull Requests. After a PR has been merged back to `master`, its feature branch can be deleted safely.

# Major Version Steps

Since the `master` branch does not allow breaking changes, all the bigger stuff has to wait until the next major version cycle. As long as development has not begun on the next major version, nothing special needs to be done.

## Beginning a Major Version Cycle

When starting to develop on a new major version number of Duality, the first thing that needs to be done it creating a new major version development branch. Say, the current Duality version is `2.x`, then the next major version step would be `3.0`. 

- Create a distinct `develop-3.0` branch from `master`, including its own continuous integration status badge on the main repository page.
- All breaking or major changes will be developed in `develop-3.0`.
- All non-breaking, non-major changes should continue to be developed in `master`.
- Occasionally merge the most recent `master` version into `develop-3.0`. Note that **this is a unidirectional merge**: It is **not** allowed to ever merge back from `develop-3.0` into `master`.

## Ending a Major Version Cycle

At some point, the next major version will be considered ready for review by the overall crowd, and eventually release. Let's keep the `2.x` / `3.0` sample from above.

- First, create a `2.x` archive branch from `master`.
- Merge `develop-3.0` back into `master` and delete `develop-3.0` afterwards.
- Let the community review the new `master` branch for a while and continue ongoing development in `master` as usual - just now on the new major version.
- When considered ready for release, merge `master` into `release` and perform a full binary package deployment as described above.

The major release cycle is now complete. Back to ongoing development!