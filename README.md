# qt-static

Prebuilt [vcpkg](https://vcpkg.io) install roots (`x64-windows-static-md`, built on the `windows-2022` runner image) published as release assets, so consumer CI never compiles Qt from source.

## Consuming

1. Download all volumes of a bundle from [Releases](../../releases) plus `SHA256SUMS.txt`.
2. Verify hashes, then extract **into your vcpkg install root** (the directory containing `x64-windows-static-md/`):
   ```powershell
   7z x vcpkg-qt6-win2022-...-x64-windows-static-md.7z.001 -o<vcpkg install root>
   ```
3. Run `vcpkg install` with `--x-install-root=<same root>`: it reads the shipped `vcpkg/status` and skips every bundled port.

No authentication: public assets are anonymous direct downloads (`curl -LO https://github.com/kkzi/qt-static/releases/download/<tag>/<file>`), so fork PRs work too.

## Rebuilding

Actions tab -> `build` workflow -> Run workflow (bundle: `qt6` for [o3ku/RdpBox](https://github.com/o3ku/RdpBox), `qt5` for [o3ku/SongBird](https://github.com/o3ku/SongBird)). Takes ~1-1.5h on a cold cache; the image's own vcpkg binary cache often makes it faster.

The Qt binaries here are unmodified builds of Qt (LGPL): sources at https://www.qt.io/download/open-source and the matching vcpkg port recipes; build triplet/configuration as recorded in each release.
