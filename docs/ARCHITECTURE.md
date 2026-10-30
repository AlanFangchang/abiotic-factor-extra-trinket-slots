# How the extra trinket slots work

[Chinese version](ARCHITECTURE.zh-CN.md) · [Back to README](../README.md)

This guide follows the current Blueprints in `Mods/TrinketSlotProbes2QpQ/Content`. It is intended for Modkit authors who want to understand the implementation or trace it in the editor.

The core idea is to **put four real items in extra entries of the player's existing equipment inventory, draw four widgets for those entries inside the native inventory, and explicitly manage their effects**. The widgets look and behave like native slots, but the game does not automatically treat indices 13–16 as native trinket slots.

```text
Class Manager → Bootstrap → components on the player
                                  ├─ State: slots, unlock rules, effects
                                  └─ UI: four widgets in the native inventory
                                                ↓
                                native equipment inventory indices 13–16
```

The state component reads the equipped hacking device directly from the equipment inventory.

## 1. Start the mod and attach components

Open `DataTables/DT_ClassManager/Class_TrinketSlotsBootstrap` first. Its `ClassManager.EntityDirector` entry registers `BP_TrinketSlots_Bootstrap` with the official Class Manager.

The bootstrap runs on the server, finds player characters and ensures that each has four slot metadata components, a State component and a UI component. The UI component subsequently creates the widgets. Their responsibilities are:

| Asset | Responsibility |
| --- | --- |
| `BP_ExtraTrinketSlot1`–`4` | Per-slot metadata: activation tags and placement requirements. These are not separate item stores. |
| `BP_TrinketSlots_State` | Equipment capacity, tier rules and effect reconciliation. |
| `BP_TrinketSlots_UI` | Creates the local player's extra widgets in the native inventory. |
| `WBP_TrinketSlot_Runtime` | Displays and handles one extra slot; created by the UI component. |

State and UI are dynamically added and explicitly configured to replicate. Their setup waits for required character and inventory objects, because component creation and replication do not necessarily finish in the same order.

## 2. Store the actual items

`BP_TrinketSlots_State` uses the player's `CharacterEquipSlotInventory`. On the server it expands `CurrentInventory` to **at least 17 entries** when necessary, then reserves four zero-based indices:

| Equipment index | Purpose |
| --- | --- |
| 7, 11 | Original trinket slots |
| 9 | Hacking device |
| 13, 14, 15, 16 | Extra trinket slots, in order |

The extra widgets point at this same inventory and these actual indices. The four metadata components do not duplicate the items. Using the native equipment array also lets the existing player save pipeline retain them; the component's temporary caches and UI can be reconstructed from that inventory.

The client waits for the server's inventory array to replicate rather than creating its own authoritative slots. This choice avoids a separate custom save file, but reserves indices that another mod or a future game update could also use. There is no automatic allocation or migration.

## 3. Turn a hacking device into slot permissions

State reads the row name of the equipped item at index 9 and checks these identifiers:

| Row name | Active extra slots |
| --- | --- |
| `keypad_hacker_t3` | 1 |
| `keypad_hacker_t4` | 2 |
| `keypad_hacker_t5` | 3 |
| `Gatekey` | 4 |
| Anything else or empty | 0 |

For each slot, State combines this tier check with the player's alive state. It writes the slot's placement requirement into `ContainerTagRequirementOverrides` and updates the corresponding metadata component's tags. The UI uses `TrinketSlotUnlocked` and the `TrinketSlotRequiresT…` tags to choose its appearance and empty-slot hint.

The unlocked tag query is `ALL(Item.Gear)`; the locked query is `NONE(Item)`. The unlocked query is a broad gear filter, not a trinket-only check. The native slot widget handles equipment-slot interaction, while the mod's explicit trinket-type check controls placement highlighting. Highlighting and inventory acceptance should not be treated as the same check.

When a slot locks, its **item stays in the equipment array**. The locked requirement blocks a new item from being placed there, while the user can still take the existing item out through the inventory interface. Effects are considered separately, so retained items in locked slots no longer contribute them. Re-equipping a suitable device makes retained items active again. During death, the same active-state check prevents extra effects from being reapplied.

## 4. Prevent duplicate world items

The native equipment inventory has an `AnySlotStateChanged` listener bound to the character's `CheckForGearStateChange`. For unfamiliar indices 13–16, that native handler can spawn a gear actor without attaching or hiding it, leaving a collectible duplicate at the player's feet.

State removes **that listener only** and binds its own `TrinketSlots_FilterNativeGear` delegate. Events for indices outside 13–16 are forwarded to `CheckForGearStateChange` as before; events for the four extra indices are not forwarded to that handler. Other inventory listeners remain in place. The mod still uses the native inventory for the real items; it only avoids the inappropriate native gear-actor handling for these extra indices.

## 5. Apply equipped buffs

On each state refresh, State reads each extra item's `EquipmentData.BuffsWhileEquipped` and builds the desired buff handles from **unlocked, alive** extra slots. On the server, it then reconciles those handles with the effects it tracks. Only the server applies or removes buffs.

- If an old effect is no longer desired, State checks original slots 7 and 11 before removing it, so a buff still needed by a native trinket is preserved.
- If a desired buff is not already present, State calls the character's buff component to apply it and remembers the handle.
- State also considers effects from retained extra items during reconciliation, including the first refresh after restoration. That gives it candidates to remove when those slots are locked.

Buff handles are deduplicated. The mod does not create an additional stack merely because two extra slots name the same buff. The native-slot ownership check only accounts for those two trinket slots; it should not be read as a general source-ownership system for every possible buff source.

## 6. Add direct temperature resistance

Not every trinket effect is a buff. `ColdResist` and `HeatResist` live directly in `EquipmentData`, so applying `BuffsWhileEquipped` alone misses them. State sums each value across valid, active extra slots, including repeated copies of the same trinket.

When equipment, extra contributions or the observed native total changes, State calls the native `UpdateTemperatureResist` to obtain the original equipment baseline, then adds the extra-slot sums and writes `CurrentColdResist` and `CurrentHeatResist`. It caches the equipment and previous values so it can detect a reset or change without repeatedly adding on top of its own previous result. These character properties use the game's replication/notification mechanism.

For example, two original slots and four unlocked extra slots containing six trinkets with `ColdResist = 1` contribute 6 in total. Locking three of the extra slots removes those three contributions while preserving their items. Direct `ArmorBonus` and damage-type mitigation are not extended by this mechanism.

## 7. Insert the visible slots into the native inventory

`BP_TrinketSlots_UI` acts only for the locally controlled player. Once the HUD and metadata components are available, it follows the native widget hierarchy:

```text
GetLocalPlayerHUD
  → W_PlayerInventory_Main
    → W_Inventory_EquipSlots
      → Pane
```

It creates four grid contexts backed by `CharacterEquipSlotInventory`, creates a `WBP_TrinketSlot_Runtime` for each, assigns indices 13–16, and adds the widgets to `Pane` as canvas children. The current layout is X = 126.6, Y = 357.7 + 88 × the zero-based extra-slot number, with a size of 84 × 84. Because they are children of the native inventory, they follow its visibility when the backpack opens and closes.

`WBP_TrinketSlot_Runtime` derives from the native item-slot widget. It reads its current item from the assigned native inventory, refreshes the icon only when item data changes, and selects opacity and tooltip from the activation metadata. For placement highlighting it checks the held item's native container compatibility and its equipment type, so unlocked extra slots highlight for trinkets rather than helmets or backpacks.

## 8. Authority, persistence and testing boundaries

State ticks at a 0.25-second interval. The server and locally controlled player evaluate relevant state; only authority expands the real inventory or applies/removes effects. UI creation is local. Players need the mod enabled on both sides for the intended multiplayer experience.

Removing the mod removes its UI and logic for accessing the reserved indices. There is no automatic refund when the mod is disabled: take trinkets out of the extra slots before uninstalling. Extra trinkets do not create attached character models, and Shift-click automatic equip into these slots is not implemented.

See [VALIDATION.md](VALIDATION.md) for what was actually tested, including the limits of the two-player automation. Editor verification does not itself certify every platform, dedicated server, save/reload scenario or future game version.
