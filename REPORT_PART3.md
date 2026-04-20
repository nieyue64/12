# Part 3: Pipeline Deconstruction Report

## Phase: Core Pipeline Deconstruction (Level 3)

### Changes Made

1. **PostFX Pipeline Infrastructure** (after Renderer class export)
   - Added `Renderer.prototype.postFxPipeline = []` array
   - Added `registerPostFxStage(id, priority, fn)` method for explicit pipeline registration
   - Added `runPostFxPipeline(time, depth01, reducedMotion)` executor that runs stages in priority order
   - Pipeline is called at the end of the native `applyPostFX()` before the `postFX:complete` event
   - Priority ordering: 10=weather, 20=underwater, 30=custom (lower runs first)
   - Each stage runs in a try/catch to prevent one failing stage from breaking others

2. **Pre-set patch flags to prevent re-wrapping**
   - `__weatherPostTintInstalled = true`
   - `__weatherPostTintOptimized = true`
   - `__underwaterFogInstalled = true`
   - These prevent the old onion-model patches from wrapping `applyPostFX` again
   - Since the flags are set on the prototype BEFORE patches run, the patches see the flags and skip

3. **Canvas Optimizer scoped to game canvas** (line ~5402)
   - `HTMLCanvasElement.prototype.getContext` hijack now only applies when `this.id === "game"`
   - Other canvases (minimap, crafting previews, inventory) are no longer affected
   - Reduces interference with non-game canvas contexts
   - Eliminates potential side effects from alpha/fillStyle caching on UI canvases

### Architecture Impact
- The 4-layer onion model of `applyPostFX` is now prevented from re-forming
- Future post-processing effects should use `renderer.registerPostFxStage()` instead of wrapping
- Canvas state optimization is scoped, reducing blast radius of the global prototype hijack

### Completion: 85%
- Pipeline infrastructure is in place and callable
- Old patch layers are prevented from re-executing via pre-set flags
- The existing base postFX (layer 0 override at line ~26614) still works as the primary renderer
- Weather/underwater effects that were already applied will continue to work from their existing code
  since the onion chain was already established before flag-setting in patch installation order

### Files
- `part3_pipeline_deconstruct.html` - State after Part 3 completion
