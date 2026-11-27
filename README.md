# fetchbox-artifacts

Builds the desktop apps (the Windows installer and `.exe`, macOS disk images for Apple
Silicon and Intel, and the Linux `.deb`, AppImage and binary) for tags of
[fetchbox](https://github.com/MichaelYagi/fetchbox), tests them, and adds them to that
tag's release in the fetchbox repo. Public, so its GitHub Actions minutes are free.

- Runs automatically when the public repo's `notify-desktop.yml` sends a `public-tag`
  dispatch. That needs `DESKTOP_DISPATCH_TOKEN` set as a secret *there*: a fine-grained
  token with "Contents: Read and write" on this repo only.
- Run one by hand:
  `gh workflow run desktop-build.yml -R MichaelYagi/fetchbox-artifacts -f tag=v0.1.0`
- Test-build a branch without publishing anything:
  `gh workflow run desktop-build.yml -R MichaelYagi/fetchbox-artifacts -f tag=main -f publish=false`
  (the builds are kept as workflow artifacts for 30 days).
- `FETCHBOX_RELEASE_TOKEN` is set here, to publish to fetchbox's release: GitHub keeps
  it hidden, in logs too, and the workflow can only be run by the repo's owner.

## What a release contains

Four files per tag, all unsigned:

| Asset | Contents |
| --- | --- |
| `Fetchbox-Windows-10-11-vX.Y.Z.zip` | `Fetchbox.exe` (tray + server) |
| `Fetchbox-macOS-Silicon-vX.Y.Z.zip` | `Fetchbox.app` for Apple Silicon (menu bar + server) |
| `Fetchbox-macOS-Intel-vX.Y.Z.zip` | `Fetchbox.app` for Intel Macs |
| `Fetchbox-Linux-x86_64-vX.Y.Z.tar.gz` | `fetchbox`, the command-line server |

Each one also holds `README.txt` (how to start it and get past Gatekeeper / SmartScreen),
`LICENSE.txt` (MIT) and `THIRD-PARTY-NOTICES.txt` (generated from what is in that
build).

## How a release happens

1. In the public repo, set the version in `pyproject.toml` and `web/package.json`,
   commit, then `git tag vX.Y.Z && git push origin vX.Y.Z`.
2. The public repo's `Release` workflow runs the tests and checks the version. It then
   publishes the wheel, the sdist and a source zip. `notify-desktop.yml` dispatches here.
3. `desktop-build.yml` validates the tag and runs the tests. It then builds the four
   platforms in parallel and smoke-tests each one: started with a scratch data folder,
   it must answer, report the version and serve its web interface. Finally it publishes
   the Release here.

Building by hand on your own machine: see `packaging/README.md` in the public repo.

## Gotchas

- Each platform builds on its own runner (no cross-compiling). `macos-15-intel` is
  GitHub's last Intel image (until about August 2027).
- A runtime dependency whose package ships no license text needs
  `packaging/license-fallbacks/<name>.txt` in the public repo, or the notices step
  fails. This is intended: nothing ships with a notice missing.
- The source stays available through the public repo, which the notices point to. That
  also covers pystray's LGPL (users can rebuild with a different pystray).
