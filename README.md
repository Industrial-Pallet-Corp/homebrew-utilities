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
exacqman --help               # CLI usage

exacqman-web --help           # web server usage
exacqman-web start            # run the local web UI in the foreground (http://localhost:8887)
brew services start exacqman  # or run the web UI as a managed background service
```

Config and credentials live in `$(brew --prefix)/etc/exacqman` (seed them with
`exacqman init`; preserved across upgrades). The web service is opt-in: `brew
install` never starts a server.

Upstream project: <https://github.com/Industrial-Pallet-Corp/exacqman>

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
