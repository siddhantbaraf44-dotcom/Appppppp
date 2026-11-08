# Ice

**Ice** is a mobile browser for Android built on the Firefox **Gecko** engine (GeckoView),
with a built-in ad blocker and support for **Firefox extensions**.

*Why "Ice"?* Ice uses the same rendering engine as Firefox, so Firefox add-ons
(`.xpi` packages) work inside it — you can install extensions straight from
`addons.mozilla.org`.

## Features

- **Rebranded end to end**: app name *Ice*, ice/snowflake logo, adaptive icon (with Android 13
  themed-icon support), navy-and-ice UI and an "ice" wordmark on the home page.
- **Persistent full screen** — once enabled, mobile Back never leaves or minimises full screen;
  only the on-screen **exit full screen** button does (see below).
- **Install Firefox extensions from a link** — paste an `addons.mozilla.org` page link, a direct
  `.xpi` link, or just the add-on's name.
- Light/dark UI that follows the system theme; Chrome-style top bar (Home, address bar, Tabs, Menu).
- Tab stack on the left edge, tab switcher grid, private tabs, links that open in new tabs.
- Tabs are restored when the app restarts (private tabs are not saved).
- Bookmarks, history, find in page, desktop site, share, downloads.
- Background play (only while audio is actually playing).
- Search-engine picker (DuckDuckGo, Google, Bing, Brave, Yahoo, Ecosia, Startpage, YouTube, Wikipedia)
  with optional suggestions (off in private tabs).
- Home page with your own shortcuts and an optional custom home page.
- Add-ons manager: uBlock Origin is built in (also active in private tabs) and any other Firefox
  extension can be added.

All data (bookmarks, history, settings, open tabs) stays on the device. Nothing is synced anywhere.

## Full-screen behaviour

| Action | Result |
| --- | --- |
| Menu → **Full screen** | Toolbar and system bars hide; the floating exit button appears |
| **Back button / Back gesture** while full screen | Goes back **inside the page** if there is history, otherwise nothing. It never exits full screen, never closes the tab and never minimises the app. |
| **Page video** going full screen | Same rule: Back stays in full screen |
| Tapping the **exit full screen** button (floating button on the right) | The only way out of full screen |

The choice is remembered, so full screen also survives rotation and an app restart, and the
system bars are re-hidden if anything brings them back.

Implementation notes (`MainActivity.kt`):
`dispatchKeyEvent` swallows Back before it reaches the page or the system (all API levels),
`OnBackInvokedDispatcher` (`PRIORITY_DEFAULT`) covers the Android 13+ predictive-back gesture,
and the window gets `FLAG_FULLSCREEN` + immersive system-UI flags without the *sticky* behaviour,
so nothing restores the bars by itself. `Back` is only allowed to leave the app when the user is
**not** in full screen.

## Install Firefox extensions from a link

Menu → **Add-ons** → **Install from link**, then paste any of:

| You paste | Ice installs from |
| --- | --- |
| `https://addons.mozilla.org/firefox/addon/darkreader/` | `.../firefox/downloads/latest/darkreader/latest.xpi` |
| `https://addons.mozilla.org/android/addon/ublock-origin/` | the Android build of that add-on |
| `https://addons.mozilla.org/f/abc123` (short link) | the linked add-on |
| `https://example.com/extension.xpi` | that exact file |
| `darkreader` (just the add-on name) | the matching `addons.mozilla.org` add-on |

If an add-on page is already open, the dialog offers "use the open page". Install errors are shown
with the resolved URL so you can see what was tried.

Installed extensions are remembered and re-installed automatically the next time Ice starts
(GeckoView keeps them for one session otherwise). Removing an extension from the Add-ons list
also removes it from that list.

> Extensions must be built for Android/Firefox for Android to work in GeckoView; desktop-only
> add-ons will report that they are not compatible.

## Build the APK with GitHub Actions

1. Create a GitHub repository and upload the project (see *Uploading* below).
2. Keep `.github/workflows/build.yml` — that is the build recipe.
3. Push. Every push starts **Actions → Build Ice APK**.
4. Open the finished run and download the **`Ice-debug-apk`** artifact — it contains
   `Ice-<run-number>.apk`, ready to install.

The workflow sets up JDK 17, the Android SDK (platform 34 + build-tools), downloads uBlock Origin
into `app/src/main/assets/extensions/ublock/`, then runs `gradle assembleDebug`.

### Building locally

```bash
# Gradle 8.7+ and JDK 17, with Android SDK platform 34 installed
gradle assembleDebug
# APK: app/build/outputs/apk/debug/app-debug.apk
```

You can also just open the folder in Android Studio and press **Run**.

`app/src/main/assets/extensions/ublock/` is filled by the workflow (it is git-ignored), so a local
build without it simply starts without the built-in ad blocker.

## Uploading to GitHub directly (ZIP)

1. Create a new repository on GitHub.
2. **Add file → Upload files**, then drop **`Ice-github.zip`** (or `Ice-source.zip`) and commit.
3. Go to **Actions → Build Ice APK**. The run finds the ZIP in the repository, unpacks it and
   builds the APK — no need to extract it locally or on GitHub.
4. Download **`Ice-debug-apk`** from the run.

If you instead push the unpacked project (or use GitHub Desktop / `git push`), the same workflow
detects `settings.gradle.kts` at the root and builds directly.

`Ice-github.zip` contains the project at the top of the archive (`Ice/settings.gradle.kts`, …),
`Ice-source.zip` contains it at the root — the workflow handles both.

## Project layout

```
app/src/main/java/com/ice/browser/
    MainActivity.kt            UI, tabs, full screen, add-ons
    BackgroundPlayService.kt   foreground service for background audio
    Store.kt                   on-device settings / bookmarks / history
    RailScrollView.kt          floating tab rail
app/src/main/res/drawable*/    icons, ic_launcher_foreground (snowflake)
.github/workflows/build.yml    GitHub Actions build
```

## License / credits

Browser UI code: this project. Engine: [GeckoView](https://mozilla.github.io/geckoview/) by Mozilla.
uBlock Origin is © Raymond Hill, GPL-3.0, downloaded at build time.
