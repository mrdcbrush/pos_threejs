# Grocery checkout kit for three.js

A set of texture-free glTF 2.0 (GLB) models of a grocery self-checkout lane, plus a small scene runtime that plays a scripted 15-second checkout in three.js. Version 0.1.0.

The kit is distributed as `grocery-checkout-kit-glb.zip`.

## What's in the zip

| Path | Contents |
| --- | --- |
| `checkout-station.glb` | Full lane, 9.4k triangles, 6 animation clips (3.39 × 2.23 × 1.43 m) |
| `components/*.glb` | The parts on their own: scanner-scale, pos-terminal, pin-pad, receipt-printer, bag-carousel, lane-light, divider-bar |
| `checkout-station-preview.png` | Rendered preview |
| `scene.json` | Manifest: which assets to load, where to place them, and a timeline of events |
| `scene-runtime.js` | three.js code that reads `scene.json`, loads the GLBs and plays the timeline |
| `source/` | Python scripts that regenerate every GLB (`build_station.py`, `glbkit.py`, `validate.py`) |
| `ASSETS.md` | Asset conventions, node names, material names, animation clips and anchors |
| `INSTRUCTIONS.md` | How to build a scene from the kit |

## Quick start

Unzip, then load the scene from a page that has three.js available:

```html
<div id="scene" style="width:100%;height:100vh"></div>
<script type="module">
  import * as THREE from 'three';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  import { createCheckoutScene } from './scene-runtime.js';

  const manifest = await (await fetch('scene.json')).json();
  createCheckoutScene({
    THREE, GLTFLoader, OrbitControls,
    container: document.getElementById('scene'),
    manifest,
    assetBase: './',
  });
</script>
```

The included scene runs one checkout on lane 4: three stand-in items ride the belt, get scanned and bagged, the total shows, the cash drawer opens, a receipt prints and tears off, and the bag carousel turns.

`createCheckoutScene` returns an object with `fire(event)`, which triggers any timeline event immediately. This is the hook for driving the scene from live data, for example blinking the lane light when a queue metric crosses a threshold:

```js
app.fire({ target: 'lane4', glow: 'Glow_LaneLamp', blink: 2, duration: 5 });
```

## Conventions

- 1 unit = 1 metre, +Y is up.
- Origins sit on the floor at the centre of the footprint. Files in `components/` have their origin at the mounting base and drop onto a counter at y = 0.87.
- Groceries flow toward +X. The cashier works from the +Z side and the customer stands on the -Z side.
- Colours are stored in linear space and render correctly in three.js with sRGB output.
- Every file uses the same `Kit_*` material names, so combined assets share one palette.
- Glowing parts use separate `Glow_*` emissive materials so screens, lamps and the scanner window can be animated independently. Values above 1.0 use `KHR_materials_emissive_strength`, which three.js r137 and later reads.

See `ASSETS.md` inside the zip for the full list of nodes, materials, clips and anchors.

## Rebuilding the models

The scripts in `source/` need only Python 3 and NumPy:

```
python3 build_station.py out
python3 validate.py out
```

`validate.py` checks headers, buffer bounds, index ranges, normals, keyframe order and unique node names. To make a different lane number, change `LANE_NUMBER` in `build_station.py`.

## License

MIT. See `LICENSE`.
