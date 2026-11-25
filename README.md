# victory_builder

This is not a VIctory project or a game project. It is a toolkit prepared to quickly set up a build environment. Extract these files into a folder, copy your chosen VIctory project into `source/`, then run `build.cmd`.

Build supported VIctory projects on Windows without editing their source code or installing a separate Rust or Visual Studio development environment beforehand.

VIctory is a compatibility layer that runs Xbox Series X|S game code on Windows and adapts console APIs, graphics commands, shaders and data to the host platform. This toolkit builds VIctory and selected forks. It does not contain the game, game assets, or the VIctory application source.

## Quick start

1. Download or clone the complete toolkit, including `compiler/`.
2. Download the VIctory project you want to build.
3. Copy the contents of its actual Cargo workspace into `source/`. The file `source/Cargo.toml` must exist directly at that location.
4. Run `build.cmd` from the toolkit root. If a required shared tool is missing, approve its installation when Windows requests administrator permission.
5. Find the built application in `release/`.

For example, a downloaded archive may contain an extra `VIctory-main/` directory. Copy the files inside that directory into `source/`, rather than placing the whole directory there.

```text
compiler/
    windows/
        bin/
        msvc-14.51.36231-sdk-10.0.28000.0-windows-x64-x86-x64.7z
        rust-1.100.0-sdk-windows-x64.7z
        victory-deps-<lockfile-hash>.7z
source/
    Cargo.toml
    Cargo.lock
    apps/
    crates/
build.cmd
README.md
```

The toolkit repository is intended to ship with an empty `source/` directory. Populate it locally with your selected project. Git does not track empty directories; create `source/` if your download or checkout does not contain it.

## Requirements

- Windows 10 or Windows 11, x64.
- Windows PowerShell 5.1 and standard Windows command-line tools.
- Sufficient disk space for compiler installation, source files and temporary build outputs.
- Administrator permission only when installing missing tools into the shared tool repository.

Building the CLI does not require running the game. Playing requires your own game files and the hardware supported by your selected fork. Consult that project's documentation for CPU, GPU, memory and runtime requirements.

## Tested projects

The following local source snapshots were successfully built on Windows x64 on November 25, 2026, using the same `build.cmd` and unchanged application sources.

| Project | Cargo.lock SHA-256 prefix | Offline release build | CLI `--help` |
| --- | --- | --- | --- |
| Upstream VIctory | `6598e0bf61531022` | Passed | Passed |
| VIctory Belobug AMD fork | `2b1d02f15e02f9a8` | Passed | Passed |
| VItoriaBr | `bfbbb8b827f71a48` | Passed | Passed |
| victory-fork | `bfbbb8b827f71a48` | Passed | Passed |
| ztrx | `13bb91f9d2816440` | Passed | Passed |

These results apply to the tested snapshots. A newer revision may change dependencies or require additional tools. Successful compilation and CLI checks do not establish that a particular game build will start or run correctly. Game execution and bootstrap on a fresh Windows installation were not part of this validation.

The unrelated React charting library named Victory is not supported by this toolkit.

## Build behavior

`build.cmd` performs tool discovery, optional installation, compilation, CLI verification and packaging. It does not edit the selected project's source files.

- Reuses tools under `%ProgramFiles%\TeTeOS.Net\toolchains`.
- Uses Rust 1.100.0, MSVC 14.51.36231 and Windows SDK 10.0.28000.0 from the bundled toolchain layout.
- Includes directly executable 7-Zip files for initial archive extraction.
- Elevates only the installation child process when shared tools are missing. Compilation continues as the original user.
- Reads Cargo workspace metadata and selects the `victory` application binary. If that name is absent, it accepts a single application binary, excluding `xtask`. Ambiguous workspaces stop with an error.
- Builds with `--release --locked --offline` for `x86_64-pc-windows-msvc`, including any features required by the chosen binary target.
- Uses static CRT linkage and checks the resulting CLI with `--help` before packaging.
- Keeps intermediate files and complete compiler diagnostics under a unique `%TEMP%\victory-build-<id>/` directory. The console reports the full `build.log` location.
- Pauses before exiting so the result remains visible when launched by double-clicking.

Warnings remain available in the full log. Normal console output focuses on build stages; a failed operation returns a nonzero exit code and displays diagnostics.

Once the tools have been installed and a usable `7z.exe` is available through PATH, subsequent builds can work without `compiler/`. Keep the complete directory when distributing the toolkit to another computer.

## Output and launcher

A successful build produces:

```text
release/
    victory.exe
    _start.cmd
    licenses.md
```

The executable name follows the selected Cargo binary target. For the supported projects, it is `victory.exe`.

The generated `_start.cmd` contains:

```bat
@echo off
cd /d "%~dp0"
victory.exe run "%~dp0game_scarlett_master.exe" --execute
```

To use this launcher, copy the generated distribution files into the folder containing your own `game_scarlett_master.exe`, or place your files together in another suitable directory outside the toolkit repository.

The launcher supplies an absolute game path. This avoids an upstream path-handling issue where a bare executable filename can produce an empty virtual package-drive root. No patch to VIctory's source is needed for this workaround.

The build itself does not launch the game or copy game files into the distribution.

## Offline dependencies and other revisions

Each dependency archive is named using the first 16 hexadecimal characters of the SHA-256 hash of `source/Cargo.lock`:

```text
compiler/windows/victory-deps-<hash-prefix>.7z
```

The archive contains vendored Cargo dependency sources and consolidated distribution license texts. Multiple source projects can share one archive when their lockfiles match.

If a required dependency bundle is missing, the script reports its exact name and stops. It does not download dependencies, modify the lockfile, or substitute another bundle. Supporting a new lockfile requires preparing and supplying a matching vendor archive and its license texts; automatic bundle generation is not currently implemented.

To check the lockfile hash in PowerShell:

```powershell
(Get-FileHash -LiteralPath .\source\Cargo.lock -Algorithm SHA256).Hash.ToLowerInvariant()
```

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| `source/Cargo.toml` or `source/Cargo.lock` cannot be found | Copy the actual workspace contents directly into `source/`. |
| A `victory-deps-*.7z` bundle is missing | Use a tested source snapshot or supply the matching offline dependency archive. |
| Administrator permission is required | Allow installation of missing shared tools, then rerun the build if installation was cancelled. |
| `Please install 7zip.` | Restore the bundled extractor or make a working 7-Zip installation available. |
| Compilation fails | Read the full temporary `build.log`; the selected fork may require tools or changes outside the tested setup. |
| The application builds but the game crashes | Check the selected fork's runtime requirements and crash logs. Compilation alone does not validate game compatibility. |

## Compiler archives on GitHub

The `compiler/` directory is part of the toolkit distribution and its archive contents are preserved. The MSVC and Rust archives exceed GitHub's 100 MiB regular Git file limit, so publishing them in the repository requires Git LFS.

For a Git checkout, install Git LFS and run the builder. The builder downloads any compiler archive placeholders and configures LFS locally without adding tracking configuration files to the published project:

```bat
git lfs install
build.cmd
```

A Git LFS pointer file is not a usable compiler archive. If downloading a repository ZIP, check that the compiler files contain the actual archives. See [GitHub's large-file documentation](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github).

## License

The original code and documentation of this build toolkit are licensed under the [MIT License](LICENSE), copyright 2026 Victory Windows Build Toolkit contributors. You may use, modify, redistribute and sell them, including as part of proprietary software, provided that the copyright and license notice are retained. Modifications do not have to be published.

This MIT license does not relicense any third-party software, compiler, SDK, dependency, notice, or VIctory source supplied separately. It grants no additional rights to those components.

## Third-party licenses

The selected VIctory project's own license applies to its application sources and resulting program. The tested VIctory family projects use GPL-3.0-or-later.

Bundled compilers, SDK components, 7-Zip and Cargo dependencies retain their respective licenses and notices. Preserve the license files supplied with those packages. `release/licenses.md` contains consolidated application and Cargo dependency license texts for the selected dependency bundle; it does not replace the separate terms governing development tools.

Redistribution of each bundled development tool must be permitted by its own license, with all required notices and other obligations preserved. In particular, the toolkit's MIT license is not permission to redistribute the complete MSVC/Windows SDK archive; that archive's applicable redistribution rights must be established before publishing it.
