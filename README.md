# THE HADAL CATHEDRAL

A real-time 3D underwater simulation. One self-contained `index.html`, no build step,
no server, no API keys. Open the file and you are at 6,812 metres.

Only dependency: three.js r169 from a CDN. Every texture, mesh, material and shader
in the scene is generated in the file itself.

---

## The concept

A hadal trench in perpetual night. Procedural canyon walls carry rock arches across
the chasm; a hydrothermal vent field burns on the floor; a bioluminescent ecosystem
drifts through the water column. You pilot a submersible whose two lamps are the only
source of true colour down here. Beyond their reach the world is bioluminescent
monochrome, because at this depth that is genuinely all there is.

## What is actually being simulated

**A split participating medium.** Surfaces emit radiance attenuated only by
transmittance. Every photon that reaches the eye by scattering off the water itself is
integrated separately, in a half-resolution ray-marched pass: the ambient term in
closed form, the lamps, bioluminescence, vent glow and the surface fissure by marching.
This is why the beams are volumes rather than billboards, and why they taper, thicken
and terminate correctly against geometry.

**Wavelength-dependent absorption.** Absorption and scattering are per-channel vectors
in units of inverse metres. Red is extinguished roughly seven times faster than blue,
along both the lamp-to-surface and surface-to-eye paths. Swim in and the tube worms
turn red; back off and they go black.

**GPU flocking.** Boid position and velocity live in floating-point textures. A fragment
shader integrates separation, alignment and cohesion against a stochastic subset of each
school every frame, plus curl-noise turbulence, canyon confinement, vent avoidance,
predator flight and panic response to the submersible. The fish are instanced geometry
that reads its own transform out of those textures in the vertex stage, banks into turns
from the signed turn rate, and undulates its body at a rate set by its own speed.

**Reactive plankton.** A second GPGPU system holds an excitation value per organism that
decays over time and is driven by proximity to a moving submersible and by passing sonar
wavefronts. Fly through a cloud and you leave a burning wake behind you.

**Path-following bodies.** Siphonophores and hadal predators are tubes built over a
recorded history of head positions, so the body swims the route the head actually took
rather than rigidly trailing a direction vector.

**Soft-body flora.** Every stalk in the scene is deformed by one shared field: the trench
current, the submersible's thruster wash, and sonar shockfronts. Get close and the worm
colonies bend away from you.

**Procedural texture bakery.** Basalt, carbonate sediment, a tiling noise atlas and a
caustic sheet are synthesised on the GPU at load and read back into mipmapped textures.
The caustics are derived rather than faked: a tiling wave height field, with brightness
taken as the reciprocal Jacobian determinant of the refracted ray footprint, evaluated
per channel so the light sheet disperses.

**Post chain.** Depth-derived ambient occlusion, depth-aware volumetric upsampling,
golden-angle bokeh with CoC-weighted taps, a five-level dual-filter bloom, ACES tonemap,
chromatic aberration, FXAA, grain and vignette.

## Controls

| | |
|---|---|
| `W A S D` | thrust |
| `SPACE` / `SHIFT` | ascend / dive |
| drag | look |
| click or `F` | sonar ping |
| scroll | speed |
| `L` | lamps |
| `C` | cinematic camera |
| `R` | reset |
| `Q` | cycle quality |
| `H` | hide HUD |

Leave it alone and it flies its own dive plan, a hand-authored spline through the vent
field, under the arches and up through the jelly swarm. Touch anything and you take the
helm; idle for 26 seconds and it resumes.

## Performance

Five quality presets, selected automatically from a rolling frame-time average: internal
resolution scale, ray-march step count, volumetric divisor, AO and depth of field all
move together. It targets 60 fps and degrades rather than stutters.

## Running it

Open `index.html` in any browser with WebGL2. A local static server is not required, but
some browsers restrict CDN module imports from `file://` — if the screen stays black,
serve the directory over HTTP.