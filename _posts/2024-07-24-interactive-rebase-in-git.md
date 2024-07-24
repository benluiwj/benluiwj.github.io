# Interactive Rebase in Git
Recently, I was contributing to [rustfmt](https://github.com/rust-lang/rustfmt). when I stumbled across this problem. I was tasked with performing 2 squashes - 1 squash on the feature implementation and 1 squash on CI update. Then, update the commit message of the squashes.

Now this was something I had not done before and thus, I tried to perform the task. I guess I partially did it correctly since the end result was 2 commits, though 1 was squashed.

## What I did

The commits of the implementation was the first set of commits and the CI commits were in the second part.

I squashed the CI commits by only looking at `HEAD~n`, where `n` is the number of CI commits.

The tricky part was squashing the remaining commits while maintaining the squashed commit. This was where I was unsuccessful.

```
pick <commit_hash_1> <1st line from commit message> # this is the squashed CI commit
pick <commit_hash_2> <2nd line from commit message>
pick <commit_hash_3> <3rd line from commit message>
pick <commit_hash_4> <4th line from commit message>
pick <commit_hash_5> <5th line from commit message>
pick <commit_hash_6> <6th line from commit message>
pick <commit_hash_7> <7th line from commit message> # first impl commit

# Rebase <base_commit>..<commit_hash_7> onto <base_commit> (7 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified); use -c <commit> to reword the commit message
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#

```

What I did was I changed all the remaining commits from `pick` to `squash` except the first and last. This caused all the commits to from the implementation to be squashed **BUT** the CI commits were lost in the process.

Lucky for me, the CI commits was effectively a simple change so I just made a single commit. 

## What I should have done

Instead of performing interactive rebase twice, I could have done it once. I could have done something like this:

```
pick <commit_hash_1> <1st line from commit message> # squashes 1 to 5
squash <commit_hash_2> <2nd line from commit message>
squash <commit_hash_3> <3rd line from commit message>
squash <commit_hash_4> <4th line from commit message>
squash <commit_hash_5> <5th line from commit message>
pick <commit_hash_6> <6th line from commit message> # squashes 6 and 7
squash <commit_hash_7> <7th line from commit message>

```

Since Git reads the file topdown, what happens is the following:

- Git sees `pick` and selects the commit
- Git sees `squash` and squashes the commit into the currently picked one
- Git sees `pick` and updates the currently picked one

This way, the commits will be nicely squashed. 

## Editing the messages on squash commits

Once the squashes are done, another interactive rebase has to be done. This time, change `pick` to `edit`. Upon exiting the console, this message pops up in the terminal:

```
Stopped at <commit hash> ...  <commit message>
You can amend the commit now, with

  git commit --amend 

Once you are satisfied with your changes, run

  git rebase --continue
```

This allows editing of the commit with `<commit hash>`. Then, running `git commit --amend`, another window will be open to let us edit the commit message. Once done, we can just continue the rebase. 

## Conclusion

This was quite an interesting experience amending commits and squashing them. Even though I have used Git before but interactive rebase was something I rarely touch. I guess it is really true that contributing to open-source lets one pick up a thing or two.
