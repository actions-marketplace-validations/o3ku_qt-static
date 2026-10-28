# qt-static bundle

Prebuilt static Qt binaries (Qt6/Qt5 + deps) as vcpkg install roots (`x64-windows-static-md`, built on the `windows-2022` runner). Saves the ~1-1.5h vcpkg source compile in CI.

## Usage

```yaml
steps:
  # Qt 6: qtbase, qtsvg, freerdp, cjson, nlohmann-json
  - uses: o3ku/qt-static@v1
    with:
      tag: qt6-win2022-20261024-0201
      dest: ${{ github.workspace }}/vcpkg_installed

  # Qt 5: qt5-base, qt5-svg
  - uses: o3ku/qt-static@v1
    with:
      tag: qt5-win2022-20261024-0121
      dest: ${{ github.workspace }}/vcpkg_installed
```

- `tag` — pick from [Releases](../../releases). The `qt6-` / `qt5-` prefix selects the Qt version.
- `dest` — where the bundle is extracted. Point vcpkg `--x-install-root` or CMake `VCPKG_INSTALLED_DIR` at it; bundled ports are seen as installed and skipped.

Downloads are anonymous (no tokens) — works in fork PRs. Every volume is SHA256-verified before extraction.

Rebuild / add bundles: Actions tab → **build** workflow → Run workflow (`bundle: qt6` or `qt5`).

Binaries are unmodified builds of Qt (LGPLv3/GPL) and other open-source libraries via vcpkg.
