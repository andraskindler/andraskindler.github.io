---
layout: post
title:  How to Fix Sass @import Rules Are Deprecated and Will Be Removed in Dart Sass 3.0.0. warning
date:   2025-06-27
categories: 
slug: how-to-fix-sass-import-rules-deprecated-and-will-be-removed
excerpt: Sass @import rules are deprecated and will be removed. Here's how to resolve the warning.
deprecated: false
main_page: false
---

> 💡 TIP / TLDR: instead of `@import`, switch to the newer rules `@use` and `@forward`.

Setting up a Jekyll site for a new project, I bumped into the following warning:

```
Deprecation Warning [import]: Sass @import rules are deprecated and will be removed in Dart Sass 3.0.0.

More info and automated migrator: https://sass-lang.com/d/import
```

Based on the suggested [post on sass-lang.com](https://sass-lang.com/documentation/breaking-changes/import/), Sass @import rules and global built-in functions are now deprecated, and eventually will be removed. There are numerous issues with `@import`:

 * it requires Sass members to be manually namespaced to avoid conflicts;
 * it slows down compilation when the same file is imported more than once;
 * and it and makes it very difficult for both humans and tools to tell where a given variable, mixin, or function comes from.

## How can I fix the `Sass @import Rules Are Deprecated and Will Be Removed in Dart Sass 3.0.0.` warning?
The maintainers of Sass provide a [Sass migrator](https://sass-lang.com/documentation/breaking-changes/import/#automatic-migration) to automatically update your stylesheets to use the module system:

```
$ npm install -g sass-migrator
$ sass-migrator module --migrate-deps your-entrypoint.scss
```

If you want to migrate manually, they also have detailed [recipes for nested imports and configured themes](https://sass-lang.com/documentation/breaking-changes/import/#migration-recipes). In my case, replacing `@import` with `@use` was the solution.

You could also [silence the warnings](https://sass-lang.com/documentation/breaking-changes/import/#can-i-silence-the-warnings), but be mindful that the Dart Sass @import rules or global built-in functions will be removed at a certain point:

*"We don’t expect to remove Sass @import rules or global built-in functions until Dart Sass 3.0.0, which will be released no sooner than two years after Dart Sass 1.80.0."* [^1]

For more details, read the [post](https://sass-lang.com/documentation/breaking-changes/import/) mentioned above, and [this post](https://sass-lang.com/blog/import-is-deprecated/) as well.

----

[^1]: As seen in the [official blog post](https://sass-lang.com/documentation/breaking-changes/import/#transition-period)