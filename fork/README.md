# Personal T3 Code fork

`custom` = **latest upstream stable release tag** + **my commits on top**.

```
pingdotgg/t3code  ──v0.1.8──v0.1.9──────v0.1.10──▶ upstream/main
                              \               \
                               A─B─C  ──rebase──▶ A'─B'─C'   = custom (my features)
```

- `main` on the fork is a plain mirror of upstream `main` (never commit to it).
- `custom` is the only branch with my changes. Each commit is one feature/fix.
- Nothing is sent upstream (CONTRIBUTING.md rejects feature PRs). Bug fixes can still be
  PR'd from a separate branch off `upstream/main`; once merged, the rebase drops my copy.

## Automation (`fork/t3fork`)

An hourly launchd agent (`com.bobby.t3fork`) runs `t3fork update`:

1. Fetches upstream, mirrors `main` to the fork.
2. If a new stable release is out, rebases `custom` from the old tag onto the new one and
   force-pushes it. Conflicts → the rebase is aborted, nothing changes, and a notification says
   to run `t3fork rebase` (git `rerere` remembers resolutions for next time).
3. If `custom` changed since the last build (new release *or* new commits of mine), builds a
   macOS app from it in a separate worktree (`~/.t3fork/build`), signs it with my Apple
   Development certificate so Keychain access persists across builds, and stages it.
4. Installs it to `/Applications/T3 Code (Beta).app` once T3 Code is not running (the previous
   app is kept in `~/.t3fork/previous` for `t3fork rollback`).

The build has no update feed, so the built-in updater is off and never replaces it with the
stock build. Same bundle ID → same settings, threads and data as the official app.

## Adding a feature

```sh
cd ~/Developer/t3code
git switch custom
# ...make the change (pnpm install && pnpm dev:desktop for live dev)...
git commit -am "feat: my thing"
t3fork build --restart    # or wait for the next hourly run
```

## Commands

| command | what |
| --- | --- |
| `t3fork status` | upstream release, base tag, my commits, built/installed state |
| `t3fork update [--restart]` | what the agent runs |
| `t3fork build [--restart]` | rebuild + install `custom` now |
| `t3fork install [--restart]` | install a staged build (`--restart` quits/reopens T3 Code) |
| `t3fork rebase` | interactive rebase onto the latest release when there are conflicts |
| `t3fork rollback` | restore the previously installed app |
| `t3fork setup` / `uninstall-agent` | install/remove the launchd agent |

Logs: `~/.t3fork/logs/latest.log`.
