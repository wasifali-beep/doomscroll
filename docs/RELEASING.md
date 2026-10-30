# Releasing Seekho

## How versions work

- Format: `MAJOR.MINOR.PATCH` (SemVer).
  - **PATCH** (1.0.0 → 1.0.1): bug fixes only.
  - **MINOR** (1.0.1 → 1.1.0): new features, nothing breaks.
  - **MAJOR** (1.1.0 → 2.0.0): big change, e.g. stored data format changes.
- Git tag = `v` + version, e.g. `v1.1.0`.
- `package.json` → `"version"` must match the tag. The release workflow refuses to build if not.
- Android `versionCode` = `MAJOR*10000 + MINOR*100 + PATCH` (v1.2.3 → 10203).
  Set automatically. Must always go up, so never reuse or lower a version.
- Release file name: `seekho-v1.2.3.apk`, plus `seekho-v1.2.3.apk.sha256`.

## One-time setup: signing key

Android only installs an update if it is signed with the same key as the installed app.
Make the key once, keep it forever.

1. Repo → **Settings → Secrets and variables → Actions → New repository secret**.
   Name `KEYSTORE_PASSWORD`, value: a strong password (8+ chars). Save it in your
   password manager too.
2. **Actions → "Create signing key (run once)" → Run workflow.**
3. When it finishes, open the run, download artifact **signing-key**. It contains:
   - `seekho.jks`: the key. **Back it up** (password manager, private drive).
     Lose it = users must uninstall to get updates.
   - `KEYSTORE_BASE64.txt`: the same key as text.
4. New repository secret: name `KEYSTORE_BASE64`, value: full contents of
   `KEYSTORE_BASE64.txt`.
5. Delete the downloaded `KEYSTORE_BASE64.txt`. Delete the artifact from the run page.

Never commit `seekho.jks` or its base64 text. `.gitignore` already blocks them.

## Shipping a release

### Checklist
1. Changes merged to `main`, dev build green, tested on a phone.
2. In `CHANGELOG.md`, rename `## [Unreleased]` items into a new section
   `## [X.Y.Z] - YYYY-MM-DD`. Leave an empty `## [Unreleased]` on top.
3. In `package.json`, set `"version": "X.Y.Z"`.
4. Commit both: `Release vX.Y.Z`.

### Publish (GitHub website)
1. Repo → **Releases → Draft a new release**.
2. **Choose a tag** → type `vX.Y.Z` → **Create new tag on publish**. Target: `main`.
3. Title: `Seekho vX.Y.Z`.
4. Notes: leave empty to copy the CHANGELOG section automatically, or click
   **Generate release notes**, or write your own.
5. **Publish release.**
6. Actions tab: the **Release** workflow runs (~5 min). When green, the release page
   shows `seekho-vX.Y.Z.apk` and `.sha256` under Assets.

### Publish (command line, optional)
```
git tag vX.Y.Z
git push origin vX.Y.Z
gh release create vX.Y.Z --title "Seekho vX.Y.Z" --notes ""
```

### Pre-releases
Use a tag like `v1.2.0` but tick **Set as a pre-release**. Same workflow, marked as
pre-release on the page. Tag must still be `vX.Y.Z` (no `-beta` suffix), because the
version code is built from it.

## If something goes wrong

| Problem | Fix |
|---|---|
| "package.json says A but tag says B" | Delete the release and the tag. Fix `package.json`, commit, publish again. |
| Missing signing secrets | Do the one-time setup above. Then Actions → Release → Run workflow → enter the tag. |
| Build broke after publishing | Fix on `main`, bump PATCH (e.g. v1.0.1), release again. Don't move an existing tag. |
| Need to rebuild an existing release | Actions → Release → Run workflow → tag. Assets are replaced. |
| Bad release already out | Mark it pre-release or delete its assets. Ship a PATCH fix. |

## What each workflow produces

| Workflow | Trigger | Output |
|---|---|---|
| `build.yml` | Push to `main`, PR, manual | Actions artifact `seekho-vX.Y.Z-dev.N-debug.apk` (raw APK, 14 days) |
| `release.yml` | Release published, manual | Release assets `seekho-vX.Y.Z.apk` + `.sha256`, also Actions artifact |
| `signing-key.yml` | Manual, once | Artifact `signing-key` (1 day) |
