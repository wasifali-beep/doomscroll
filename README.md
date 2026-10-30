# Seekho

Doomscroll replacement. Same swipe habit, but every card teaches one tech concept
in Roman Urdu, with English technical terms. Daily goal reached, feed stops.

## Features

- **Swipe feed.** One concept per card: what, why, a desi example, one-line takeaway.
- **Topics.** Backend/networking, databases, mixed tech, plus any custom topic you type.
- **Levels.** Beginner, Intermediate, Advanced.
- **Card actions.** Asaan karo (simplify), Detail mein (expand), Aur misaal (examples),
  Use case, Sawal pucho (ask anything about the card).
- **Quizzes.** One after every 5 cards, "Yaad hai?" checks on older cards, and a full
  "Mera test lo" test.
- **Spaced repetition.** Each card returns after 1, 3, 7, 14, 30 days. A wrong answer
  resets it to 1 day.
- **Saved cards** (★), **stats** (📊): streak, accuracy, mastered cards, 7-day chart, topics.
- **Daily reminder** notification at a time you pick.
- **Voice.** Cards and answers read aloud.
- **Offline pack.** Download 20 cards to read without internet.

## Install

1. Open the repo's **Releases** page.
2. Download `seekho-vX.Y.Z.apk` from the latest release.
3. Open it on your phone. Allow "install unknown apps" if Android asks.
4. Updating: install the newer APK over the old one. Your data stays.

Optional: check the download against `seekho-vX.Y.Z.apk.sha256`.

## First-time setup in the app

Seekho needs an AI key. The key stays on your phone only.

| Provider | Where to get a key | Cost |
|---|---|---|
| Anthropic Claude (default) | platform.claude.com → Settings → API keys | Prepaid credit |
| Google Gemini | aistudio.google.com → Get API key | Free tier |

In the app: ⚙︎ → choose provider → paste key → **Save karo**.
Default models: `claude-haiku-5-5`, `gemini-flash-latest`. Change the model name in
Settings if the provider renames it.

## Privacy

- No server. The app talks straight from your phone to the AI provider you chose.
- Key, cards, progress and settings live in the app's local storage.
- Uninstalling the app deletes everything.

## Project layout

```
www/index.html                    The whole app (HTML, CSS, JS in one file)
package.json                      Version number + Capacitor dependencies
capacitor.config.json             App id (com.seekho.app), name, native HTTP
.github/workflows/build.yml       Dev build on every push → debug APK artifact
.github/workflows/release.yml     Release build → signed APK on the GitHub Release
.github/workflows/signing-key.yml One-time signing key generator
CHANGELOG.md                      What changed in each version
docs/RELEASING.md                 How to ship a new version
docs/ROADMAP.md                   Planned versions
```

Stack: plain HTML/JS wrapped by [Capacitor](https://capacitorjs.com) 8. The `android/`
folder is generated in CI and is not committed.

Plugins: `@capacitor/local-notifications` (reminder),
`@capacitor-community/text-to-speech` (voice).

## Builds

| Build | When | Where to download | Signed with |
|---|---|---|---|
| Dev | Every push to `main`, PRs, manual | Actions → run → Artifacts → `seekho-vX.Y.Z-dev.N-debug.apk` | Throwaway debug key |
| Release | Publishing a GitHub Release | Releases → `seekho-vX.Y.Z.apk` | Your release key |

Dev builds are for testing. Each one has a different debug key, so uninstall before
switching between a dev build and a release build.

Shipping a release: see [docs/RELEASING.md](docs/RELEASING.md).

## Local preview

Open `www/index.html` in Chrome. Everything except reminder works. Voice uses the
browser's speech engine.

Local Android build (optional; needs Node 22, JDK 21, Android SDK):

```
npm install
npx cap add android
npx cap sync android
cd android && ./gradlew assembleDebug
```

## Troubleshooting

| Problem | Fix |
|---|---|
| "Key ya model galat hai" | Re-paste the key. Check credit balance. Check model name. |
| "Model nahi mila" | Provider renamed the model. Put the current name in Settings. |
| No reminder | Allow notifications for Seekho in Android settings. Turn off battery optimisation for Seekho. |
| No voice | Install or update "Speech Services by Google". |
| "App not installed" on update | The APK was signed with a different key. Uninstall, then install. |
| Release workflow fails at "Check tag and version" | `package.json` version must equal the tag without `v`. |
| Release workflow fails at "Check signing secrets" | Do the one-time setup in docs/RELEASING.md. |
