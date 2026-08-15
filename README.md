# WholeBudget — releases

Download page and release artifacts for the WholeBudget household budgeting app.

**[Download for macOS →](https://alex-bisgould.github.io/wholebudget-releases/)**

## What this repository is (and is not)

This repository holds **only** the landing page (`index.html` and
`whats-new.html`, served by GitHub Pages) and the packaged artifacts attached
to each release. **The application source lives in a separate private
repository** — nothing in this repo is buildable, and no code, data or
credential belongs here.

It is public for one reason: so that anyone can download a release and so the
in-app updater can read the release feed without a token.

## Release assets — the names are load-bearing

Every release attaches exactly three assets. **Never rename them.** Each name
is depended on by something that will break silently, not loudly:

| Asset | Depended on by |
| --- | --- |
| `WholeBudget-mac-arm64.dmg` | The download button on this site, which links `releases/latest/download/WholeBudget-mac-arm64.dmg` — a rename turns it into a 404. Also the per-release download links on `whats-new.html`. |
| `WholeBudget-mac-arm64.zip` | The in-app updater (electron-updater) downloads and verifies this, then installs it in place. |
| `latest-mac.yml` | The update manifest. electron-updater fetches it first to learn the current version, the zip's filename and its sha512. If it is missing, renamed, or points at an asset name that does not exist, **every installed copy silently stops updating** — no error is shown to the user. |

electron-builder produces all three when `mac.target` includes both `dmg` and
`zip` and `mac.artifactName` is pinned to `${productName}-mac-${arch}.${ext}`.
If you ever change `productName`, the artifact names change with it and the
download button must be updated in the same commit.

Two further consequences worth stating plainly:

- **Publish the dmg and the zip together, every time.** A release with a dmg
  but no zip (or no `latest-mac.yml`) leaves new downloads working while
  existing installs quietly stall on the old version.
- **Don't delete or re-tag an old release** that a user might still be sitting
  on. The updater walks forward from wherever it is.

## Where user data lives

Everything a user enters — accounts, transactions, receipts, bills, mortgage
history, settings — lives in `~/Library/Application Support/WholeBudget`,
entirely outside the app bundle. Updating the app, replacing it, or deleting
and reinstalling it never touches that folder. Nothing in it is ever uploaded:
there is no account and no server.

## Publishing a new version

From the private app repository:

```bash
./scripts/release.sh 1.93.0       # bump → build → sign → notarize → publish
```

That script exports the publish token from
`~/.secrets/wholebudget/gh-publish-token` and hands it to electron-builder as
`GH_TOKEN`. The token is a fine-grained PAT scoped to **this repository only**,
with `Contents: read and write` and nothing else — it cannot touch the source
repo or any other repository on the account.

## Signing

Releases are signed with a Developer ID Application certificate, notarized by
Apple through the `wholebudget-notary` keychain profile, and stapled (both the
app and the dmg), so they open with no Gatekeeper warnings and no Terminal
steps.

## The site

`index.html` and `whats-new.html` are plain static files with no build step —
edit and push. Both read the GitHub releases API at page load, so the version
badge and the release list stay current without redeploying. Before the first
release they render an honest empty state rather than a broken one; the
download button will 404 until a release exists, which is expected.
