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

## Updates

Every 2 weeks, we move our applications to the newest LTS:

1. Copy the latest Snapshot to a new, correctly-named file
1. Update its `resolver`
1. See if any Git `extra-deps` have been released to Hackage
1. See if any Hackage `extra-deps` have been added to this Resolver

## Testing

Test changes by updating an application to use a local copy in an adjacent
clone:

```yaml
# stack.yaml
resolver: ../../stackage-snapshots/lts/18/1.yaml
```

---

[LICENSE](./LICENSE)
