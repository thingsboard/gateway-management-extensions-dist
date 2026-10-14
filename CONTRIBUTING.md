# Contributing

This repository holds the built distribution of
[gateway-management-extensions](https://github.com/thingsboard/gateway-management-extensions):
the compiled `gateway-management-extension.js`, the widget bundles and the
Gateways dashboard. Source changes to the extension itself are made in that
repository; see its `CONTRIBUTING.md`.

## Branches

There are two working branches. Every change lands in one of them first, and
the release branches are only ever updated from them by pull request.

| Branch | ThingsBoard | JS built from (source repo) | Angular | Released to |
|---|---|---|---|---|
| `rc-lts` | 4.2.x, 4.3.x | `master` | 18 | `release/4.2`, `release/4.2.0`, `release/4.3.0` |
| `rc` | 4.4.x | `release/4.4` | 20 | `release/4.4.0` |

`release/4.2`, `release/4.2.0` and `release/4.3.0` must always have identical
content. Always update all three together.

`release/3.9`, `release/4.0` and `release/4.0.0` hold older releases and are
not updated by this flow. `main` and `develop` have not been updated since
2025 and are not part of it either.

All `release/*` branches are protected: they accept changes only through a pull
request.

### Why the JS differs between the lines

ThingsBoard moved to Angular 20 inside patch releases (4.2.2 and 4.3.1), so the
version number alone does not tell you which Angular a platform runs. The
source repository builds the 4.2/4.3 line on Angular 18 against the lowest
supported platform, so that one build runs on every 4.2.x and 4.3.x release. An
Angular 18 build also runs on the Angular 20 patch releases.

You can tell the two builds apart:

| Build | Marker |
|---|---|
| `rc-lts` (Angular 18) | contains an `@license Angular v18.2.6` header |
| `rc` (Angular 20) | imports `@angular/core/rxjs-interop` in its `System.register([...])` list |

Neither build may contain `ngDevMode`. If it does, the build is broken and
throws `ngDevMode is not defined` on the platform.

## What may differ between `rc-lts` and `rc`

Only these two files may differ. Anything else that diverges is a bug.

| File | Why it differs |
|---|---|
| `resources/gateway-management-extension.js` | Different source branch and Angular version (see above). |
| `dashboards/gateways_dashboard.json` | `rc` uses features that exist only in ThingsBoard 4.4. |

The ThingsBoard 4.4-only dashboard features are:

- the "Manage by agent" and "Auto-provision" actions, which call
  `agentDeployDialogService` and `agentAutoProvisionDialogService`
- the dialog toolbars styled with `color="tb-primary"` and the
  `tb-bottom-border` class

None of these exist in ThingsBoard 4.2 or 4.3, so they must never reach
`rc-lts`. Check before merging:

```bash
git diff --name-only origin/rc-lts origin/rc
git show origin/rc-lts:dashboards/gateways_dashboard.json \
  | grep -cE 'agentDeployDialogService|agentAutoProvisionDialogService|tb-bottom-border|color=\\"tb-primary\\"'
# must print 0 (the tb-primary-fill classes are fine on 4.2/4.3)
```

## Where to make a change

### A change for both lines

**Open the pull request against `rc-lts`.** Once it is merged, **merge
forward** into `rc` (see below). This is the same direction the source
repository uses (`master` first, then forward into `release/4.4`).

**Do not cherry-pick between `rc-lts` and `rc`.** Cherry-picking makes git
treat the same change as two unrelated ones, and every later merge conflicts
with itself.

### A change only for ThingsBoard 4.4

Open the pull request against `rc`. It never goes to `rc-lts`.

### A new JS build

Each line commits its own build, because the two are built from different
source branches:

1. In the source repository, check out the branch from the table above
   (`master` for `rc-lts`, `release/4.4` for `rc`) and build it:

   ```bash
   yarn install && yarn build
   ```

   The result is `target/generated-resources/gateway-management-extension.js`.
2. Check the markers above (Angular version, no `ngDevMode`).
3. Copy it to `resources/gateway-management-extension.js` on a branch from
   `rc-lts` or `rc`. Name the source commit in the commit message, for example
   `Updated build from gateway-management-extensions master (db5e414)`.
4. Open the pull request against that branch.

A source change that reaches both source branches therefore needs two build
pull requests here: one into `rc-lts` and one into `rc`.

## Merging `rc-lts` forward into `rc`

`resources/gateway-management-extension.js` conflicts on every merge, because
the two builds always differ. **Resolve it by keeping the `rc` build.** Do this
on a separate branch rather than in `rc-lts`, so that `rc-lts` never receives
anything from `rc`:

```bash
git fetch origin
git switch -c merge/rc-lts-into-rc-$(date +%Y%m%d) origin/rc
git merge origin/rc-lts
git checkout --ours resources/gateway-management-extension.js
git add resources/gateway-management-extension.js
git commit --no-edit
git push -u origin HEAD
gh pr create --base rc --title "Merge rc-lts into rc"
```

The dashboard merges on its own: `rc-lts` never touches the 4.4-only parts, so
git keeps them from `rc`. After the merge, `git diff --stat origin/rc HEAD`
should show only the changes you expect to bring in.

Delete the `merge/...` branch once the pull request is merged.

**Never merge `rc` into `rc-lts`.** That would bring the 4.4-only dashboard
features and the Angular 20 build into the 4.2/4.3 line.

## Releasing

Open one pull request per release branch, with the working branch as the head:

```bash
# 4.4 line
gh pr create --base release/4.4.0 --head rc --title "Merge rc into release/4.4.0"

# 4.2/4.3 line
for b in release/4.2 release/4.2.0 release/4.3.0; do
  gh pr create --base "$b" --head rc-lts --title "Merge rc-lts into $b"
done
```

Merge them with a merge commit, not squash or rebase, so the release branch
keeps the same commits as the working branch.

Before releasing, test the build on a real platform. For the 4.2/4.3 line this
means one platform on Angular 18 (4.2.0–4.2.1.x or 4.3.0.x) and one on Angular
20 (4.2.2+ or 4.3.1+).
