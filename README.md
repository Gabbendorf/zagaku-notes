# Git rebase/Git rebase --onto

I did a Zagaku about the basics of `git rebase` command and the option `git rebase --onto` on Monday 11th December 2017.
These are my notes which I used for the Zagaku.

## git rebase

`git rebase` takes the commits of a branch and appends them to the commits of a different branch.
The commits to rebase are saved into a temporary area and then reapplied to the new branch, one by one, in order.

Assuming we have 2 different branches, a `master` branch and a `topic` branch based on `master`.
The current branch is `topic`:

```
   A--B--C--D--E                master
          \
           F--G--H              topic
```

We want `topic` branch to be rebased on `master` so that it includes the commits of `master` after the fork was made:

```
   A--B--C--D--E                master
                \
                 F'--G'--H'     topic
```

The commits of `topic` are re-created and applied after the last commit of `master`.
`topic` branch now includes the commits of `master` after it was branched from it.

In order to achieve this, we can use the following Git command:

```
git rebase master
```

, if you're in current branch `topic`, or

```
git rebase master topic
```

, if you're not inside branch `topic` already. In this case, Git first performs an automatic `git checkout topic` before rebasing it to `master`.
When rebase exits, `topic` will remain the checked-out branch.

The technical syntax for the `git rebase` command is:

```
git rebase [<upstream>] [<branch>]
```

`<upstream>` is the starting point at which to create the new commits for `<branch>`.

In case of conflict, `git rebase` will stop at the first problematic commit and leave conflict markers in the tree.
You can use `git diff` to locate the markers (<<<<<<) and make edits to resolve the conflict.
For each file you edit, you need to tell Git that the conflict has been resolved with the following commands:

```
git add <file-name>
git rebase --continue
```

Alternatively, you can undo the git rebase with

```
git rebase --abort
```

## git rebase --onto

This command is used to transplant a branch based on one branch to another, to pretend that it was forked from the latter.
The syntax for `git rebase --onto` is:

```
git rebase [--onto <new-base>] [<upstream> [<branch>]]
```

`<new-base>` becomes the starting point at which to create the new commits of `<branch>`.
As for `git rebase` without the `--onto` option, if you're already inside `<branch>` then you don't need to specify it in the command.
If you specify it, then Git first checks-out `<branch>`, which defaults to `HEAD`, and then operates the rebase.
`<upstream>` is the branch to compare against.

### 1st case
Let's start from the following case to understand how `git rebase --onto` works:

```
   A--B--C--D--E                master
       \
        F--G--H--I              topic1
                  \
                   J--K--L      topic2
```

`topic2` is based on `topic1`. For example, a feature developed in `topic2` depends on some functionality which is found in `topic1`.
We want to fork `topic2` from `master` now. We might want this because a functionality on which `topic2` depends was merged into `master`.
We want to achieve something like this:

```
                 J'--K'--L'     topic2
                /
   A--B--C--D--E                master
       \
        F--G--H--I              topic1
```

We can get this by using the command:

```
git rebase --onto master topic1 topic2
```

`<branch>` `topic2` (specified in the command), which was forked from `topic1`, is now rebased on a `<new-base>` `master`.
The commits of `topic2` are appended to `master` whereas `topic1` remains as it was.

### 2nd case
Imagine we have this case where `topic2` does not depend on `topic1` but was forked from it (maybe mistakenly):

```
   A--B--C--D--E                        master
                \
                 F--G--H                topic1
                        \
                         I--J--K        topic2
```

We want `topic2` to be rebased on `master` and not to contain the commits of `topic1`:

```
   A--B--C--D--E                        master
               |\
               | F--G--H                topic1
                \
                 I'--J'--K'             topic2
```

Again, to achieve this we use the following command:

```
git rebase --onto master topic1 topic2
```

`topic2`, previously forked from `topic1`, is now rebased on `master`.

### `git rebase --onto` to remove commits

`git rebase` can also be used to remove a range of commits in the same branch.
Let's say we have a case like this:

```
   A--B--C--D--E--F     master
```

We want to delete the commits `B` and `C` from branch `master` and ensure that the commits from `D` on are rebased on `A`.
This is useful if `B` and `C` had some defects or they shouldn't be in `master`.
To obtain this:

```
   A--D'--E'--F'        master
```

, we use the following command:

```
git rebase --onto master~5 master~3 master
```

The argument to `--onto` and the `<upstream>` parameter can be any valid commit.
In the case above, the `<new-base>` (the starting point) for the commits of `master` is the fifth commit (`~5`) from the last one (`A`);
the previous base to compare against was the third commit (`~3`) from the last one in `master` (`C`).





