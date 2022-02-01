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

## Updating Hackage package checksums

- Update the version, and throw the previous SHA off in some minor way

  ```diff
       # Newer versions that will arrive in next major LTS
     - faktory-1.1.1.0@sha256:e025e23f2d07d21def0d143302051a87f526b70620679036815696676daf9f91,9047
     - aws-xray-client-persistent-0.1.0.2@sha256:f00b3c3da576c488f2605ab5d049437d935eae429e7fe0eb9a6dc53d0367aebc,1682
  -  - freckle-app-1.0.0.3@sha256:1eb1b4cf1fb9313bd42cec1b3400586bd5b1de65f5c692f292ff541e0198f784,6269
  +  - freckle-app-1.0.0.4@sha256:1eb1b4cf1fb9313bd42cec1b3400586bd5b1de65f5c692f292ff541e0198f785,6269
  ```

- Use `bin/check`, which will fail

  ```
  Could not find freckle-app-1.0.0.4@sha256:1eb1b4cf1fb9313bd42cec1b3400586bd5b1de65f5c692f292ff541e0198f785,6269 on Hackage
  The specified revision was not found.
  Possible candidates: freckle-app-1.0.0.4@sha256:8cb624bb3e8805d626b700127690d54d1ddc736073720ac9806a3b04dd3c3216,6244.
  ```

  If you forgot to modify the SHA in step 1, you will get a _Mismatched package
  identifier_ error instead of this one.

- Use the SHA visible in the error message

  ```diff
       # Newer versions that will arrive in next major LTS
     - faktory-1.1.1.0@sha256:e025e23f2d07d21def0d143302051a87f526b70620679036815696676daf9f91,9047
     - aws-xray-client-persistent-0.1.0.2@sha256:f00b3c3da576c488f2605ab5d049437d935eae429e7fe0eb9a6dc53d0367aebc,1682
  -  - freckle-app-1.0.0.4@sha256:1eb1b4cf1fb9313bd42cec1b3400586bd5b1de65f5c692f292ff541e0198f785,6269
  +  - freckle-app-1.0.0.4@sha256:8cb624bb3e8805d626b700127690d54d1ddc736073720ac9806a3b04dd3c3216,6244
  ```

## Testing the snapshot

`bin/check` performs a `--dry-run` build, which is fast but not a real guarantee
that the snapshot will compile. CI builds the representative package fully.

## Migrating Applications

Remove any app-specific `extra-deps` are no longer required.

---

[LICENSE](./LICENSE)
