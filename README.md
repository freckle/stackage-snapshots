# Stackage Snapshots

https://docs.haskellstack.org/en/stable/pantry/#snapshot-location

Centralizing the `extra-deps` we require throughout our Haskell applications.
Freckle team members can see more details in the [RFC][rfc].

[rfc]: https://renaissancelearning.atlassian.net/wiki/spaces/EN/pages/41987178508/Shared+Backend+Stackage+Snapshot

## Usage

```yaml
# stack.yaml
resolver: https://raw.githubusercontent.com/freckle/stackage-snapshots/main/lts/17/15.yaml
```

## Snapshots

We define Snapshots built on a given LTS and match the same file structure:

```
lts-X.Y -> lts/X/Y.yaml
```

Once committed to `main`, **Snaphots are immutable**. These files are cached in
various ways as they're used and modifying one in place could result in a very
confusing and frustrating experience for developers.

To make updates to a Snapshot without changing the base resolver, create a new
snapshot at:

```
lts/X/Y/FR{revision}.yaml
```

_See [this Issue][issue] for a more complete discussion._

[issue]: https://github.com/freckle/stackage-snapshots/issues/4

## Creating a new snapshot

Every two weeks, the `bump.yml` workflow will run and, if a new resolver is
available, create a PR that adds it. You can also do this manually at any time
by running `bin/new-snapshot`.

This PR should be reviewed to understand the changes in the new resolver, if it
contains important packages or a GHC upgrade, etc. It may require edits to
compile. **In general**, if our tests and linting pass, it is acceptable to
merge.

If you would like to more thorough testing, you can update any app to use the
resolver from the branch by changing its `stack.yaml`:

```diff
- resolver: https://raw.githubusercontent.com/freckle/stackage-snapshots/main/lts/18/21.yaml
+ resolver: https://raw.githubusercontent.com/freckle/stackage-snapshots/create-pull-request/patch/lts/18/23.yaml
```

## Automated linting

CI will run [SLED][], which checks for:

[sled]: https://github.com/freckle/stack-lint-extra-deps

- Any Hackage dependencies with same-or-newer version in Stackage

  We should remove the `extra-dep`, unless there's a reason we're staying behind.

- Any Hackage dependencies with a newer released version

  We should update, unless there's a reason we're staying behind.

- Any Git dependencies with a version-like tag newer than the commit

  This likely means a released version on Hackage we can move to.

- Any Git dependencies with newer commits on the default branch

  We may want to update.

If the `lint` step generates a false-positive, you can silence it by adding an
`--exclude` option.

## Hackage checksums

We used to add extra-deps with their sha256 checksums. These were difficult to
maintain and Stackage authors have suggested not using them. Provided you are
committing `.lock` files in your projects (we are), then those checksums will be
maintained for you there and any differences visible in diffs already.

## Testing the snapshot

The CI suite checks out our most complicated Haskell project (megarepo) and
builds it using the most recent snapshot in this repository. This example
project is private, so CI won't work on PRs from forks.

### Handling Red CI Caused by Example Project

Sometimes, a new snapshot may make megarepo fail to compile without changes (new
exports conflict, imports made redundant). This is a chicken-and-the-egg
scenario since you can't update megarepo until you land the snapshot, and red CI
is blocking it.

For this case,

- Locally use the new snapshot in megarepo (see above) and resolve the errors
- Once good there, merge the new snapshot even though CI is red
- Update megarepo to use the new snapshot immediately

---

[LICENSE](./LICENSE)
