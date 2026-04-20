# Part 2: Event-Driven Migration Report

## Phase: Event-Driven Migration (Level 2)

### Changes Made

1. **Native `SaveSystem.markTile()` enhanced** (line ~9523)
   - Absorbed world version updates from `__logicMarkTilePatched` directly into native method
   - Now updates `world._tileVersion` and `world._waterVersion` (for water tiles) inline
   - Calls `__tuBumpWaterChunkVer` for water chunk versioning
   - Added new `save:tileMarked` event emission with `{ x, y, newId, oldId }` payload
   - This replaces the need for any downstream code to wrap `markTile`

2. **`__logicMarkTilePatched` wrapper eliminated** (line ~28314)
   - Was wrapping `SaveSystem.prototype.markTile` to update world tile/water versions
   - Logic absorbed into native `markTile` (see above)
   - Old patch reduced to a flag-set no-op

3. **`__machineIndexPatched` converted to event listener** (line ~30124)
   - Was wrapping `SaveSystem.prototype.markTile` to call `game._syncMachineAt`
   - Converted to listen for `save:tileMarked` event instead
   - Uses `window.__tuMachineMarkTileListener` guard to prevent duplicate registration
   - Machine index sync now fully decoupled from SaveSystem prototype chain

### Architecture Impact
- `SaveSystem.markTile` prototype chain reduced from 3 layers to 1 (native only)
- Machine sync is now event-driven, not prototype-chain-dependent
- Future extensions can listen to `save:tileMarked` without touching SaveSystem

### Completion: 90%
- Both markTile hijacks resolved
- game:init:post double trigger was already fixed in Part 0
- Remaining lifecycle patterns (game:update listeners) are architectural and working well via GameEvents

### Files
- `part2_event_driven.html` - State after Part 2 completion
