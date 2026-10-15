# Pietari's AeroSpace fork

This is a fork of [nikitabobko/AeroSpace](https://github.com/nikitabobko/AeroSpace) (MIT), kept at github.com/ppikkuaho/AeroSpace. It stays frozen on an upstream release tag plus a few fixes. Nothing updates it automatically: there is no Homebrew cask and no update check, so a new version arrives only when it is built and installed by hand as described below.

## What the `pietari` branch holds

Base: upstream tag `v0.21.3-Beta`. On top of it:

1. Three upstream commits from `main` that let the tag build with Xcode 27: `8571004d` (fix new compiler warnings), `455d991c` (ask swift for the build path), `f613da8f` (Swift 6.4.0 in `.swift-version`). The tag pins Swift 6.3.2, which crashes against the macOS 27 SDK.
2. Upstream PR [#2316](https://github.com/nikitabobko/AeroSpace/pull/2316) (`0a756f9f`, Martin Pucik): apps that macOS reports with process ID -1 are matched to their real process through the owner of their windows. On macOS 27 Safari is such an app, so without this AeroSpace ignores Safari.

`git log v0.21.3-Beta..pietari` lists the exact commits.

## Build

Dependencies (once): Xcode 27, `brew install bash fish swiftly ruby@3.4 xcbeautify`, `swiftly init --no-modify-profile --skip-install`, then `swiftly install` in the repo (reads `.swift-version`), and Rust (`cargo`) for shell completion.

```
cd ~/Documents/aerospace
export PATH="/opt/homebrew/opt/ruby@3.4/bin:/opt/homebrew/bin:$PATH"
./build-release.sh --build-version 0.21.3-Beta-pietari \
  --codesign-identity "Apple Development: pietari.pikkuaho@gmail.com (GXX559D62R)"
```

The result is in `.release/AeroSpace-v<version>/`. Tests: `swiftly run swift test` (382 XCTest tests pass). `./test.sh` also runs `lint.sh`, whose periphery step fails on Swift 6.4 at this base; upstream fixed that later in `c1742862`.

## Install

```
osascript -e 'quit app "AeroSpace"'
R=.release/AeroSpace-v0.21.3-Beta-pietari
rm -rf /Applications/AeroSpace.app && ditto "$R/AeroSpace.app" /Applications/AeroSpace.app
cp "$R/bin/aerospace" ~/.local/bin/aerospace
open -a AeroSpace
```

The config stays at `~/.aerospace.toml`. Rebuilding with the same signing identity keeps the Accessibility grant. A build signed by anyone else needs Accessibility granted again in System Settings, Privacy & Security, Accessibility. Man pages and shell completion are in the release folder but are not installed.

## Update deliberately

1. `git fetch upstream --tags` and pick a new release tag.
2. `git rebase --onto <new-tag> v0.21.3-Beta pietari`, dropping the toolchain commits and any PR that the new tag already contains. If PR #2316 was merged upstream, drop it too.
3. Build, run the tests, install as above, check that `aerospace list-windows --all` shows Safari.
4. Update this file and `git push --force-with-lease origin pietari`.
