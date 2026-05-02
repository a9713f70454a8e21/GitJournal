<p align="center">
  <img width="400" height="auto" src="https://gitjournal.io/images/logo.png">
  <br/>Mobile first Markdown Notes integrated with Git</b>
</p>

<p align="center">
  <a href="https://play.google.com/store/apps/details?id=io.gitjournal.gitjournal&utm_source=github&utm_medium=link"><img alt="Get it on Google Play" src="https://gitjournal.io/images/android-store-badge.png" height="75px"/></a>
  <a href="https://apps.apple.com/app/gitjournal/id1466519634&utm_source=github&utm_medium=link"><img alt="Download on the App Store" src="https://gitjournal.io/images/ios-store-badge.svg" height="75px"/></a>
</p>

<p align="center">
  <a href="https://circleci.com/gh/GitJournal/GitJournal"><img alt="Build Status" src="https://circleci.com/gh/GitJournal/GitJournal.svg?style=svg"/></a>
  <a href="https://www.gnu.org/licenses/agpl-3.0"><img alt="License: AGPL v3" src="https://img.shields.io/badge/License-AGPL%20v3-blue.svg"></a>
  </br>
  <a href="https://api.reuse.software/info/github.com/GitJournal/GitJournal"><img alt="REUSE status" src="https://api.reuse.software/badge/github.com/GitJournal/GitJournal"></a>
  <a href="https://github.com/sponsors/vHanda"><img alt="Donate via GitHub" src="https://img.shields.io/badge/Sponsor-Github-%235a353"></a>
  </br>
</p>

# Summary

GitJournal is a note taking app focused on privacy and data portability. It stores all its notes in a standardized Markdown + YAML header format (optional). The notes are stored in a Git Repo of your choice - GitHub / Gitlab / Custom-provider. This means you can easily self host or host your notes in one of the many [Git providers](./docs/git_hosts.md).

# Screenshots

<p float="left">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-1.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-2.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-4.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-5.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-6.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-7.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-16.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-11.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-13.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-12.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-18.png" width="240" height="auto">
<img src="https://gitjournal.io/screenshots/android/2020-06-04/en-GB/images/phoneScreenshots/Nexus 6P-20.png" width="240" height="auto">
</p>

# Build Linux amd64 .deb

The following commands build a release Linux desktop bundle and package it as an
`amd64.deb`. They were verified on Ubuntu 24.04 in GitHub Codespaces.

Requirements:

- Ubuntu/Debian amd64 environment
- Network access for installing packages and running `flutter pub get`

Copy and run from the repository root. The script reuses `flutter` if it is
already available; otherwise it installs Flutter 3.41.5 into `$HOME/sdks/flutter`.

```bash
set -euxo pipefail

export DEBIAN_FRONTEND=noninteractive
sudo apt-get update
sudo apt-get install -y \
  clang \
  cmake \
  dpkg-dev \
  fakeroot \
  git \
  libblkid-dev \
  libglu1-mesa \
  libgtk-3-dev \
  libjsoncpp-dev \
  liblzma-dev \
  ninja-build \
  pkg-config \
  unzip \
  xz-utils

FLUTTER_VERSION=3.41.5
FLUTTER_HOME="$HOME/sdks/flutter"

if ! command -v flutter >/dev/null 2>&1; then
  if [ ! -x "$FLUTTER_HOME/bin/flutter" ]; then
    rm -rf "$FLUTTER_HOME"
    git clone --depth 1 --branch "$FLUTTER_VERSION" \
      https://github.com/flutter/flutter.git "$FLUTTER_HOME"
  fi
  export PATH="$FLUTTER_HOME/bin:$PATH"
fi

flutter --version
flutter config --enable-linux-desktop --no-analytics

# Generate the local environment file. If secrets/env.json is unavailable or
# still encrypted, create the minimum local file required by error reporting.
if [ -s secrets/env.json ]; then
  dart ./scripts/setup_env.dart || true
fi

if ! grep -q 'sentry' lib/.env.dart 2>/dev/null; then
  cat > lib/.env.dart <<'EOF'
class Env {
  static final String sentry = "";
}
EOF
fi

flutter pub get --suppress-analytics
flutter build linux --release

APP=gitjournal
VERSION="$(grep '^version:' pubspec.yaml | awk '{print $2}' | cut -d+ -f1)"
PKGVER="${VERSION}-local1"
STAGE="/tmp/${APP}_${PKGVER}_amd64"
OUT="$(pwd)/${APP}_${PKGVER}_amd64.deb"

rm -rf "$STAGE" "$OUT"
install -d -m 0755 \
  "$STAGE/DEBIAN" \
  "$STAGE/opt/gitjournal" \
  "$STAGE/usr/bin" \
  "$STAGE/usr/share/applications" \
  "$STAGE/usr/share/icons/hicolor/256x256/apps"

cp -a build/linux/x64/release/bundle/. "$STAGE/opt/gitjournal/"
ln -s /opt/gitjournal/gitjournal "$STAGE/usr/bin/gitjournal"
cp assets/icon/icon.png "$STAGE/usr/share/icons/hicolor/256x256/apps/gitjournal.png"

cat > "$STAGE/usr/share/applications/io.gitjournal.journal.desktop" <<'EOF'
[Desktop Entry]
Type=Application
Name=GitJournal
Comment=Markdown notes with Git sync
Exec=/opt/gitjournal/gitjournal
Icon=gitjournal
Terminal=false
Categories=Office;Utility;
StartupWMClass=gitjournal
EOF

cat > "$STAGE/DEBIAN/control" <<EOF
Package: gitjournal
Version: $PKGVER
Section: utils
Priority: optional
Architecture: amd64
Maintainer: GitJournal contributors <noreply@example.com>
Depends: libgtk-3-0 | libgtk-3-0t64, libblkid1, liblzma5, libstdc++6, libc6
Description: GitJournal Linux desktop package
 GitJournal packaged from a Flutter Linux release build.
EOF

chmod -R go-w "$STAGE/DEBIAN"
find "$STAGE/DEBIAN" -type d -exec chmod 0755 {} +
find "$STAGE/DEBIAN" -type f -exec chmod 0644 {} +

fakeroot dpkg-deb --build "$STAGE" "$OUT"
dpkg-deb --info "$OUT"
ls -lh "$OUT"
sha256sum "$OUT"
```

The resulting package will be written to the repository root, for example:

```text
gitjournal_1.89.0-local1_amd64.deb
```

Install it locally with:

```bash
sudo apt install ./gitjournal_*-local1_amd64.deb
```

# Migrating from Existing Apps

- [Google Keep](https://github.com/vHanda/google-keep-exporter)
- [Day One Classic](https://gist.github.com/sanzoghenzo/fb5011aa566292a4eb1b62fc7a4a50cc)
- [Narrate](https://gist.github.com/sanzoghenzo/fb5011aa566292a4eb1b62fc7a4a50cc)
- [Simplenote](https://github.com/isae/gitjournal-simplenote-exporter)

# Contributing

Please feel free to [open an issue](https://github.com/GitJournal/GitJournal/issues/new) for any bug or feature request. Additionally, you can vote on existing [Issues](https://github.com/GitJournal/GitJournal/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc) by reacting with a '👍'.

# License

All code contributed by [Vishesh Handa](https://github.com/vhanda) is licensed under the [AGPL](https://www.gnu.org/licenses/agpl-3.0.en.html). All code contributed by anyone else is licensed under the Apache License 2.0. This is done so that GitJournal can avoid needing a CLA, and it can be distributed it on the Apple App Store which doesn't allow AGPL.

The documentation (including this file) and translations are under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
