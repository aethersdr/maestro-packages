# Maestro packages

The pacman repository for the AetherSDR Maestro console's system software: the
front-panel daemon (`maestro-panel`) and the kiosk session (`maestro-kiosk`). The
packages are built and signed by the AetherConsole project's CI and published as the
assets of this repository's [`repo`](../../releases/tag/repo) release. No source code
is kept here.

A console installed from the AetherSDR Maestro image already uses this repository and
updates from it with the rest of the system (the settings panel's **Software Update**).

## Using it by hand

Trust the signing key, then add the repository to `/etc/pacman.conf`:

```sh
curl -LO https://raw.githubusercontent.com/aethersdr/maestro-packages/main/maestro-packages.asc
sudo pacman-key --add maestro-packages.asc
sudo pacman-key --lsign-key 0C67AF7364C6AE0D8A16216ACFC6A5050060030D
```

```ini
[maestro]
SigLevel = Required DatabaseRequired
Server = https://github.com/aethersdr/maestro-packages/releases/download/repo
```

Signing key: `AetherSDR Maestro Packages <packages@aethersdr.com>`, ed25519,
fingerprint `0C67 AF73 64C6 AE0D 8A16  216A CFC6 A505 0060 030D`.

## License

The packages may be installed and run only on a FlexRadio Maestro that the AetherSDR
project supports, and may not be sold or redistributed. See [`LICENSE`](LICENSE).
