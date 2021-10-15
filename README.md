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

To make updates to a Snapshot without changing the base resolver, create a new
snapshot at:

```
lts/X/Y/FR{revision}.yaml
```

_See [this Issue][issue] for a more complete discussion._

[issue]: https://github.com/freckle/stackage-snapshots/issues/4

## Creating a new snapshot

We try to do this every two weeks, but feel free to do it whenever you like.

1. Run `bin/new-snapshot`

Proceed with the following maintenance steps:

### Update any dependencies

**Do any git dependencies have the changes we need in Hackage now?**

If so, make them Hackage dependencies. If not,

**Do any git dependencies have newer commits?**

If so, and they're our own package, update the `commit` to latest. If they are
not our own package, leave them be. Unless there is explicit reason to update,
we'll wait until the next Hackage version for packages we don't own ourselves.

**Do any Hackage dependences have this version (or newer) in the new resolver?**

If so, remove them and rely on the resolver version. If not,

**Do any Hackage dependencies have newer versions on Hackage?**

If so, update to them.

### Updating Hackage package checksums

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

### Testing the snapshot

`bin/check` performs a `--dry-run` build, which is fast but not a real guarantee
that the snapshot will compile. CI builds the representative package fully.

### Migrate each Application to the new snapshots

Remove any app-specific `extra-deps` are no longer required.

---

[LICENSE](./LICENSE)
