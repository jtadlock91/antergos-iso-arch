# HANDOFF: Antergos NeXT — Arch Edition

Authoritative spec for this project. Read this in full before writing any code or creating the repo.

## Context

John (jtadlock91) already maintains **Antergos NeXT**, a hobby Artix Linux fork
(dinit + KDE Plasma) at `github.com/jtadlock91/antergos-iso`. This handoff
defines a **new, separate edition** of the same distro identity, built on Arch
proper (systemd) instead of Artix (dinit).

## Do Not Touch

`jtadlock91/antergos-iso` (the Artix edition) is untouched by this work. No
merging into a shared monorepo, no refactoring its archiso profile to be
multi-base. This is a sibling repo, not a branch.

## Goal

A publishable, Arch-based (systemd) Linux distro sharing the "Antergos NeXT"
brand and Calamares look-and-feel as the Artix edition. KDE Plasma 6 on
Wayland is the primary desktop.

## New Repo

- GitHub: `jtadlock91/antergos-iso-arch` (to be created)
- Suggested local path: `~/projects/antergos-iso-arch`
- **First commit must include `CLAUDE.md`** — standing rule from prior
  projects (lost `distro-tool`'s original source once in a fresh install by
  skipping this)

## Artifact Naming

Umbrella brand stays **Antergos NeXT**. Edition is a suffix, not a rename.

- Existing: `antergos-nxt-artix-x86_64.iso`
- New: `antergos-nxt-arch-x86_64.iso`

## Build Toolchain

- **archiso**, based on the official `releng` profile
- **Calamares** — port the existing `antergos-iso` Calamares branding folder
  wholesale (logo, theme, slideshow, palette). It's init-agnostic; this is a
  straight copy, not a rework.
- **Do not port**: any dinit-specific hooks/services, or the
  `~/.config/artools/pacman.conf.d/iso-x86_64.conf` symlink setup — that's
  Artix/artools-only infrastructure with no Arch equivalent.

## Build Environment

Do not reuse the existing Artix ISO-build workstation. Stand up a fresh
Proxmox CT on the homelab dedicated to this edition's builds (same role CT
103 plays for pcrepair services). Keeps `pacman.conf` state and package
caches from cross-contaminating between the two editions' build hosts.

## Repo / Package Tier Architecture

Three-tier `pacman.conf` priority, highest wins:

1. **Own repo** — distro identity packages, any patched/forked packages,
   kernel packaging choices, and a systemd-native successor to
   `distro-tool doctor` (see Open Questions — not required for v1)
2. **CachyOS repo** — auto-detect microarch at install time, don't assume
   v4 hardware the way the personal desktop script could:
   - `x86_64-v4` optimized tier if the CPU supports it
   - `x86_64-v3` fallback otherwise
   - generic CachyOS repo always included as baseline
   - Detection logic to reuse (already proven working on the desktop):
     ```bash
     /lib/ld-linux-x86-64.so.2 --help | grep -q "x86-64-v4 (supported)"
     ```
   - This needs to run during install (Calamares hook or first-boot script)
     and write the correct repo block into `/etc/pacman.conf`
3. **core / extra / multilib** — vanilla Arch fallback, lowest priority

## Signing

- Generate a **dedicated** GPG keypair for this repo — do not reuse any
  existing personal or other-project key
- Ship the public key as a keyring package in the base package set, same
  pattern as `archlinux-keyring` / `cachyos-keyring`, so pacman trusts the
  repo with no manual `pacman-key` step on first boot

## DE / Package Story

Reuse and adapt the existing scripts already proven on Arch/EndeavourOS/CachyOS:

- `kde-setup-universal.sh` — GPU auto-detection (AMD/Nvidia/Intel/hybrid),
  barebones Plasma, audio, networking, fonts, snapper
- `kde-debloat.sh` — protected-package list + safe-removal pattern
- `kde-setup-personal.sh` — CachyOS RC kernel + RDNA4/BORE tuning; this one
  is Zen4/5-gated and **not** relevant to a general public install, keep it
  out of the base image

Decide during scaffolding: bake the debloated package list directly into
`packages.x86_64` (preferred — smaller ISO, no surprise post-install removal
step) vs. ship the scripts as optional convenience tools.

No DE-package forking/self-hosting tradeoff needed here, unlike the open
question on the Artix edition — i3/Sway/Hyprland configs, `layan-theme`,
`tela-circle-icon-theme-git` all live in `extra`/AUR on Arch. There's no
`antergos-pkgs`-style dead-upstream problem to work around.

## Hosting

Repo + ISO releases on the homelab (new build CT above, or adjacent to CT
103), fronted by Cloudflare Tunnel — likely a new subdomain off
`tadlockfamily.net` (open question below). ISO releases can follow the same
GitHub Releases pattern `antergos-iso` already uses.

## Branding Caution

Public-facing, so avoid implying official Arch endorsement. Describe as
"Arch-based," the same convention EndeavourOS/CachyOS/Manjaro use — never
"Arch Linux" itself.

## Open Questions

Resolve these with John before or during scaffolding — don't decide silently:

1. Domain/hosting: subdomain off `tadlockfamily.net` vs. a standalone domain
2. Health-check tool: build a systemd-native successor to
   `distro-tool doctor` now, or defer to a later release
3. Package-list-only vs. post-install-script approach for the KDE debloat step
4. Multi-DE at launch (GNOME as a secondary session, mirroring the old
   `kde-setup-universal.sh` intent) or KDE-only for v1

## First Tasks

1. Create `jtadlock91/antergos-iso-arch`, commit `CLAUDE.md` first
2. Scaffold the archiso profile from the `releng` base
3. Port the Calamares branding folder from `antergos-iso` as-is
4. Stand up the dedicated build CT on the homelab
5. Draft `packages.x86_64` from the `kde-setup-universal.sh` +
   `kde-debloat.sh` keep-list
6. Implement the microarch-detection → `pacman.conf` tier-writing logic as
   an airootfs hook or Calamares module
