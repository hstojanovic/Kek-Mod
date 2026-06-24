# Changelog

All notable changes to this project will be documented in this file.

The format is inspired by [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and [Common Changelog](https://common-changelog.org/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Kek-Mod is based on K-Mod; K-Mod's own changelog is bundled separately under
`credits/K-Mod/changelog.txt`. Entries here cover Kek-Mod's changes, not the
details of each K-Mod rebase.

## [Unreleased]

### Changed

- Rework the build system into a `uv`-driven SCons setup with VS Code integration and clang-based tooling.

### Added

- Add `CyGame.claimSlot` / `CyGame.retireSlot` for starting-location auctions, and start the PitBoss Civ4Shell console when a saved game is loaded, not only on game start.

## [0.3.1] - 2024-10-01

_Based on K-Mod v1.46._

### Fixed

- Fix a crash on exit by bundling a newer multiplayer executable (`BTS_Wrapper.exe`).

## [0.3.0] - 2024-09-23

_Based on K-Mod v1.46. **Breaking:** save games from earlier versions are not compatible._

### Changed

- **Breaking:** Rename the mod folder to `Kek-Mod`.
- Switch the project to Semantic Versioning.
- Replace the Makefile with an SConstruct build.
- Change most BUG mod defaults.
- Enable BUG mod in PitBoss.
- Stop autoplaying Advanced Start on exit.
- Increase the Disorganized strength penalty for barbarian ships to -20%.
- Add game and multiplayer options to the OOS checksum.

### Added

- Bundle all K-Mod files and Blue Marble.
- Add multiplayer executables and a batch file that launches the mod with them.
- Add a `pyconsole` backend (by YggdrasiI and Zulan).
- Show a warning on the first turn on which barbarians can spawn.
- Expose easier access to a player's yields for PitBoss.

### Fixed

- Fix Python `isSynchLogging`.

## [v0.26] - 2022-02-05

_Based on K-Mod v1.46._

### Changed

- Improve starting-location selection to better avoid bad terrain and the edges of bad terrain.

### Added

- Slow down PitBoss games when no one is connected, to conserve cloud resources (configurable via `GlobalDefinesAlt.xml`).
- Add a new "Flat" climate with fewer peaks, available to all mapscripts using the default climate system (by Fran).

### Fixed

- Fix an integer overflow in the value-of-ending-war calculation.
- Fix a Great Mediator event bug that could cause master/vassal war-status inconsistency.

## [v0.25b] - 2020-10-06

_Based on K-Mod v1.46._

### Changed

- Make Internet victory count players (civilizations) rather than teams, and raise the Internet permanent-alliance threshold by 1 per additional team member.

### Fixed

- Fix an OOS bug in the Partisans event related to the new draft method.
- Fix OOSLogger.

## [v0.25a] - 2019-07-18

_Based on K-Mod v1.46._

### Fixed

- Fix a crash when pillaging is intercepted by a sea patrol.
- Fix a C++ exception in Python when founding the first city of the game.

## [v0.25] - 2019-07-16

_Based on K-Mod v1.46._

### Changed

- Require debug mode for cheat actions and tooltips.
- Use the player text color instead of the primary color in the wonder list.
- Stop granting free food when a city starts with more than 1 population in Advanced Start.
- Strengthen food requirements for starting locations.
- Adjust era factors and add a count of non-early religions, primarily for mod compatibility and consistency.
- Adjust domination thresholds for permanent alliances.
- Make minor scoreboard changes.

### Removed

- Remove the buggy mod-updater autostart, replaced by an "Update Mod" button in the in-game options.

### Fixed

- Integrate OOSLogger fixes from AdvCiv (by f1rpo).
- Fix a Crusade event bug caused by the new draft method.
- Fix the missing icon for Build Espionage.
- Fix the attitude cache not updating reliably after a war declaration (fix by f1rpo).
- Various minor fixes.

## [v0.24] - 2019-04-15

_Based on K-Mod v1.46._

### Changed

- Revert PitBoss files from PB Mod v8 to PB Mod v7 due to a buggy interface.
- Prevent free units from tribal villages from moving on their first turn.
- Change details of how starting locations are picked.

### Added

- Add a new "Warm" climate with less ice and tundra, available to all mapscripts using the default climate system (by AjmoCiv).
- Add an option to adjust the water percentage in the "Not too Big or Small" mapscript.
- Add a "Gigantic" map size, larger than Huge.

### Fixed

- Fix a crash related to the new drafting method.
- Various minor fixes.

## [v0.23] - 2019-03-18

_Based on K-Mod v1.46._

### Changed

- Always show the Dawn of Man screen (idea from History Rewritten by Xyth).
- Queue war declarations that trigger further war declarations instead of nesting them, so defensive pacts and related effects fire in the correct order.
- Select the draft unit type by strength and combat abilities instead of by conscription value.
- Give a liberated colony the best available defenders instead of free draft-type units.
- Improve the maximum-distance calculation so maintenance is more reasonable on toroidal maps (idea from Tides of War by SevenSpirits).
- Require barbarians to have the reveal tech for a resource before building units that need it.
- Mark a city that will be autorazed on conquest with a `*` next to its population.
- Prevent selling techs acquired via the Internet under the "no tech brokering" option.
- Stop a unit gifted to a team member from becoming immobile next turn.
- Permanent alliances:
  - Fix the espionage-visibility bug.
  - Preserve "no tech brokering" status correctly.
  - Adjust and fix counter and item sharing during team merges.
  - Raise victory requirements (cities and parts for cultural and space victories) for teams with more than one member.
  - Scale team espionage costs similarly to research (formula idea by Fran).
- Advanced Start:
  - Apply the advanced-start points handicap only to the AI, adjusting modifiers so humans always get the number shown in game setup.
  - Stop granting starting gold and free handicap techs.
  - Give barbarians starting research progress based on what other players receive.
- Scoreboard:
  - Allow showing unmet dead civs.
  - Add leader and civ icons.

### Added

- Merge components of PB Mod v8 (by Ramkhamhaeng and Zulan): an in-game Mod Updater, object files needed to compile new PB Mod features, and the ability to extend counterespionage on its last turn.
- Integrate OOSLogger (idea from Fall from Heaven 2 by Kael).
- Add a Build Espionage process (idea from Convert Production to Espionage by deanej).
- Add a choice of map wraps to the "Not too Big or Small" mapscript.
- Integrate fixes from AdvCiv (by f1rpo): a fix for a bug caused by producing multiple limited units, a lighter player color over water on the minimap, and other bugfixes.

### Fixed

- Address Nightingale's warning about DllExports and virtual functions ([details](https://github.com/We-the-People-civ4col-mod/Mod/issues/184)).
- Various smaller bugfixes.

## [v0.22] - 2018-09-09

_Based on K-Mod v1.46._

### Changed

- Remove automation from AI units when the AI takes over a human player (previously the automation was ignored, so no functional change).

### Removed

- Remove the C2C "minimize AI turn time" option.

### Fixed

- Fix a bug in culture removal after city destruction.

## [v0.21] - 2018-08-02

_Based on K-Mod v1.46._

### Fixed

- Fix an error in the reimplementation of random leader/civ selection.
- Fix a crash when nuking a non-combat unit.

## [v0.20] - 2018-06-02

_Based on K-Mod v1.46. **Breaking:** save games from earlier versions are not compatible._

### Changed

- **Breaking:** Ship a single DLL supporting 48 civilizations (previously a separate 18-civ DLL).
- Rebase onto K-Mod v1.46.
- Switch to PB Mod v7 PitBoss (requires the v7 Python PBStats files).
- Stop colonies from reusing player slots (this caused bugs); colonies no longer inherit espionage points from the parent civ (but still inherit `EspionagePointsEver`).
- Prevent moving the capital while a spaceship is underway.
- Improve AI evaluation of SDI and Bomb Shelters (the AI previously thought Bomb Shelters were worthless).
- Stop the first AI city in a new area from always being coastal.
- Move missile-carrier missile-building out of the assault case (mirroring K-Mod's aircraft-carrier handling).
- Apply the Imperialistic settler cost reduction in Advanced Start when purchasing cities.
- Stop granting extra free food when purchasing population points in Advanced Start.
- Require cities on different landmasses to also be two tiles apart.
- Calculate the average handicap by rounding instead of rounding down.
- Make gifted units unable to move for 2 turns.
- Adjust project AI to earlier changes.
- Game options:
  - Always set hidden game options to their default value.
  - Add an "Unlimited Permanent Alliances" option that removes the two-member team-size limit.
  - Add "Advanced Settlers" options where the settler era tracks game progress.
- City ownership and razing:
  - Destroy national and team wonders when a city changes owner.
  - Keep a razed or transferred city's spot on everyone's map and update culture coverage.
  - Erase most plot culture when razing a city.
  - Transfer all owner city culture and most generated plot culture when trading a city.
  - Leave the owner's culture intact on city transfers from diplomatic votes.
  - Prevent cities from culture-flipping from master to vassal.
- UN/AP resolutions:
  - Allow war resolutions against voting members.
  - Cancel defensive pacts before a war resolution takes effect.
  - Prevent vassals from defying resolutions; allow players to defy resolutions assigning them a city.
  - Divide religious population for AP votes by the number of religions in the city.
  - Let the AI choose and vote to repeal resolutions.
  - Apply the defiance penalty to the whole team when one member defies a passing resolution.
- Barbarians:
  - Turn a destroyed civilization's culture barbarian, shown on the minimap and in the globe view.
  - Set barbarian unit support costs to 0.
  - Apply the same handicap modifiers to barbarians as to other AI players.
  - Spawn barbarians only on food-producing tiles (and at sea only adjacent to coastal food tiles), with a higher chance near more food.
  - Give Raging Barbarians cities a starting worker.
  - Enable barbarian spawns in all eras and remove the BBAI sea-barbarian spawn limit.

### Added

- Add BUG mod Scoreboard options to show team score and team ID number.
- Add the game era to the era display in BUG mod.
- Add verification of state religion.
- Support production overflow for mods with multiple buildings of the same type in a city (from Final Frontier Plus by TC01 and T-hawk).

### Fixed

- Fix the Bomb Shelter effect for non-combat units (it used to reduce nuke death chance to 2-3%; now ~39.5%).
- Fix the number of missiles needed for missile carriers.
- Fix the "Unrestricted Leaders" game option, which the EXE hardcodes in the 8th position, by reimplementing random leader/civ assignment (idea from Fall from Heaven 2 by Kael).
- Fix detection of when the AI should do Advanced Start in simultaneous-turns games.
- Fix a bug in unit-gifting restrictions.
- Fix the color of unknown players in the project list.
- Minor spelling and other adjustments.

## [v0.10] - 2017-04-07

_Based on K-Mod v1.45. **Breaking:** save games from earlier versions are not compatible._

### Changed

- **Breaking:** Make the 48-civ DLL the default (the 18-civ DLL is still included).
- Rebase onto K-Mod v1.45.
- Change starting espionage weight from 1 to 0.
- Sort vote sources alphabetically in the victory screen.
- Stop team members from blocking each other from building wonders and projects.
- Sort buildings in Sevopedia ignoring a leading "the".
- Make diplomatic votes, tribute demands, and accepting help create one-way peace treaties.
- Make nukes ignore neutral players and not be blocked by them.
- Show the correct (black) color for unknown players regardless of the maximum number of players.
- Force human vassals to vote for their master unless they are also a candidate (matching the AI).
- Prevent unused nuclear plants from triggering a meltdown (e.g. a nuclear plant in a city that also has a hydro plant).
- Rework the fallout feature so the AI understands and scrubs it efficiently: fallout is treated like jungle/forest, so improvements can be built on it and doing so removes it.
- Barbarians:
  - Let them build all normal land and sea military units (only those with a supported unit AI type).
  - Block additional special units (spies, etc.) for consistency.
  - Let them build and upgrade units without resources, needing only the revealing and enabling technologies.
  - Let them upgrade units outside their territory (with spawn-like limitations) and hurry production in their cities.
  - Stop them valuing commerce when building improvements, working plots, and picking specialists, and stop them valuing great-person points.
  - Adjust their available buildings (include culture buildings, exclude gold/commerce buildings).
  - Remove the Disorganized promotion from barbarian ships with hidden nationality.

### Added

- Integrate additional alberts2 bugfixes.
- Integrate Zholef's German-language text fixes.
- Add a high-resolution waiting cursor (globe) from Leoreth's Dawn of Civilization.

### Fixed

- Fix permanent-alliance bugs with the UN/AP leader team.
- Fix a bug related to the minimum yield required for building improvements and removable yield-lowering features.
- Make the Glance screen with restricted leaders always show all rows.
- Fix the Advanced Start bug enabling infinite gold for the Expansive trait.
- Fix the founding-year display for AI cities built in Advanced Start with simultaneous turns.
- Restrict and fix unit gifting:
  - Fix a bug in the AI evaluation of a received unit.
  - Prevent player A from gifting combat units to player B when B is at war with C and A has a peace treaty with C.
  - Require the receiving player to have the prerequisite techs for both the unit and its required resources.
- Fix various minor bugs introduced previously.

## [v0.04] - 2016-05-26

_Based on K-Mod v1.44b._

### Changed

- Rebase onto the latest post-v1.44b K-Mod changes.
- Change defensive-pact behavior to BBAI option 1: defensive pacts no longer cancel when attacked and can be signed during war.
- Block players who have vassals from becoming peace vassals (to stop them releasing vassals needlessly).
- Improve the foreign advisor for high civ counts: your own leaderhead in the Relations tab is now an on/off toggle, so only selected leaderheads show in the Relations and Glance tabs.
- Disable gifting foreign-religion missionaries to theocracies (and generally where a non-state religion cannot spread) to block a religion-spreading exploit, and disable gifting transports unless all cargo can be gifted.

### Added

- Integrate most of alberts2's changes: bugfixes, the C2C "minimize AI turn time" option, and WorldBuilder using Platy Builder 4.17b.
- Apply Ramkhamhaeng's patch for missing and duplicate language tags.

## [v0.03] - 2016-04-14

_Based on K-Mod v1.44b._

### Added

- Add a 48-player DLL alongside the standard 18-player one.

## [v0.02] - 2016-03-20

_Based on K-Mod v1.44b._

### Changed

- Count team tech score only once instead of once per player.
- Always preserve the circumnavigation bonus when forming a permanent alliance (previously only when the holder had the smaller team number).

### Fixed

- Fix a major bug causing unremovable espionage city visibility after forming a permanent alliance (for all cities the higher-team-number player could see at that moment).
- Tweak the trade-screen unpause-bugfix text and button location.

## [v0.01] - 2016-02-28

_Based on K-Mod v1.44b._

### Changed

- Bump enemy units out of enemy borders when a city flips instead of deleting them.

### Added

- Merge PB Mod v5, enabling its web-interface features, the pause/trade-screen bugfix, and other minor fixes (credit to the PB Mod creators).

[Unreleased]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_0.3.1...HEAD
[0.3.1]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_0.3.0...DLP_0.3.1
[0.3.0]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.26...DLP_0.3.0
[v0.26]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.25b...DLP_v0.26
[v0.25b]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.25a...DLP_v0.25b
[v0.25a]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.25...DLP_v0.25a
[v0.25]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.24...DLP_v0.25
[v0.24]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.23...DLP_v0.24
[v0.23]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.22...DLP_v0.23
[v0.22]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.21...DLP_v0.22
[v0.21]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.20...DLP_v0.21
[v0.20]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.10...DLP_v0.20
[v0.10]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.04...DLP_v0.10
[v0.04]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.03...DLP_v0.04
[v0.03]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.02...DLP_v0.03
[v0.02]: https://github.com/hstojanovic/Kek-Mod/compare/DLP_v0.01...DLP_v0.02
[v0.01]: https://github.com/hstojanovic/Kek-Mod/compare/v1.44b...DLP_v0.01
