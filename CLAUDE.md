# Creature -- working rules

A big lovable creature on a white floor: auto-rigged, spring spikes, cartoony clips, a mouth rig (visemes), and (coming) a voice
to talk with. Single page, three.js r180 vendored in `vendor/`, native ES modules, **no build step**. GitHub Pages serves `main`.

- **Push straight to `main`** (Colin previews on his phone; a branch cannot be tested). No PRs unless he asks.
- **Raise `BUILD` in index.html before every push** -- it is the cyan number top-right and the `?v=` on the model URL, so a phone
  can tell an old cache from a new build.
- He tests on his phone; you say "shipped unverified" with the build number. A quick headless load check is fine.
- No API keys in this repo, ever (the voice step will need a server; keys live there, never in chat or in git).

## How the model is made (re-run after any change to the source mesh)

    blender-addons/pipeline: orient.py (rotation 0 for this Tripo export) -> run_rig.sh (Auto-Rig Pro) -> spikes.py --mixamo
                             -> visemes.py (cut the mouth open, cavity + tongue, 11 shape keys)
    zap/tools/forge.cjs <rig in zap root> retarget-anims/library/profiles/creature.json out.glb --lib <clips> --mesh
    gltf-transform resize 2048 -> webp -> SPARSE (the shape keys: 6.5 -> 4.0 MB) -> draco  ->  models/creature.glb

- The source is `models/source/creature_01.zip` (his Tripo FBX + texture). The rig is ARP renamed to `mixamorig_*`, 24 crest
  chains `spike_<k>_<j>` (3 bones each). Clips are Mixamo, baked by the Forge (private library; never commit raw Mixamo files).
- Spike weight is GATED by how far fur stands out from the skull (spikes.py `GATE`, build 5): a chain root sits inside the skull, so
  scalp fur weighted to it bent his head whenever a spring moved.
- Springs, not clips, drive the spikes (`springs()` in index.html): inertia, a pull to rest, gravity, a head-sphere collider,
  a bend limit. `window.cr.SPRING` is live.

## The mouth (visemes.py)

- His mouth was a closed SLOT ~3.5 cm deep, joined at its back wall. The cut is a MINIMUM CUT on the faces round the mouth (above
  the line vs below it), with edges cheap deep in the pocket -- three seam heuristics (painted line, normals, depth path) all left
  the lips joined somewhere; a cut always separates. The dark cavity and tongue sit BEHIND the slot, hidden at rest.
- Jaw weight fades to zero at the corners (they are the hinge), and nothing is moved on a piece the cut did not label (eyes).
- Shape keys: jawOpen, AI E O U MBP FV L etc, smile, frown -- all three primitives (skin, cavity, tongue) carry them. The page
  drives them (`mouthStep`): TALK = browser voice + letters-to-visemes, MIC = loudness, held buttons for inspection.
