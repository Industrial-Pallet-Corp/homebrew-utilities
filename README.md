# Industrial Pallet Corp utilities tap

A [Homebrew](https://brew.sh) tap that distributes Industrial Pallet Corp's internal
command-line utilities for **macOS and Linux**. Some utilities target a single
platform; most are cross-platform.

## Install

```bash
brew tap industrial-pallet-corp/utilities
brew install <utility>
```

Tapping once makes every formula in this repository installable by name. The tap
shorthand `industrial-pallet-corp/utilities` expands to this repository
(`Industrial-Pallet-Corp/homebrew-utilities`).

## Available utilities

| Utility | Description | macOS | Linux |
| ------- | ----------- | :---: | :---: |
| [`exacqman`](Formula/exacqman.rb) | Extract, timelapse, and compress footage from ExacqVision servers; ships a CLI (`exacqman`) and a local web UI (`exacqman-web`). | Yes | Yes |

### exacqman

```bash
brew install industrial-pallet-corp/utilities/exacqman

exacqman init                 # seed config + credentials into $(brew --prefix)/etc/exacqman
exacqman --help               # CLI
exacqman extract dock-10 5/30 9am 9:05am --server ch

exacqman-web start            # run the local web UI in the foreground (http://localhost:8887)
brew services start exacqman  # or run the web UI as a managed background service
```

Config and credentials live in `$(brew --prefix)/etc/exacqman` (seed them with
`exacqman init`; preserved across upgrades). The web service is opt-in: `brew
install` never starts a server.

Upstream project: <https://github.com/Industrial-Pallet-Corp/exacqman>

> The interactive crop selector (`exacqman crop` / `-c true`) opens an OpenCV
> GUI window and therefore requires a display.

#### ExacqMan dependency strategy

ExacqMan is a Python app whose heavy, native dependencies are provided by
**Homebrew formulae** rather than installed as pip wheels/sdists. The formula
`depends_on` `numpy`, `opencv`, `pillow`, `pydantic` (which brings in the Rust
`pydantic-core`), and `ffmpeg`; Homebrew builds the virtualenv with
`--system-site-packages`, so the package imports them from Homebrew. Only the
pure-Python dependencies are pinned as source `resource` blocks in the formula.

Why: Homebrew's build sandbox has no network access, and the offending packages
cannot be source-built offline - `opencv-python` ships no usable source
distribution at all, and `numpy` / `pydantic-core` need build toolchains (meson,
Rust) that aren't available inside the sandbox. Delegating them to Homebrew keeps
the formula source-built and portable across every platform Homebrew supports.

Trade-offs (revisit if this becomes painful):

- First install is heavy/slow: Homebrew `opencv` pulls a large native tree (vtk,
  tesseract, openvino, ceres-solver, ffmpeg, ...). We accept this for
  maintainability.
- The formula is pinned to `python@3.14` so the venv matches Homebrew `opencv`'s
  Python bindings; bumping Python means bumping it here in lockstep with `opencv`.
- `imageio-ffmpeg` is installed from its sdist (no bundled binary), so the formula
  points it at the Homebrew `ffmpeg` via the `IMAGEIO_FFMPEG_EXE` environment
  variable in the generated command wrappers.

The alternative - pinning per-platform binary wheels for the heavy deps - would
make installs small and fast but verbose to maintain (wheel URLs + checksums per
OS/arch on every bump) and limited to enumerated architectures.

## Updating

```bash
brew update
brew upgrade <utility>
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to add a new utility, the required
formula conventions, and how CI validates formula style and structure.

## License

[MIT](LICENSE)
