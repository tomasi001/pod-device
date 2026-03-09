# Stress Test — Scroll-Animated 3D Exploded Device Website

> Date: 2026-03-09
> This prompt is a capability evaluation. Your output will be assessed for technical depth, visual quality, animation craftsmanship, and creative ambition. Treat this as a portfolio piece — the bar is the highest work you are capable of producing.

---

## What You Are Building

A single-page, scroll-driven website that showcases a futuristic screenless AI device. The device is a smooth, oval pod — roughly phone-sized, thinner than a phone, with tapered edges on the long axis (think a river stone or a flattened egg, not a box with rounded corners). As the user scrolls, the pod splits open and its internal components explode outward in a choreographed sequence, revealing the engineering inside. Scrolling further pulls the camera back to show the full exploded constellation, then rapidly collapses everything back into the assembled pod.

There is no physical product. All geometry must be procedurally generated — no external 3D models, textures, or assets. The entire experience lives in a single HTML file with no build step. CDN imports for libraries are fine.

Ground yourself in the latest web graphics research and standards as of March 2026. Use the most current stable versions of any libraries you choose. Do not rely on outdated patterns or deprecated APIs.

---

## The Device

**Exterior:** A smooth, organic pod shape. Symmetrically tapered on both the top/bottom faces and the left/right long edges. The silhouette should read as an elongated oval or a superellipsoid — not rectangular. Premium material — think ceramic, polished composite, or pearl. The shell splits into a top half and a bottom half along the equator.

**Interior components (minimum — add more if you can justify them):**

- Phased antenna array (distributed around the perimeter)
- Dual haptic engines (left and right)
- Solid-state battery (large, flat, metallic)
- Flex ribbon cables (organic curves connecting subsystems)
- Main logic board / PCB (dark green, with visible gold circuit traces)
- Primary processing chip (small, dark, with a heat spreader above it)
- Memory module
- Wireless communications module
- Camera module (housing + glass lens element)
- Microphone array (cluster of tiny elements)
- Speaker / audio transducer
- Depth sensor (emitter + receiver pair)
- Connector port (USB-C style with internal gold contact pins)
- Surface-mount components scattered across the PCB (capacitors, resistors — use instancing)
- Solder pads and a ball-grid array under the main chip

Each component should have a distinct material that reflects what it actually is — brushed metal for the battery, gold for traces and antenna elements, dark silicon with iridescence for chips, copper for the heat spreader, translucent glass for the lens, matte green for the PCB substrate. Every material should respond to environment lighting with physically accurate reflections.

---

## Scroll Choreography

The page is divided into 8 scroll sections. The 3D canvas is fixed behind the scroll content. Text sections overlay the 3D with clean, minimal typography. The full scroll is roughly 1600vh.

### Section 1 — Hero (short, ~100vh)
The assembled pod, gently rotating. Title and subtitle appear. The device looks pristine, whole, precious.

### Section 2 — Shell Opens (tall, ~250vh)
The top and bottom shell halves separate vertically, revealing the interior. A warm inner glow or light intensifies as the gap widens. This is the first dramatic moment — it should feel like cracking open a jewel.

### Section 3 — Connectivity Layer (~200vh)
The antenna array and haptic engines float outward from the device. Labels appear, connected to their components with thin indicator lines. Camera orbits gently to showcase the parts.

### Section 4 — Power Layer (~200vh)
The battery drops downward and away. Flex cables drift out. The camera swings to frame this separation.

### Section 5 — Compute Layer (tall, ~250vh)
This is the centrepiece. The PCB/logic board separates downward, away from the components mounted on it. The surface-mount components (capacitors, resistors, solder pads) detach from the board and remain floating in the centre — a suspended constellation of tiny parts.

**Critical moment:** The camera dives in extremely close to these floating central components. Almost touching them. It should orbit slowly around them at this intimate distance — the user should feel like they're inspecting the components under a magnifying glass. The device's idle rotation should be nearly still during this phase so the camera movement does all the work.

These floating components should have subtle 3D noise-driven movement — gentle oscillation on all three axes so they feel alive, not frozen. Each component should have its own phase offset so they move independently.

The camera then slowly pulls back as the user continues scrolling.

### Section 6 — Sensor Layer (~200vh)
Camera, microphone array, speaker, and depth sensor float outward. Labels appear. Camera continues pulling back to a medium distance.

### Section 7 — Full Exploded View (~200vh)
Everything is maximally separated. The camera is wide and elevated, slowly drifting. All labels visible. This is the "beauty shot" of the full teardown. Linger here.

### Section 8 — Reassembly (short, ~100vh)
**Rapid collapse.** Every component snaps back to its assembled position in a fast, aggressive animation. The easing should be heavy — slow start, explosive finish (like gravity pulling everything together). Labels cut instantly. The pod is whole again. Final title: "The future has no screen."

---

## Camera

The camera should follow a smooth spline path through 3D space, driven by scroll position. No linear jumps between positions — the movement should feel like a continuous Steadicam shot through the device. Key requirements:

- Hero: front-centre, medium distance
- Shell open: slight orbit, pull back to frame the separation
- Connectivity/Power: orbital sweeps to showcase floating parts
- Compute: **dramatic dive inward to extreme close-up**, then slow orbit at intimate distance, then gradual pull-back
- Full explode: wide, elevated, slow drift
- Reassembly: snap back to hero framing

The camera always looks at the centre of the device.

---

## Lighting & Environment

Do not use basic directional lights. Use a procedural environment map (studio HDRI generated at runtime) so that every metallic and glossy surface shows realistic reflections. Use area lights for soft, shaped illumination — a warm key light, a cool fill, and a coloured accent/rim light.

Lighting intensity should shift with scroll progress — brighter and more dramatic during the shell-open reveal, even and clinical during the full explode, snapping back to hero lighting during reassembly.

---

## Materials

Every material should be physically based with properties appropriate to its real-world counterpart:

- Shell: high clearcoat, pearlescent sheen, near-zero metalness
- Chips: silicon iridescence (thin-film interference), high metalness
- Battery: brushed metal with anisotropic reflections
- Gold elements: full metalness, warm specular
- Copper: full metalness, orange-tinted specular
- Lens: glass transmission, refraction, chromatic dispersion
- PCB: non-metallic with solder-mask gloss (clearcoat)
- Cables: warm sheen, low metalness
- Machined steel parts: high metalness, anisotropic finish

The shell should have a Fresnel rim glow effect that activates when the shell opens and fades during reassembly — a subtle edge luminance in a cool accent colour.

Gold circuit traces should have animated emissive pulses — data flowing through the circuits, visible when the PCB is exposed.

---

## Post-Processing

Build a film-grade post-processing pipeline:

- Bloom (selective — only bright emissives and specular highlights should glow)
- Chromatic aberration (radial, stronger at screen edges, intensity scales with scroll velocity)
- Film grain (subtle, photographic, animated)
- Vignette (with a slight colour tint in the darkened edges)
- Anti-aliasing (SMAA or equivalent on desktop, skip on mobile)
- Tone mapping appropriate for product visualisation — accurate colour reproduction over artistic contrast

---

## UI & Labels

- Clean sans-serif typography (Inter or system font)
- Section text: large headlines (font-weight 200–300), small body text (weight 300, muted opacity)
- Text fades in and out as the user scrolls through each section
- 3D-projected labels that track their associated components, with thin dashed SVG connector lines
- A minimal scroll progress indicator (thin vertical bar on the right edge)
- Loading screen with visual feedback (animated wireframe preview of the device shape)

---

## Performance

- Detect GPU capability and establish quality tiers (low/medium/high)
- Conditionally disable expensive post-processing passes on lower tiers
- Use instanced meshes for repeated small components
- Debounce resize handlers
- Pre-compile all materials and render a warm-up frame before dismissing the loader (prevents first-frame shader compilation flicker)
- Target 60fps on desktop, 30fps minimum on mobile

---

## Technical Constraints

- **Single HTML file.** No build step. No framework scaffolding.
- **No external 3D models or textures.** All geometry is procedural. All materials are code-defined.
- **CDN imports only** for libraries (Three.js, GSAP, Lenis, etc.)
- **British spelling** in all visible text and code comments
- Smooth scroll library for normalised, buttery scroll input
- Scroll-to-animation binding via a timeline scrubbed by scroll position

---

## What "Better" Means

You are expected to exceed the specification above, not merely meet it. Some directions to consider (these are suggestions, not requirements — surprise us):

- More components with higher geometric detail
- Micro-animations on individual parts (spinning fans, pulsing LEDs, breathing heat pipes)
- Material transitions (wireframe → solid → textured as components are revealed)
- Procedural textures (circuit board trace patterns generated mathematically, not hard-coded)
- Spatial audio cues tied to scroll position
- View-dependent effects (parallax layers, depth-aware blur on non-focused components)
- A "free explore" mode after the scroll narrative completes where the user can orbit/zoom manually
- Anything else that demonstrates mastery of real-time 3D on the web in 2026

---

## Evaluation Criteria

Your output will be evaluated on:

1. **Visual fidelity** — Do the materials look physically real? Does the lighting create mood and depth?
2. **Animation craft** — Is the scroll choreography cinematic? Do transitions feel intentional and timed? Is the camera path smooth?
3. **Technical execution** — Is the code clean, performant, and well-structured? Does it handle edge cases (resize, mobile, GPU tiers)?
4. **Creative ambition** — Did you go beyond the brief? Did you add details, effects, or ideas that elevate the experience?
5. **Polish** — Are there any visual artefacts, timing glitches, or rough edges? Does it feel finished?

The complete output should be a single HTML file that can be opened in any modern browser and immediately delivers a world-class scroll-driven 3D product experience.
