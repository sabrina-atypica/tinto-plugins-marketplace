# Working on this repo across parallel sessions

This repo replaced a setup where every Claude Cowork session treated one shared
zip file (`Claude outputs/tinto-travels-plugin-marketplace.zip`) as the
source of truth, and repackaging it from a local checkout meant silently
overwriting whatever anyone else had changed. That caused real, repeated data
loss (twice in one day, across three different sessions) before this repo
existed. Git replaces "last write wins" with an actual merge, but only if
every session follows the same few steps below.

## Where things live

- Remote: `git@github.com:sabrina-atypica/tinto-plugins-marketplace.git` (private).
- Working copy: `TintoTravels\tinto-plugins-marketplace` on Sabrina's computer.
- Deploy key: `TintoTravels\tinto-plugins-marketplace\.deploy-key\id_ed25519` (+ `.pub`),
  gitignored, never commit it. Every session uses this same key; there is
  nothing to hand out or regenerate per session.
- **Run git from Sabrina's computer, not the cloud container.** GitHub's SSH
  port isn't reachable from the cloud sandbox's network (only proxied HTTP/S
  gets out), so all clone/pull/push happens via a shell on her computer.
  A cloud session that needs to read or edit files still does that in the
  cloud workspace as usual; only the actual git operations need to run on her
  computer, e.g.:

  ```
  export GIT_SSH_COMMAND="ssh -i $(pwd)/.deploy-key/id_ed25519 -o IdentitiesOnly=yes -F /dev/null -o UserKnownHostsFile=$HOME/.ssh/known_hosts"
  git pull origin master
  git push origin <branch>
  ```

## The workflow

1. **Say which plugin you're working on before you start**, and check with
   Sabrina (or just ask her) whether another session already claimed it.
   Two sessions can safely work in this repo at once as long as they're not
   both editing the same plugin's files or `.claude-plugin/marketplace.json`
   in the same window.

2. **Pull before you touch anything.** `git pull origin master` on Sabrina's
   computer, every time, before editing or repackaging. This is what the old
   zip-diffing dance was standing in for; git does it properly now.

3. **Branch per plugin (or per session).** `git checkout -b
   <plugin-name>-<short-description>` rather than committing straight to
   `master`. Keeps parallel work isolated and makes it obvious in the repo's
   branch list what's in flight.

4. **Validate before pushing.** `claude plugin validate plugins/<name>` for
   any plugin you touched, and `claude plugin validate .` from the repo root
   if you touched `marketplace.json`. Don't push a branch that fails either.

5. **Push your branch, not `master` directly**, unless Sabrina says otherwise:
   `git push -u origin <branch-name>`. Tell her the branch exists and what it
   changes so she can decide when it merges into `master`.

6. **Merging to `master` and repackaging the zip is a "master" operation**,
   meaning only do it once a branch is actually meant to ship, and pull
   immediately before merging in case another branch merged first. After
   merging, that's the point to rebuild `tinto-travels-plugin-marketplace.zip`
   for `Claude outputs` (see below), not before.

7. **If a real merge conflict comes up**, that's git doing its job, not a
   failure. Resolve it normally, or flag it to Sabrina if it's a substantive
   design conflict (e.g. two sessions changed the same skill's logic
   differently), rather than picking one side and discarding the other.

## Repackaging the marketplace zip

The zip in `Claude outputs` is an export, not the source of truth anymore.
Rebuild it from a clean, up-to-date `master` (`git pull` first) whenever
Sabrina wants a fresh copy there, the same way as before:

```
cd TintoTravels/tinto-plugins-marketplace
zip -X -r tinto-travels-plugin-marketplace.zip . -x '.git/*' -x '.deploy-key/*' -x '.gitignore'
```

Deliver it the same way as always (`SendUserFile` + committed into
`Claude outputs`), keeping only the zip at that folder level per Sabrina's
existing preference (no loose `.plugin` files there).
