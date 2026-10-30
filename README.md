# Extra Trinket Slots for Abiotic Factor

[简体中文](README.zh-CN.md) · [Blueprint architecture](docs/ARCHITECTURE.md)

Editable source assets for an official-Modkit mod that adds four trinket slots to the native inventory UI. Maintained by [AlanFangchang](https://github.com/AlanFangchang).

## Implementation at a glance

The mod extends the player's native equipment inventory, stores extra items at indices 13–16, and inserts four custom slot widgets into the native inventory canvas. A state component reads the equipped hacking device and controls slot access. The server reconciles the items' buffs and temperature resistance.

Start with [the implementation guide](docs/ARCHITECTURE.md) for the complete flow, including the gear-event filter that prevents duplicate pickups.

## Features

| Equipped hacking device | Extra active slots | Total active trinket slots |
| --- | ---: | ---: |
| None / below T3 | 0 | 2 |
| T3 | 1 | 3 |
| T4 | 2 | 4 |
| T5 | 3 | 5 |
| Gatekey | 4 | 6 |

All four extra slots remain visible while the inventory is open. Locked slots retain their items and allow removal, but reject insertion and provide no effects. Unlocked slots use native-looking icons, tooltips and placement highlighting.

The mod handles equipped buffs and additive `ColdResist` / `HeatResist`. Identical resistance-granting trinkets stack. Real items reside in the native equipment inventory, using its existing persistence mechanism.

## Source layout

`Mods/TrinketSlotProbes2QpQ/` is the complete current content plugin. The `.uasset` files contain the editable Blueprint graphs and their defaults; they are the source, not cooked game files. This is not a standalone Unreal `.uproject` or a C++ project.

Use the official Abiotic Factor Modkit to open/import the local mod source. Preserve the `TrinketSlotProbes2QpQ` folder/plugin name and internal `/TrinketSlotProbes2QpQ/` asset paths: existing references depend on them. The historical “Probe” names remain for reference compatibility; no experimental runtime assets are included.

Start reading at `Class_TrinketSlotsBootstrap`, then `BP_TrinketSlots_Bootstrap`, `BP_TrinketSlots_State`, `BP_TrinketSlots_UI` and `WBP_TrinketSlot_Runtime`. See [the implementation guide](docs/ARCHITECTURE.md) for their roles.

The official Modkit supplies the referenced game and engine classes, widgets, textures and string tables. These dependencies are not bundled. No Python bridge, external MCP server, UE4SS or development script is needed to run the mod.

## Publication

This source snapshot includes the Cold/Heat Resistance fix completed on **2026-10-31**. It has not been newly cloud-cooked as part of this source export. Use the official cloud cooking/publishing workflow to produce distributable game files; do not install these editor assets directly into the game.

`ModId` and `TestingModId` are both reset to `0` in this copy. Associate your own mod.io listing through the official editor before publishing. The live development project was not changed. The display name is **Extra Trinket Slots**; the internal plugin name remains unchanged to preserve asset references. The `.uplugin` version fields are retained authoring metadata, not a new public release version.

## Compatibility and limits

- Every player must install and enable the mod for multiplayer use.
- Equipment indices **13–16** (zero-based) hold the extra items; the backing inventory has at least 17 entries. Another mod or a future game update using the same indices can conflict. There is no automatic index allocation or migration.
- Remove all trinkets from the extra slots before disabling/uninstalling the mod. Automatic refunds without the mod are not implemented.
- Extra trinkets do not add attached character models. Shift-click auto-equip into extra slots is not implemented.
- This snapshot handles equipped buffs and temperature resistance. Direct `ArmorBonus` and damage-type mitigation have not been extended.
- Console and dedicated-server runtime compatibility is not certified by this source export. See [validation notes](docs/VALIDATION.md) for the actual test scope.

## License

MIT for the author's original mod logic and documentation; retain the copyright and license notice when reusing it. Game/engine assets and other third-party material are excluded from that license grant. See [LICENSE](LICENSE) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
