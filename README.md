AutoTag
=======
Automatically increment version tags to a git repo based on commit messages.

Installing
----------
[releases]: https://github.com/acnosov/autotag/releases/latest

Usage
-----

The `autotag` utility will use the current state of the git repository to determine what the next
tag should be and then creates the tag by executing `git tag`. The `-n` flag will print the next tag but not apply it.

`autotag` scans the `main` branch for commits by default. If no `main` branch is found, it will
fall back to the `master` branch.  Use `-b/--branch` to scan a different branch. The utility first
looks to find the most-recent reachable tag that matches a supported versioning scheme. If no tags
can be found the utility bails-out, so you do need to create a `v0.0.0` tag before using `autotag`.

Once the last reachable tag has been found, the `autotag` utility inspects each commit between the
tag and `HEAD` of the branch to determine how to increment the version.

Commit messages are parsed for keywords via schemes. Schemes influence the tag selection according
to a set of rules.

Schemes are specified using the `-s/--scheme` flag:

### Scheme: Autotag (default)

The autotag scheme implements SemVer style versioning `vMajor.Minor.Patch` (e.g., `v1.2.3`).

Before using autotag for the first time create an initial SemVer tag,
eg: `git tag v0.0.0 -m'initial tag'`

The next version tag is calculated based on the contents of commit message according to these
rules:

- Bump the **major** version by including `[major]` or `#major` in a commit message, eg:

```
[major] breaking change
```

- Bump the **minor** version by including `[minor]` or `#minor` in a commit message, eg:

```
[minor] new feature added
```

- Bump the **patch** version by including `[patch]` or `#patch` in a commit message, eg:

```
[patch] bug fixed
```

If no keywords are specified a **Patch** bump is applied.

### Scheme: Conventional Commits

Specify the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#examples) v1.0.0
scheme by passing `--scheme=conventional` to `autotag`.

Conventional Commits implements SemVer style versioning `vMajor.Minor.Patch` similar to the
autotag scheme, but with a different commit message format.

Examples of Conventional Commits:

- A commit message footer containing `BREAKING CHANGE:` will bump the **major** version:

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

- A commit message header containing a _type_ of `feat` will bump the **minor** version:

```
feat(lang): add polish language
```

- A commit message header containg a `!` after the _type_ is considered a breaking change and will
  bump the **major** version:

```
refactor!: drop support for Node 6
```

If no keywords are specified a **Patch** bump is applied.

### Pre-Release Tags

`autotag` supports appending additional test to the calculated next version string:

- Use `-p/--pre-release-name=` to append a pre-release **name** to the version. Pre-release names are subject to the rules outlined in the [SemVer](https://semver.org/#spec-item-9)
  spec.

- Use `-T/--pre-release-timestmap=` to append **timestamp** to the version. Allowed timetstamp
  formats are `datetime` (YYYYMMDDHHMMSS) or `epoch` (UNIX epoch timestamp in seconds).

### Build metadata

Optional SemVer build metadata can be appended to the version string after a `+` character using the `-m/--build-metadata` flag. eg: `v1.2.3+foo`

Build metadata is subject to the rules outlined in the [SemVer](https://semver.org/#spec-item-10)
spec.

A common uses might be the current git reference: `git rev-parse --short HEAD`.

Multiple metadata items should be seperated by a `.`, eg: `foo.bar`

Examples
--------

```console
$ autotag
3.2.1
```

```console
$ autotag -p pre
3.2.1-pre

$ autotag -T epoch
3.2.1-1499320004

$ autotag -T datetime
3.2.1-20170706054703

$ autotag -p pre -T epoch
3.2.1-pre.1499319951

$ autotag -p rc -T datetime
3.2.1-rc.20170706054528

$ autotag -m g$(git rev-parse --short HEAD)
3.2.1+ge92b825

$ autotag -p dev -m g$(git rev-parse --short HEAD)
3.2.1-dev+ge92b825

$ autotag -m $(date +%Y%M%d)
3.2.1-dev+20200518

$ autotag  -m g$(git rev-parse --short HEAD).$(date +%s)
3.2.1+g11492a8.1589860151
```

For additional help information use the `-h/--help` flag:

```console
autotag -h
```

Troubleshooting
---------------

### error getting head commit: object does not exist [id: refs/heads/master, rel_path: ]

```
error getting head commit: object does not exist [id: refs/heads/master, rel_path: ]
```

You may run into this error on certain CI platforms such as Github Actions or Azure DevOps
Pipelines. These platforms tend to make shallow clones of the git repo leaving out important data
that `autotag` expects to find. This can be solved by adding the following commands prior to
running `autotag`:

```sh
# fetch all tags and history:
git fetch --tags --unshallow --prune

if [ $(git rev-parse --abbrev-ref HEAD) != "master" ]; then
  # ensure a local 'master' branch exists at 'refs/heads/master'
  git branch --track master origin/master
fi
```

Release information
-------------------

Autotag itself uses `autotag` to increment releases. The default [autotag](#scheme-autotag-default)
scheme is used for version selection.
