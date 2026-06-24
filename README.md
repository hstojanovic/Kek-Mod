# Kek-Mod

Kek-Mod is a modmod for *Sid Meier's Civilization IV: Beyond the Sword* (3.19),
built on [K-Mod](https://forums.civfanatics.com/resources/k-mod.13282/) v1.46.

It stays in the spirit of K-Mod — sharper AI, cleaner gameplay, and a long list of
bugfixes — with extra emphasis on **multiplayer**: faster multiplayer login (with
the right host setup), support for current NatNeg servers, and PitBoss play through
PB Mod.

- **Download:** https://forums.civfanatics.com/resources/kek-mod.24808/
- **Discussion:** https://forums.civfanatics.com/threads/modmod-kek-mod.563377/
- **Changelog:** [CHANGELOG.md](CHANGELOG.md)

## Features

- **Multiplayer focus:** faster MP login, current NatNeg server support, and an
  included launcher (`Kek-Mod.bat`) that turns them on.
- **PitBoss** support via PB Mod, plus an in-game mod updater.
- **48 civilizations** supported.
- Reworked **permanent alliances**, **barbarians**, drafting, and UN/AP resolutions.
- Improved **starting-location** selection, extra "Flat" and "Warm" climates, and a
  "Gigantic" map size.
- The **BUG** in-game interface and options, with Kek-Mod defaults.

See [CHANGELOG.md](CHANGELOG.md) for the full history.

## Installing and playing

- Requires **Beyond the Sword 3.19** (Windows).
- Unzip into `…/Beyond the Sword/Mods/Kek-Mod`, or update in place from inside the
  game via **Advanced → Options → Update Mod** (available since 0.3.0, when the
  update server is online).
- Launch via **Advanced → Load a Mod → Kek-Mod**, or run **`Kek-Mod.bat`** (which
  also enables the multiplayer login and NatNeg improvements).
- Multiplayer executables in the Beyond the Sword folder are **not** updated by the
  in-game updater.

### PitBoss

Normal PitBoss does not work with this version — use the PB Mod version instead:
[forum thread](https://forums.civfanatics.com/threads/mod-for-pitboss-games.533346/)
· [PBStats on GitHub](https://github.com/YggdrasiI/PBStats). Their tools are worth a
look for any PitBoss host.

## Building

Only the C++ game-core DLL needs building; content (XML, Python, art) does not. See
[BUILDING.md](BUILDING.md).

## Credits

Kek-Mod builds on [K-Mod](https://forums.civfanatics.com/resources/k-mod.13282/) by
karadoc, which itself builds on Better BTS AI and the BUG mod. It also borrows fixes
and ideas from other Civ4 projects (PB Mod, AdvCiv, and more); specific borrowings
are credited in code comments and in the [changelog](CHANGELOG.md). Bundled
third-party documentation lives under [`credits/`](credits/).

## Related

- **FF+Kek-Mod** — a Final Frontier Plus variant built on Kek-Mod:
  [forum thread](https://forums.civfanatics.com/threads/modmod-ff-kek-mod.614192/).
