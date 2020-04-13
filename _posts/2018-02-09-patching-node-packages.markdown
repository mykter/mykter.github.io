---
layout: post
title: "Patching node packages"
date: 2018-02-09
categories: node.js electron
---

An investigation into managing patches for node libraries.

I'm developing an [electron application][tagtime], and whilst using a third party library came across a [missing feature][tab] that upstream don't want to implement. It can be implemented with a trivial one-line change, but I found the real difficulty was working out how to manage this patch. This isn't an uncommon scenario, but I'd found very few people talking about how to manage such situations.

This post goes over the various options I considered and ultimately makes a recommendation on what approach to use when.

## Local copy

You could copy the entire module into your project, and commit it with your fix. Gross.

## Fork

You could fork the module and make the changes in the fork. npm [supports][install] installing a module directly from a git repo, so that's straightforward. This is the usual approach to this problem and in many cases is probably the best solution.

The downside is tools will no longer detect your dependency: say goodbye to npm, [greenkeeper], or [snyk] notifications about updates or vulnerabilities in that module, for example. I'd quite like to have my cake and eat it too -- I want to track upstream without having to remember to do something different for a special subset of dependencies.

## post-install patch

npm will run the `postinstall` script if it's specified in package.json. You could use this to apply a patch to the installed module. At first glance this is neat -- it happens at the right time, and requires minimal maintenance -- just add a patch file to the repo and tweak package.json.

One hiccup is the availability of `patch` -- it's a unix command. Fortunately git exposes patch functionality under `git apply` and any development platform will have git installed if your application is managed in git.

However: whilst this will work for the _first_ `npm install`, subsequent invocations (for example after updating any other module, or in a CI system that caches `node_modules`) will try and apply the patch again to the already-patched module, which will fail because patches aren't idempotent.

You could work around this by making the script leave a marker behind, say adding a `.<myproject>.patched` file in the module's root or similar. This loses some of the cleanliness of the approach, but it could work.

Update: it turns out about a year before I wrote this [patch-package][patch-package] was released which uses this approach in a very nice manner, and I just didn't find it when doing my research. It allows you to make the fix directly in `node_modules`, and will then generate the patch for you and apply it to future installs.

## There's a module for that

[node_module_patches][node_modules_patches] gets around the idempotence issue by overwriting entire files instead of applying a standard patch. You set up a mirror-hierarchy of `node_modules` under `node_modules_patches`, any files in there are copied over the top of the original ones, and it displays a diff for the user. You could (should?) run this from `postinstall` as well.

This might be a reasonable halfway-house, but I don't like the "commit a random file from another project into your repo" and I _really_ don't like that you could totally break your dependency and not notice. Because it's not a proper patch, the module could change in any way and this will happily overwrite the file. It's on the user to decide that the diff doesn't look right any more. Yuk.

## Build system patch

A tool like grunt, gulp, or webpack could implement the same behaviour as the postinstall script described above by having one of the build steps apply a patch. I see two options: directly patch the installed module in `node_modules`, or exploit node's search behaviour to create a (patched) copy of the module. Directly patching the installed module has the same issue as using post-intstall in that patches aren't idempotent.

The latter option is more promising. In my case, I'm already using gulp to build my code; the output goes into an `app` directory. If the patched package is placed under `app/node_modules/` it will get picked up by node in preference to the original in the application root.

The approach I've taken is to create a `patches` directory, which contains a directory for each package that needs patching. These directories contain one or more patch files that will be applied to that package.

The [gulp-apply-patch] plugin fits the bill nicely for this task. Here's the relevant part of my gulpfile, I'm fairly happy with how simple it is:

```javascript
const patch = require("gulp-apply-patch");

const BASE_DIR = path.resolve(".");
const BUILD_DIR = path.join(BASE_DIR, "app");
const PATCHES_DIR = path.join(BASE_DIR, "patches");
const NODE_MODULES = path.join(BASE_DIR, "node_modules");
const PATCHED_MODULES_DIR = path.join(BUILD_DIR, "node_modules");

gulp.task("patch", function () {
  // Get all the directories in the patches folder
  let to_patch = fs
    .readdirSync(PATCHES_DIR)
    .filter((f) => fs.statSync(path.join(PATCHES_DIR, f)).isDirectory());

  // If there's only a single entry in the patches directory, the {set glob}
  // syntax won't match, so add a dummy entry to work around that
  to_patch.push("nonexistent_dummy_package");

  // Convert the patch directories into a glob that matches all the files
  // from the corersponding source packages.
  let globs_to_patch = NODE_MODULES + "/{" + to_patch.join(",") + "}/**/*";

  return gulp
    .src(globs_to_patch, { base: NODE_MODULES }) // copy only the modules that have patches
    .pipe(patch(PATCHES_DIR + "/**/*.patch"))
    .pipe(gulp.dest(PATCHED_MODULES_DIR));
});
```

This has some nice properties:

- The change being made is explicit, as there's a patch file in the repo.
- When upstream changes, either the patch will continue to apply and all is probably well (diff line numbers and context lines haven't changed), or it will fail to apply and you know to check and update.
- From the perspective of tools looking at `package.json`, you're using the standard upstream library. Everything works normally.

The main downside compared to the fork approach is maintaining a patch file is more work compared to maintaining a forked repo. You can't just `git pull`, unless you combine both approaches and generate the patch from a fork. It is also slightly higher risk, as after an upstream change the patch might still successfully apply, but it now breaks the package in a way that would have been picked up had you run the package's tests after pulling the changes into a forked repo.

## Monkey patch

Thanks to the wonder that is javascript we could use the original module unmodified, but at run-time try and fix it up to have the behaviour we want. Depending on the patch this _could_ be trivial and relatively clean -- perhaps an 'internal' integer should have a different value. It could be really ugly if, say, a large function needs a slight logic change -- the whole function will need to be replaced, implying your patching code has to have a complete copy of said function.

{% highlight javascript %}
/_ eslint-disable _/
// monkey patch to make it do the right thing
originalModule.func = function() {
...
{% endhighlight %}

Good luck getting that to type-check in TypeScript!

This feels fragile, kind of like `node_module_patches` -- you might well only know when it breaks when you (or your users) hit the bugs at run time.

## Other languages

How is this solved in other languages?

[Composer] is a package manager for php, partly inspired by npm. There is a plugin for composer called [composer-patches] which does exactly what I'm looking for: it applies a patch to dependencies as they are installed by composer. Because it's integrated with the package manager, it avoids the issue with using npm's `postinstall` script described previously -- the patches are only applied when the module is installed. It goes further than this and supports dependencies applying patches to their own dependenciesto their own dependencies -- this seems a little problematic, as it encourages creating libraries that need patches to be applied, which in turn forces any users of your library to use composer-patches as well. Neither npm nor yarn support plugins.

Go has some particular problems with the fork approach, because dependencies include their full path: `import "github.com/a/b"`. These can be [worked around][dep], but that's all I've found on this topic.

The tradition in ruby seems to be monkey patching.

I couldn't see anything beyond the fork approach mentioned for python or rust.

## Conclusion

Forking is the most robust approach, and you should probably do this. However if you'd like your existing dependency management tools to continue working with this patched dependency, there a few options.

Update: you should probably just use [patch-package][patch-package], a robust install-time patching tool.

It may be worth considering monkey patching - is your change small, easy to implement at runtime, and when upstream changes likely to either be robust or fail noisily? This could be the simplest approach.

In preference to monkey patching I would probably recommend applying a source code patch as part of a build step. It's relatively robust, easy to manage, and less fiddly than trying to monkey patch.

Hopefully this will save someone the time I spent searching for something like this post! If you have any thoughts on the subject, or decide to re-implement my solution as a gulp plugin, please let me know!

[tagtime]: https://www.github.com/mykter/TagTime-desktop
[tab]: https://github.com/moroshko/react-autosuggest/pull/164
[install]: https://docs.npmjs.com/cli/install
[greenkeeper]: http://greenkeeper.io/
[snyk]: https://snyk.io
[node_modules_patches]: https://www.npmjs.com/package/node_modules_patches
[composer]: https://getcomposer.org/
[composer-patches]: https://github.com/cweagans/composer-patches
[dep]: https://blog.gopheracademy.com/advent-2017/managing-dependencies-forks-and-code-patches-with-dep/
[gulp-apply-patch]: https://github.com/kaa/gulp-apply-patch
[patch-package]: https://github.com/ds300/patch-package
