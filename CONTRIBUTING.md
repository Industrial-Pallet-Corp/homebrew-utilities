# Contributing to the utilities tap

This repository is a Homebrew **tap** (`industrial-pallet-corp/utilities`). Each
utility is a single Ruby formula in [`Formula/`](Formula/). This document
describes how to add or update one and the conventions every formula must follow.

## Repository layout

```
Formula/                 # one <name>.rb per utility (auto-discovered by brew)
.github/workflows/       # CI: lint/audit + cross-platform install & test
README.md                # tap intro + utility index
```

## Prerequisites

- Homebrew installed (`brew --version`).
- This tap available locally for development:

  ```bash
  brew tap industrial-pallet-corp/utilities
  # then edit files in:
  brew --repository industrial-pallet-corp/utilities
  ```

  (Or develop in a clone and point brew at it with `brew tap-new` / a symlink.)

## Adding a new utility

1. Create `Formula/<name>.rb`. The file name is the install name; the class name
   is the file name in UpperCamelCase (`my-tool.rb` -> `class MyTool < Formula`).
2. Point `url` at an **immutable, tagged release tarball** plus its `sha256`.
   Never reference a branch or moving ref.

   ```bash
   # compute the checksum for the url you are pinning
   curl -fsSL <release-tarball-url> | shasum -a 256
   ```

3. Fill in the required stanzas (below).
4. Validate locally on at least one platform, ideally both:

   ```bash
   brew install --build-from-source industrial-pallet-corp/utilities/<name>
   brew test industrial-pallet-corp/utilities/<name>
   brew audit --tap=industrial-pallet-corp/utilities <name>
   brew style industrial-pallet-corp/utilities
   ```

5. Open a PR. CI runs `brew style`/`brew audit` and then installs + tests the
   changed formula on macOS (Apple Silicon and Intel) and Linux.

## Required formula conventions

Every formula should include:

- `desc` - one line, no trailing period, does not start with the formula name or
  "A"/"An"/"The".
- `homepage`.
- `url` + `sha256` - tagged release tarball (see above).
- `license` - SPDX identifier (e.g. `"MIT"`).
- `livecheck` - so `brew livecheck` can detect new upstream releases. For GitHub
  tags this is typically:

```ruby
livecheck do
  url :stable
  strategy :github_latest
end
```

- `test do` - a real, non-trivial check that the installed artifact works
  (e.g. asserting `--help`/`--version` output). Avoid tests that need network,
  credentials, or a display.
- `bottle do` - leave a placeholder until bottles are built by CI (see
  [Bottles](#bottles)).

## Platform support

This tap serves both macOS and Linux. Gate platform-specific behavior explicitly:

- Single-platform utility:

      depends_on :macos   # or: depends_on :linux

- Per-platform dependencies or steps:

      on_macos do
        depends_on "some-macos-dep"
      end
      on_linux do
        depends_on "some-linux-dep"
      end

Document each utility's platform support in the README index table.

## Python utilities

For Python apps, prefer a self-contained virtualenv install (inside the formula
class):

    include Language::Python::Virtualenv

    depends_on "python@3.13"

    def install
      virtualenv_install_with_resources
    end

- The build sandbox has **no network access**, so every Python dependency must be
  declared as a `resource` block. Generate/refresh these with:

  ```bash
  brew update-python-resources Formula/<name>.rb
  ```

- This requires the upstream project to be `pip install`-able (a `pyproject.toml`
  with console entry points and PyPI-resolvable dependencies). If the upstream
  repo is not yet packageable, prepare it there first (a `src/`-layout PEP 621
  project with declared `[project.scripts]`), then point the formula at a tagged
  release. `exacqman` is a worked example of this pattern.

## Bottles

Bottles (prebuilt binaries) are optional. When enabled, CI builds them per
OS/arch and uploads them to the release; the resulting `bottle do` block is then
committed back into the formula. Until then, formulae install from source.

## Style

Formulae must pass `brew style`. Run `brew style --fix industrial-pallet-corp/utilities`
to auto-correct most issues before committing.
