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

If we make multiple Snapshots on the same base resolver, we may use branches,
tags, or distinct files with descriptive suffixes.

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

### Test the snapshot

Test changes by updating an application to use a local copy in an adjacent
clone:

```yaml
# stack.yaml
resolver: ../../stackage-snapshots/lts/18/1.yaml
```

*Testing with `megarepo` is usually enough.*

### Migrate each Application to the new snapshots

Remove any app-specific `extra-deps` are no longer required.

---

[LICENSE](./LICENSE)
