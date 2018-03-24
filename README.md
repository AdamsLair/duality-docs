# Duality Docs

This repository hosts the online documentation of the [Duality game engine](https://duality.adamslair.net/) using GitHub pages. Contributions are always welcome! If you have questions or just want to say Hi, feel free to join us in the [forum](http://forum.adamslair.net/) or our [chat](http://chat.adamslair.net/).

## What to Contribute

Both new manual pages and improvements to existing ones are greatly appreciated. If you want to contribute, but are not yet sure what to do, check out our [issue tracker](https://github.com/AdamsLair/duality-docs/issues) where we organize page requests, errors, improvements and tasks. Look out especially for issues tagged with `Help Wanted` and `Good First Issue` labels, but do not shy away from others if they look like something you'd like to do - and when in doubt, just ask.

## How to Contribute

The project is built upon the technology provided by [Jekyll](https://jekyllrb.com) and [GitHub Pages](https://pages.github.com). In almost all cases though, it's sufficient for you to know your way around [Markdown](https://www.google.de/search?q=markdown), the markup language in which all documentation pages are written.

### Two Ways to Edit Files

The easiest way to modify or add pages is by doing so online using the GitHub web interface: Just browse to the file you want to edit, click `Edit` and GitHub will automatically fork the repository and help you create a new Pull Request once you're done editing. Using `Create new file`, you can also add new pages the same way. 

While this approach is enough for most small to medium sized contributions, bigger and more complex ones involving multiple files, images and other content usually require a bit more control. In those cases, prefer working offline and committing all your changes into a new `feature/XY` branch on your own fork of the repository, for which you can submit a new Pull Request manually as soon as you're ready for review.

### Improving Existing Pages

Improvements to existing pages are often the more lightweight kind that work well by editing files directly on GitHub - for those cases, every page on the published website has an `Edit Page` button, which is essentially a shortcut to browsing to the file on GitHub and clicking `Edit` there. Feel free to use this shortcut, or go the offline route with a manually created feature branch.

### Adding New Pages

The only difference between adding new pages and improving existing ones is that you'll need to figure out where to put the new files and how to initialize them with the right contents. 

Generally, you'll find all pages in the [`pages`](https://github.com/AdamsLair/duality-docs/tree/master/pages) subfolder. Note that all pages need to be sorted into a subfolder that corresponds to their version code, such as `v2` or `v3`. Which name you choose for the file itself doesn't matter, but make sure to choose something that properly describes its content and ends with `.md`.

All pages need to specify a Jekyll Front Matter and define some style specific variables that our Jekyll templates use to structure the docs website and provide the right links to navigate it properly. A typical front matter looks like this:

```
---
title: "Name of the article"
category: "top-level-category-id"
displayOrder: 0
version: "vX"
---
```

Or this, if it's not a top-level, but a child page:

```
---
title: "Name of the article"
parent: "/pages/vX/FileNameWithoutExtension"
displayOrder: 0
version: "vX"
---
```

Before submitting your new page for review, make sure to check all the points listed in the [Pull Request checklist](https://github.com/AdamsLair/duality-docs/blob/master/.github/PULL_REQUEST_TEMPLATE.md) that applies in your case.

### Creating a Local Test Setup

For very large scale changes that require previewing content on the actual generated website, rather than just any markdown capable editor of your choice, you can set up a local Jekyll installation to get a localhost preview in your browser. This is generally not required unless you're making style or generator template changes, but if you do, take a look at [this GitHub docs page](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/).