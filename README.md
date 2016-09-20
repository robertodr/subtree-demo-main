# subtree-demo-main

Main repository for a demo of Git subtrees based on [this
article](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec#.mxwgscjs5)

# Why and How

This repository is meant as a barebones demo of using Git subtrees to manage
external dependencies.
The demo is copied from the tutorial published
[here](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec#.mxwgscjs5)
We will use the same nomenclature: _container_ is the main project, while
_plugin_ is the project that will be added as a subtree.
In principle, both the manual and `git-subtree` strategies described in the
article can be used with these demo repositories.
Only the manual strategy is documented here, since I think it's the best
solution.

We can single out three different workflows that will apply to different
classes of developers in the container/plugin developer teams:

0. Container developers. These generally do not care about the evolution of the
   plugin codebase and/or the additional Git infrastructure as long as
   everything works.
1. Interface developers. These are the ones working on both the container _and_
   the plugin codebase. Most likely these are also the ones in charge of
   updating the plugin code in the subtree and possibly backport modifications
   from the subtree to the main plugin development line.
2. Plugin developers. Mirror situation to the container developers. They are
   most likely only interested in backports from subtrees.

This demo aims at the interface developers.

# Try It Out!

0. Fork this repository and its [companion plugin](https://github.com/robertodr/subtree-demo-plugin)
1. Create a subtree inside the container repository:

   ```
   git remote add <plugin-alias> <plugin-remote>
   git fetch <plugin-alias>
   git read-tree --prefix=<plugin-local> -u <plugin-alias>/<plugin-branch>
   git push
   ```

   The placeholders in angular brackets can be decided by the user:
   * `<plugin-alias>` is a convenient alias for the remote of the plugin
   * `<plugin-remote>` is the full address of the plugin remote repository
   * `<plugin-local>` is the local directory for the plugin source code
   * `<plugin-branch>` is the branch (or tag) of the plugin source code
2. You can now try to clone the container source code in a different directory
   and check that the subtree was actually added.
3. Make an arbitrary modification to the plugin source code in its own
   repository and push it. In this way you'll be able to try out an update of
   the plugin source code from remote:

   ```
   git fetch
   git merge -s subtree --squash <plugin-alias>/<plugin-branch>
   git commit -m "Updated plugin"
   git push
   ```

   In case the merge fails, reset it first and then try:

   ```
   git merge -X subtree=<plugin-local> --squash <plugin-alias>/<plugin-branch>
   ```

Modifications to the source code within the main repository can fall into one
of the following four scenarios:
0. Commits that only touch the subtree. These are the ones we would like to
   backport to the plugin main development line. I think it is good practice to
   mark these commits with `[Backport from <container>]` in the commit message.
1. Commits that only touch the container. These are managed _exactly in the
   same way_ as one would if no subtrees were present.
2. Commits that touch both container and subtree. The latter are meant for
   backporting. This is a quite usual situation when prototyping an interface
   between container and plugin.
3. Commits that only touch the subtree, but are not meant for backporting. This
   might be the case for commits that modify the plugin in a container-specific
   way. IMO, these should be avoided.

In all cases, the strategy is to commit to the container repository and then
_cherry pick_ the commits to be backported to the plugin repository.
We create a branch for this series of operations in the container repository to
get a meaningful history.
In the first scenario one would issue:
```
git checkout -b backport-<plugin> <plugin-alias>/<plugin-branch>
git cherry-pick -x <commit-sha>
git push
```
while in the third scenario:
```
git checkout -b backport-<plugin> <plugin-alias>/<plugin-branch>
git cherry-pick -x --strategy=subtree <commit-sha>
git push
```
While `--strategy=subtree` is not mandatory in the first scenario, it will
ensure that files outside the subtree are correctly ignored during cherry
picking.

