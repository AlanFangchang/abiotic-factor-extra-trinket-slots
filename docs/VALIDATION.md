# Validation scope — 2026-10-31

The local temperature-resistance fix passed these editor checks before source packaging:

| Check | Scope |
| --- | --- |
| Single-player UI transfers | 22 recorded checks: native/extra stacking to 6, tier changes, locked removal, extra-slot movement, native recalculation, and coexistence with an equipped buff. |
| Heat/data-path fixtures | 11 checks using scripted placement of existing armor rows to exercise HeatResist and values greater than 1. This bypassed placement rules for testing only; no item definitions were changed. |
| Two-player PIE | 16 checks using authoritative native transfers, verifying server and client resistance values and independence between players. |
| State recreation and death | Locked items retained through component recreation, totals restored on reactivation, and extra resistance removed after the asynchronous native death sequence. |
| Asset verification | State graph: 188 live nodes, no compiler/validation errors or warnings, replication enabled, 0.25-second tick. |

Direct Python calls to the client item-slot function produced local prediction without a complete server transfer. Those calls were not counted as successful end-to-end client input tests. The multiplayer checks instead drove native transfers on authority and asserted replicated client results.

This run did not perform a disk save reload, an independent dedicated-server test, console runtime testing, or a new cloud cook/upload. State recreation is not a substitute for a full save/reload test. Earlier project testing is not re-certified by this snapshot.

The release copy retains all 11 current assets byte-for-byte. Internal mod dependencies were checked for missing and scratch/test assets. Publication IDs were cleared only in the release copy; no developer tools, logs, saves, credentials or test scripts are distributed.
