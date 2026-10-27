# qt-static

Prebuilt **static Qt binaries** (and common C++ dependencies), packaged as vcpkg install roots for the `x64-windows-static-md` triplet and built on the `windows-2022` GitHub runner image.

Purpose: **save the ~1-1.5h source compile** that vcpkg otherwise performs the first time a CI job (or a new machine) needs Qt static. Downloading and extracting a bundle takes minutes.

## What a bundle is

Each [Release](../../releases) tag carries one bundle: a 7z archive (split into ≤2GiB volumes) of a complete vcpkg install root — `x64-windows-static-md/` with headers, static libs, CMake config files, plus `vcpkg/status` so vcpkg itself knows the ports are installed. `SHA256SUMS.txt` lists and verifies every volume.

Two bundles exist, built by the same workflow:

| Bundle | Ports |
|---|---|
| `qt6-win2022-*` | qtbase, qtsvg, freerdp, cjson, nlohmann-json |
| `qt5-win2022-*` | qt5-base, qt5-svg |

## Use in a GitHub Actions workflow

One line, via this repo's composite action (downloads, verifies SHA256, extracts — anonymous, no tokens, so it also works in fork PRs):

```yaml
steps:
  - uses: o3ku/qt-static@main
    with:
      tag: qt6-win2022-20261024-1200   # pick a tag from the Releases page
      dest: ${{ github.workspace }}/vcpkg_installed
```

Then use the tree as your vcpkg install root. With classic vcpkg:

```yaml
  - run: vcpkg install cjson nlohmann-json --triplet=x64-windows-static-md
          --x-install-root=${{ github.workspace }}/vcpkg_installed
    # ports already in the bundle are skipped via the shipped vcpkg/status
```

Or configure CMake directly against it with the vcpkg toolchain and `-DVCPKG_INSTALLED_DIR=<dest>`.

If you prefer no action, plain steps work the same:

```yaml
  - run: |
      base=https://github.com/o3ku/qt-static/releases/download/<tag>
      curl -LO $base/SHA256SUMS.txt
      # download every .7z.NNN listed, sha256sum -c, then: 7z x <first volume> -o vcpkg_installed
```

## Use after a manual download

1. From a release, download **all** volumes `vcpkg-<tag>-x64-windows-static-md.7z.001, .002, ...` and `SHA256SUMS.txt`.
2. Verify (PowerShell: `Get-FileHash`, or `sha256sum -c SHA256SUMS.txt`).
3. Extract the **first volume** — 7-Zip pulls in the rest — into the directory you want as your vcpkg install root:
   ```powershell
   7z x vcpkg-qt6-win2022-...-x64-windows-static-md.7z.001 -oD:\deps\vcpkg_installed
   ```
4. Either:
   - run `vcpkg install <your ports> --triplet=x64-windows-static-md --x-install-root=D:\deps\vcpkg_installed` — bundled ports are recognized as installed and skipped; or
   - consume the tree without vcpkg: `cmake -DCMAKE_PREFIX_PATH=D:\deps\vcpkg_installed\x64-windows-static-md ...`

## Compatibility

- Triplet: `x64-windows-static-md` (static libs, /MD runtime), debug + release configs.
- Compiler: built with the windows-2022 image's MSVC. MSVC's stable C++ ABI keeps the libs linkable from newer MSVC versions.
- If your vcpkg expects a different port revision, vcpkg will simply rebuild that port — the bundle still covers Qt itself.

## Rebuilding / adding bundles

Actions tab → **build** workflow → Run workflow (`bundle: qt6` or `qt5`). Takes ~1-1.5h cold. Rebuild when you want to pick up a new Qt version or a new runner image generation.

## License

The binaries are unmodified builds of Qt and other open-source libraries. Qt is available under LGPLv3/GPL: sources at https://www.qt.io/download/open-source, port recipes and build configuration in the vcpkg repository (this repo's build workflow records the exact invocation per release).
