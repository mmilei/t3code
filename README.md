# T3 Code — no-telemetry local build

A personal fork of [pingdotgg/t3code](https://github.com/pingdotgg/t3code) with two small changes on top of upstream `f99808e7` (2026-11-23). Everything else is upstream as-is.

The original README is kept in [README_old.md](./README_old.md).

## Why

I run T3 Code as a desktop app on my own machine to drive coding agents (mainly Claude Code). On that machine, the only outbound traffic I want is the agent providers' own. Upstream T3 Code ships with product analytics enabled by default and pointing at its PostHog ingest, so a stock build sends usage events unless you opt out through an environment variable. I wanted a build where telemetry is off by default and has nowhere to go even if something turns it back on.

The second change is only needed to produce that build on Windows.

## What changed

| Commit                                                              | File                                            | Change                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `chore(local): telemetry off by default and no PostHog destination` | `apps/server/src/telemetry/AnalyticsService.ts` | Default `T3CODE_TELEMETRY_ENABLED` flipped from `true` to `false` (both in the env config and in the host-preference check). Default PostHog key emptied, default host changed from `https://ingest.t3.codes` to `http://127.0.0.1:9` (the discard port on localhost). |
| `chore(local): raise Windows payload file guard for local build`    | `scripts/build-desktop-artifact.ts`             | `WINDOWS_PACKAGED_PAYLOAD_FILE_LIMIT` raised from `80` to `100`.                                                                                                                                                                                                       |

### Telemetry

Three layers, so no single setting re-enables sending data:

1. **Disabled by default.** With no env var set, the analytics service starts disabled.
2. **No key.** The baked-in PostHog project key is gone; an empty key means events have no project to land in.
3. **Dead-end host.** If telemetry were enabled anyway, events would go to `127.0.0.1:9`, which drops them locally instead of reaching `ingest.t3.codes`.

All three are still regular config defaults: setting `T3CODE_TELEMETRY_ENABLED`, `T3CODE_POSTHOG_KEY` and `T3CODE_POSTHOG_HOST` explicitly overrides them, exactly as upstream.

### Windows build guard

The desktop build script checks that the packaged Windows server payload stays under a file-count limit. A local Windows build crossed upstream's limit of 80 files and the build stopped there. Raising it to 100 lets the local artifact build; nothing else in packaging changes.

## How to build

Same as upstream (see [README_old.md](./README_old.md) and [docs/](./docs/README.md)). The desktop artifact is produced by `scripts/build-desktop-artifact.ts`.

## Keeping up with upstream

The changes are two isolated commits, so updating is a rebase onto the new upstream `main`:

```sh
git remote add upstream https://github.com/pingdotgg/t3code.git
git fetch upstream
git rebase upstream/main
```

If upstream changes the telemetry config shape, re-apply the same three defaults by hand.

## License

MIT, same as upstream. See [LICENSE](./LICENSE).
