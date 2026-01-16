# Merge Conflicts & Conflict Resolution

When a file is modified and committed on one branch, and the same file is modified and committed at the same location on another branch, merging the latter branch into the former branch will result in conflicts.

To resolve conflicts, merging can be performed locally using a side-to-side diff tool, such as Beyond Compare. One should select the files that contain conflicts and choose to apply either the remote text or the local text at each conflict location.

Relevant commands:
To retain local changes when conflicts occur, use `git merge -Xours other_branch`.
To retain remote changes when conflicts occur, use `git merge -Xtheirs other_branch`.
To retain only local changes even when no conflicts exist (that is, to completely roll back remote changes), use `git merge -s ours other_branch`.
