# gitlify

Tool to convert FCM repositories to git, and keep them in sync.

## Usage

### Simple migration

- `gitlify migrate URL` - main migration step. `URL` may be any URL to a
  subversion repository, as well as an FCM alias such as `fcm:um.x`.

  Migration can take a long time, depending entirely on the size of the
  repository.  A few hours should be plenty for a reasonably sized repository
  (~several thousand commits), but the UM example would probably take weeks!

  It is recommended to redirect the output to a file.

- `gitlify prune` - remove merged branches and empty commits.

  SVN introduces many commits that do not modify any files, such as creating a
  branch, which in git are meaningless and should be removed.

  It is also a surprisingly common workflow for FCM-based projects to not
  delete branches after merging them.

- `gitlify resync` - fetch new commits from the FCM repository.

  This will additionally prune as described above, but limited to newly-merged
  branches only.

### Committing to FCM

No additional wrappers are provided for commiting; use
[`git svn dcommit`](https://git-scm.com/docs/git-svn#Documentation/git-svn.txt-emdcommitem)
if required. Remember to **always** use the `--dry-run` option first, to ensure
you are committing what you expect where you expect: subversion commits are
much more permanent than git commits.

Alternatively, patches can always be applied manually:
```
cd some_project/git
git diff --no-prefix @{upstream} > ../branch.patch
cd ../some_fcm_branch
fcm patch ../branch.patch
fcm commit
```

It is probably not worth attempting to create branches via `git svn`. Just do
it the FCM way.

### Mirroring

It can be useful to maintain a single git mirror of an FCM repository. There
are some additional commands to help with this:

- `gitlify track` - create git branches for each FCM branch. Such branches will
  then show up (by default) when this mirror is cloned.

- `gitlify resync -t` - shorthand to run `gitlify track` after a `resync`.

Two-way synchronisation is **not** currently recommended. Enforce this by
creating a `pre-receive` hook in the `.git` folder of any mirror repo:
```sh
#!/bin/sh
echo "This repository is a mirror"
exit 1
```

### Advanced

`gitlify migrate` is simply a shorthand for `gitlify init` followed by
`git svn fetch`, where `gitlify init` is itself a wrapper around `git svn init`
with additional defaults and conveniences. There is therefore an opportunity to
review the result of `gitlify init` before the lengthy `fetch`. In particular:

- The branch layout is set up as expected for Met Office hosted repositories,
  with subfolders `branches/dev/$username` and `branches/pkg/$username`. This
  can be modified in the `[svn-remote "svn"]` section of `gitconfig`.

- A list of all contributing authors is automatically fetched and saved (in
  the current working directory) as `users-all.txt`. Furthermore, the file
  `/etc/aliases` is queried for corresponding email addresses and saved (in
  the `.git` folder) as `users-metadata.txt`. Check that the metadata file
  includes an appropriate email address for all authors.

- An executable file `missing-name` can be added to the `.git` folder. This
  should accept one argument - a username not found in `users-metadata.txt` -
  and print a name and email to stdout, in the form `Name <name@example.com>`.
  See [git-svn documentation](https://git-scm.com/docs/git-svn#Documentation/git-svn.txt---authors-progltfilenamegt)
  for details.
