# Antergos NeXT — Arch Edition

Arch-based (systemd) live/install medium sharing the Antergos NeXT brand.
See `HANDOFF.md` for the full project spec and `CLAUDE.md` for repo-specific
notes.

This profile is scaffolded from the official archiso `releng` profile
(upstream commit `f900196af8f293ec7e4ef452b368b9db8012d79f`), with identity
fields in `profiledef.sh` rebranded. Package set, Calamares branding, and
the pacman.conf repo tiers are not yet customized — see HANDOFF.md's "First
Tasks" for what's still open.

## Build

Requires `archiso`:

```bash
sudo mkarchiso -v -w work/ -o out/ .
```

Builds are intended to run on the dedicated Proxmox CT described in
HANDOFF.md's "Build Environment" section, not on this workstation — that CT
keeps this edition's `pacman.conf` state and package caches isolated from
the Artix edition's build host.

`work/` and `out/` are gitignored build/output directories.
