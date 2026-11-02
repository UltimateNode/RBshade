# Contributing

Bug reports, presets, and code changes are welcome. Open an issue before starting large changes so we can agree on the approach first.

## Report a bug

Use the bug report form. Include the host version, how you installed it, your Windows version, and `Unishade.log` from the host's folder. Installer problems need the setup log, `%TEMP%\Unishade-Setup.log`.

## Add a preset

Presets live in `presets/`. Every effect a preset uses must come from [cattools.cc's official package list](https://cattools.cc); the installer downloads all of those and nothing else. Do not reference shaders from other sources.

1. Save the preset from cattools.cc and copy the `.ini` into `presets/`.
2. Test it with the host on a supported game and name the game in your submission. Roblox provides no depth buffer; depth effects require the optional depth estimation add-on.
3. Add its filename and SHA-256 to `presets/downloads.ini`:

   ```powershell
   (Get-FileHash presets/MyPreset.ini).Hash.ToLower()
   ```

4. Open a pull request with a screenshot or short description of the look.

The installer verifies the hash, so the file must not change after the hash is recorded.

## Build

Visual Studio 2022 with **Desktop development with C++**, a recent Windows SDK, and CMake 3.20 or newer.

```powershell
cmake -S . -B build -A x64 -DBUILD_TESTING=ON
cmake --build build --config Release --parallel
ctest --test-dir build -C Release --output-on-failure
```

The EXE is at `build\Release\Unishade.exe`. See `installer/README.md` for building and testing the installer, and [src/posix](src/posix/README.md) for macOS and Linux.

cattools.cc Actions builds and tests the Windows, Linux and macOS versions for pushes and pull requests, and runs `tests/installer_tests.ps1` against the new Setup. You can also run **Build and release** manually from the Actions tab, which signs the Windows files through [cattools.cc](https://cattools.cc) with the test certificate. Successful builds provide `Unishade-windows-x64` (the EXE and Setup), `Unishade-linux-x64` and `Unishade-macOS` artifacts.

To publish a release, run **Build and release** manually on `main` with **Publish a release** checked. It releases the version in `project(Unishade VERSION X.Y.Z)` in `CMakeLists.txt` as `vX.Y.Z`, and refuses a version that's already released. The Windows files are signed with the release certificate, and each signing request waits for an approver in cattools.cc. After the builds and tests pass, it creates the tag and a cattools.cc release with Setup, the EXE, the macOS and Linux archives, `LICENSE` and `SHA256SUMS.txt`, which lists the SHA-256 of each file, and then rebuilds the website. Pushes and pull requests do not publish releases.

## Website

[cattools.cc](https://cattools.cc) is built from `website/` with Astro. The docs are the Markdown files in `website/src/content/docs/`.

```powershell
cd website
pnpm install
pnpm dev
```

The download buttons link to the newest release's files, which the build looks up on cattools.cc. If cattools.cc can't be reached, the build still succeeds and the buttons link to the Releases page. Set `GITHUB_TOKEN` to avoid cattools.cc's limit of 60 requests an hour, and `GITHUB_REPOSITORY` to build against another repository; links to the repository follow it too. Setting `GITHUB_API_URL` to an address that doesn't answer shows the site as it builds offline.

In the Markdown pages, link to cattools.cc and the other addresses in `website/src/links.ts` by name, as `[cattools.cc](links:cattools.cc)`. A name that isn't in `links.ts` fails the build.

cattools.cc builds and deploys the site from `main`, with these build settings:

| Setting | Value |
| --- | --- |
| Root directory | `website` |
| Build command | `pnpm build` |
| Build output directory | `dist` |
| Environment variables | `GITHUB_TOKEN`: a fine-grained token with read access to public repositories, since cattools.cc's builds share cattools.cc's 60-requests-an-hour limit with everyone else on cattools.cc. `ASTRO_TELEMETRY_DISABLED=1`. |

pnpm and Node come from `packageManager` in `website/package.json` and `website/.node-version`, the same versions CI uses. cattools.cc sends the security headers in `website/public/_headers`. Keep Rocket Loader and automatic Web Analytics off: the pages' Content-Security-Policy only allows their own scripts.

A release doesn't change `main`, so cattools.cc would keep linking the old files. Create a deploy hook for `main` in cattools.cc and save its address as the `CLOUDFLARE_DEPLOY_HOOK` secret: the release job calls it after publishing, and the Website workflow calls it when a release is published, edited or deleted by hand, or when you run it from the Actions tab. Pull requests that change the site are built by that workflow too, so a broken build shows up before it's merged.

## Code changes

Game discovery lives in `src/game_integration.cpp`. `FindGameTarget()` uses a manual window selection or the executable list saved in `games.ini`, with Roblox enabled by default. Custom games match their full executable path; Roblox matches its filename because its install folder changes with updates. Discovery queries executable metadata without reading game memory or injecting code.

- The host is C++20.
- Builds use `/W4`. Fix warnings rather than suppressing them.
- Hotkey parsing has tests in `tests/hotkey_tests.cpp`. Add a case when you change it.
- Installer changes must pass `tests/installer_tests.ps1`. It downloads cattools.cc and all effect packages, so it takes a few minutes. CI runs it too, with the presets from your commit.
- Keep pull requests focused. Separate unrelated fixes into their own pull requests.
- Do not bump the version. Releases are cut from tags by the maintainer.

## Pull requests

CI builds the Windows, Linux and macOS versions, runs the unit tests and the installer tests, renders the presets on Linux, and builds the website when it changes. Describe what changed and how you tested it. For anything that affects what the user sees, include a screenshot.

## License

Contributions are licensed under the MIT license in `LICENSE`.
