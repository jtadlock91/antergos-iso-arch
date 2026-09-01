# antergos-iso-arch — Antergos NeXT (Arch Edition)

Arch Linux (systemd) edition of the "Antergos NeXT" distro brand. KDE Plasma
6 / Wayland primary desktop, built with **archiso** from the official
`releng` profile.

Full spec: see `HANDOFF.md` in this repo — read it before making
architectural decisions here. Key points below are the ones that most affect
day-to-day work.

## Sibling repo — do not touch

`jtadlock91/antergos-iso` is the **separate** Artix (dinit) edition of the
same brand, maintained locally at `~/projects/antergos-iso`. This repo does
not merge into it, depend on it, or refactor it. Treat it as read-only
reference material only:

- Its Calamares branding folder is ported here **wholesale, as-is** (logo,
  theme, slideshow, palette) — it's init-agnostic.
- Its dinit-specific hooks/services and its
  `~/.config/artools/pacman.conf.d/iso-x86_64.conf` symlink setup are
  Artix/artools-only and have **no Arch equivalent** — do not port them.
- Its build tooling is `artools`/`buildiso` (Artix-specific). This repo
  builds with plain `archiso`/`mkarchiso` instead — different toolchain,
  don't try to unify them.

## Build toolchain

- **archiso**, scaffolded from the official `releng` profile (not the
  artools/buildiso pattern used by the Artix edition).
- Build host: a dedicated Proxmox CT on the homelab (separate from the
  Artix edition's build host, to avoid `pacman.conf`/cache
  cross-contamination). Not yet stood up — see HANDOFF.md First Tasks.

## Repo / package tier architecture

Three-tier `pacman.conf`, highest priority first:

1. Own repo (distro identity packages, patched/forked packages, kernel
   choices, signed with a **dedicated** GPG key — never reuse another
   project's key)
2. CachyOS repo, microarch-detected at install time (`x86_64-v4` if
   supported, else `x86_64-v3`, generic CachyOS always included) — do not
   assume v4 hardware
3. `core`/`extra`/`multilib` — vanilla Arch fallback

## Branding

Public-facing distro. Describe as "Arch-based" (EndeavourOS/CachyOS/Manjaro
convention) — never imply official Arch Linux endorsement.

## Open questions

Several decisions (hosting domain, health-check tool scope, debloat
packaging approach, multi-DE at launch) are intentionally unresolved — see
HANDOFF.md "Open Questions". Resolve with John before deciding silently;
don't assume an answer to move a task forward.
