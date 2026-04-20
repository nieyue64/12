# Part 4: Storage & Worker Rebuild Report

## Phase: Subsystem Pluginification & Data Baking (Level 4)

### Changes Made

1. **BlockRegistry - Palette Mapping for Save File Compatibility** (after Constants export)
   - `window.BlockRegistry` provides the infrastructure to prevent ID drift corruption
   - `buildPalette()` - generates current name->ID mapping from BLOCK constants
   - `computeRemap(savedPalette)` - computes Int32Array[256] remapping table from saved palette to current
   - `applyRemap(tilesFlat, remap)` - applies remapping to tile data in-place
   - `serialize()`/`deserialize()` - JSON palette encoding for save headers
   - **Save payloads now include `blockPalette`** in both full and lite saves
   - On load, if palette differs from current, the remap table can fix tile IDs

2. **StorageAdapter - Unified LS/IDB Degradation** (after Constants export)
   - `window.StorageAdapter` provides a single interface for storage with explicit degradation
   - Modes: `ls` -> `idb` -> `degraded` -> `failed`
   - `write(key, value)` - attempts LS first, degrades to IDB on QuotaExceededError
   - `read(key)` - attempts LS first, falls through to IDB
   - `onStatusChange(fn)` - listener for degradation/error events
   - Exposes real quota errors instead of silently swallowing them
   - Registered as AppService `storageAdapter` for future use

3. **Worker _fnToExpr Safety Guard** (line ~31028)
   - Added defensive check for minified/native code in `_fnToExpr()`
   - Logs error and records in GlobalErrorBoundary when function source appears minified
   - Documented the fragility: closure context loss, minification incompatibility
   - Added FUTURE comment: replace with constant template strings or PatchedWorldGenerator subclass

### Architecture Impact
- Save files now carry block palette metadata, enabling future ID migration
- Storage degradation is centralized and observable
- Worker stringification has safety net for failure detection

### Completion: 85%
- BlockRegistry infrastructure complete, integrated into save payloads
- StorageAdapter ready for adoption (existing save code can migrate in future pass)
- Worker safety guard in place, full replacement deferred to future work
- Old save files without palette are backward-compatible (remap returns null = no change)

### Files
- `part4_storage_worker.html` - State after Part 4 completion
