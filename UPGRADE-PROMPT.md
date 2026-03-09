# Pod Device — Second Pass Upgrade Prompt

> Copy everything below this line and paste it as the opening message to a fresh Claude Code session.
> Working directory: `~/codex-root/pod-device/`

---

## Context

You are upgrading a single-file Three.js scroll-driven 3D exploded view experience (`index.html`, 1,068 lines). The project is a procedurally generated pod device ("AURA") that explodes into 16+ internal components as the user scrolls. It uses Three.js r170, GSAP ScrollTrigger, Lenis smooth scroll, UnrealBloom, vignette post-processing, SMAA, and ambient particles. Everything is procedural — no external models or textures. It must remain a single HTML file with no build step.

The goal: take every aspect of this experience to photorealistic, film-quality, award-winning levels. This is the second pass — the architecture and scroll choreography are solid. Now we push materials, lighting, shading, post-processing, detail density, and visual polish to the absolute frontier of what's possible in a browser.

**Rules:**
- Single `index.html` file — no build step, no external assets (CDN imports are fine)
- All geometry remains procedural (no GLB/GLTF)
- Preserve the existing 8-section scroll structure and choreography
- Preserve the superellipsoid pod shape, all 16+ components, the label system
- British spelling in all text/comments
- Commit incrementally after each major upgrade phase completes (not at the end)
- Deploy to Vercel after all phases complete (`vercel --yes --prod`)
- GitHub repo: `tomasi001/pod-device` — push to `main`

## Phase 1: Environment & Lighting Overhaul

### 1A. Replace DirectionalLights with RoomEnvironment + RectAreaLights

The current lighting uses 3 DirectionalLights + 1 AmbientLight. This produces flat, unrealistic illumination. Replace with:

**RoomEnvironment (procedural studio HDRI — zero external files):**
```js
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';

const pmremGenerator = new THREE.PMREMGenerator(renderer);
const envTexture = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture;
scene.environment = envTexture;
// Do NOT set scene.background to envTexture — keep the dark background
pmremGenerator.dispose();
```

This gives every MeshPhysicalMaterial and MeshStandardMaterial realistic environment reflections and ambient lighting from a procedural studio setup. It replaces the AmbientLight entirely.

**RectAreaLights for hero product lighting:**
```js
import { RectAreaLightUniformsLib } from 'three/addons/lights/RectAreaLightUniformsLib.js';
import { RectAreaLightHelper } from 'three/addons/helpers/RectAreaLightHelper.js'; // dev only

RectAreaLightUniformsLib.init();

// Key light — large soft panel, warm white, top-right
const keyRect = new THREE.RectAreaLight(0xfff8f0, 8, 4, 4);
keyRect.position.set(3, 4, 4);
keyRect.lookAt(0, 0, 0);
scene.add(keyRect);

// Fill light — cool blue panel, left side
const fillRect = new THREE.RectAreaLight(0xb8c8e8, 3, 3, 3);
fillRect.position.set(-4, 1, -2);
fillRect.lookAt(0, 0, 0);
scene.add(fillRect);

// Rim/accent — narrow purple strip, behind
const rimRect = new THREE.RectAreaLight(0xa78bfa, 5, 1, 4);
rimRect.position.set(0, -1, -4);
rimRect.lookAt(0, 0, 0);
scene.add(rimRect);
```

Keep the PointLight (`innerGlow`) for the shell-open reveal — it's perfect for that.

Remove all DirectionalLights and the AmbientLight. The RoomEnvironment PMREM handles ambient, and RectAreaLights handle directed illumination.

### 1B. Animate lighting intensity with scroll

Drive light intensities from scroll progress via the GSAP master timeline:
- Hero (0–8%): Key at 8, fill at 3 — bright product shot
- Shell open (8–22%): Dim key to 5, increase rim to 8 — dramatic reveal
- Full explode (78–92%): Key at 6, fill at 4, rim at 3 — even illumination for all components
- Reassemble (92–100%): Return to hero lighting

---

## Phase 2: Material Upgrade — MeshPhysicalMaterial Everywhere

Every material currently uses MeshStandardMaterial (except shell and lens). Upgrade ALL materials to MeshPhysicalMaterial with physically accurate properties. The RoomEnvironment from Phase 1 is critical — metallic materials look dead without an environment map.

### Material Recipes

**Shell (already MeshPhysicalMaterial — enhance):**
```js
const shellMat = new THREE.MeshPhysicalMaterial({
  color: 0xf0f0f4,
  roughness: 0.15,        // smoother — premium feel
  metalness: 0.02,         // nearly dielectric
  clearcoat: 1.0,          // full clearcoat — like ceramic/glass
  clearcoatRoughness: 0.05, // mirror-sharp clearcoat
  sheen: 0.3,              // subtle sheen for pearlescence
  sheenRoughness: 0.4,
  sheenColor: new THREE.Color(0xd0d0ff), // cool pearl tint
  reflectivity: 0.9,
  envMapIntensity: 1.2,
  side: THREE.DoubleSide,
});
```

**PCB Board:**
```js
const pcbMat = new THREE.MeshPhysicalMaterial({
  color: 0x0a3520,          // dark green FR4
  roughness: 0.65,
  metalness: 0.0,           // non-metallic substrate
  clearcoat: 0.4,           // solder mask gives slight gloss
  clearcoatRoughness: 0.3,
  specularIntensity: 0.3,
  specularColor: new THREE.Color(0x225533),
});
```

**Chips (Neural Engine, Memory, Wireless):**
```js
const chipMat = new THREE.MeshPhysicalMaterial({
  color: 0x12122a,
  roughness: 0.12,
  metalness: 0.7,
  iridescence: 0.4,         // silicon wafer rainbow
  iridescenceIOR: 1.3,
  iridescenceThicknessRange: [200, 500],
  specularIntensity: 1.2,
  envMapIntensity: 1.5,
});

const chipTopMat = new THREE.MeshPhysicalMaterial({
  color: 0x22223a,
  roughness: 0.08,
  metalness: 0.8,
  iridescence: 0.6,
  iridescenceIOR: 1.4,
  iridescenceThicknessRange: [150, 400],
  clearcoat: 0.3,
  clearcoatRoughness: 0.1,
  envMapIntensity: 1.8,
});
```

**Battery (Graphene Cell):**
```js
const batteryMat = new THREE.MeshPhysicalMaterial({
  color: 0xc0c0ca,
  roughness: 0.1,
  metalness: 0.95,
  anisotropy: 0.6,           // brushed metal finish
  anisotropyRotation: 0,     // horizontal brush direction
  clearcoat: 0.2,
  clearcoatRoughness: 0.15,
  envMapIntensity: 1.4,
});
```

**Gold Traces:**
```js
const goldTraceMat = new THREE.MeshPhysicalMaterial({
  color: 0xdaa520,
  roughness: 0.1,
  metalness: 1.0,
  clearcoat: 0.3,            // solder mask creates slight gloss
  clearcoatRoughness: 0.4,
  specularIntensity: 1.5,
  specularColor: new THREE.Color(0xffdd66),
  envMapIntensity: 1.6,
});
```

**Antenna (Gold):**
```js
const antennaMat = new THREE.MeshPhysicalMaterial({
  color: 0xc8a84e,
  roughness: 0.15,
  metalness: 1.0,
  specularIntensity: 1.3,
  specularColor: new THREE.Color(0xffe088),
  envMapIntensity: 1.5,
});
```

**Haptic Engines:**
```js
const hapticMat = new THREE.MeshPhysicalMaterial({
  color: 0x888890,
  roughness: 0.08,
  metalness: 0.98,
  anisotropy: 0.4,           // machined finish
  clearcoat: 0.5,
  clearcoatRoughness: 0.08,
  envMapIntensity: 1.6,
});
```

**Copper (Heat Spreader):**
```js
const copperMat = new THREE.MeshPhysicalMaterial({
  color: 0xb87333,
  roughness: 0.08,
  metalness: 1.0,
  specularIntensity: 1.5,
  specularColor: new THREE.Color(0xffaa66),
  envMapIntensity: 1.8,
});
```

**Camera Housing:**
```js
const cameraMat = new THREE.MeshPhysicalMaterial({
  color: 0x1a1a22,
  roughness: 0.25,
  metalness: 0.7,
  clearcoat: 0.8,            // camera glass cover
  clearcoatRoughness: 0.05,
  envMapIntensity: 1.2,
});
```

**Lens (already MeshPhysicalMaterial — enhance):**
```js
const lensMat = new THREE.MeshPhysicalMaterial({
  color: 0x080830,
  roughness: 0.0,
  metalness: 0.0,
  transmission: 0.95,
  thickness: 1.5,            // magnifying distortion
  ior: 1.62,                 // flint glass
  attenuationColor: new THREE.Color(0xaabbff), // blue tint
  attenuationDistance: 1.0,
  dispersion: 0.15,          // chromatic dispersion — rainbow edges
  specularIntensity: 1.0,
  envMapIntensity: 2.0,
});
```

**Cable:**
```js
const cableMat = new THREE.MeshPhysicalMaterial({
  color: 0xe89030,
  roughness: 0.45,
  metalness: 0.15,
  sheen: 0.5,                // flex cable has a slight sheen
  sheenRoughness: 0.6,
  sheenColor: new THREE.Color(0xffcc88),
});
```

**Speaker:**
```js
const speakerMat = new THREE.MeshPhysicalMaterial({
  color: 0x222228,
  roughness: 0.4,
  metalness: 0.5,
  clearcoat: 0.3,
  clearcoatRoughness: 0.2,
});
```

**LiDAR Emitter — add emissive pulsing:**
```js
const lidarEmitterMat = new THREE.MeshPhysicalMaterial({
  color: 0x200020,
  roughness: 0.2,
  metalness: 0.5,
  emissive: 0x6600aa,
  emissiveIntensity: 0.5,   // animate this with time for pulsing
});
```

**Inner Glow — keep as MeshBasicMaterial** (it's meant to be an unlit emissive sphere).

---

## Phase 3: Post-Processing Pipeline — Film Grade

### 3A. Replace the current post-processing chain

Current chain: RenderPass → UnrealBloomPass → Vignette (custom) → SMAA → OutputPass

New chain (order matters):

```
RenderPass
  → Custom Fresnel Glow Pass (new)
  → UnrealBloomPass (tuned)
  → Chromatic Aberration Pass (new)
  → Depth of Field / Bokeh Pass (new — optional, scroll-driven)
  → Film Grain Pass (new)
  → Vignette Pass (enhanced)
  → SMAA Pass (desktop only)
  → OutputPass
```

### 3B. Tone Mapping

Switch from ACES Filmic to **Khronos PBR Neutral** — purpose-built for product visualisation with true-to-life colour reproduction:

```js
renderer.toneMapping = THREE.NeutralToneMapping;
renderer.toneMappingExposure = 1.0;
```

Three.js r170 has `THREE.NeutralToneMapping` built in (maps to the Khronos PBR Neutral tonemapper).

### 3C. Chromatic Aberration Pass

Add a custom ShaderPass for subtle lens chromatic aberration:

```js
const chromaticAberrationShader = {
  uniforms: {
    tDiffuse: { value: null },
    uIntensity: { value: 0.0015 },    // subtle — increase for sci-fi
    uDirection: { value: new THREE.Vector2(1.0, 0.0) },
  },
  vertexShader: `varying vec2 vUv; void main() { vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }`,
  fragmentShader: `
    uniform sampler2D tDiffuse;
    uniform float uIntensity;
    uniform vec2 uDirection;
    varying vec2 vUv;
    void main() {
      vec2 offset = uIntensity * uDirection;
      // Radial modulation — stronger at edges like a real lens
      float dist = length(vUv - 0.5) * 2.0;
      offset *= dist * dist;
      float r = texture2D(tDiffuse, vUv + offset).r;
      float g = texture2D(tDiffuse, vUv).g;
      float b = texture2D(tDiffuse, vUv - offset).b;
      float a = texture2D(tDiffuse, vUv).a;
      gl_FragColor = vec4(r, g, b, a);
    }
  `
};
const chromaticPass = new ShaderPass(chromaticAberrationShader);
```

Drive `uIntensity` from scroll velocity for dynamic lens feel:
```js
// In animation loop:
const velocity = Math.abs(ST.getVelocity?.() || 0) / 1000;
chromaticPass.uniforms.uIntensity.value = 0.001 + velocity * 0.003;
```

### 3D. Film Grain Pass

Subtle photographic noise for cinematic feel:

```js
const filmGrainShader = {
  uniforms: {
    tDiffuse: { value: null },
    uTime: { value: 0 },
    uIntensity: { value: 0.04 },
  },
  vertexShader: `varying vec2 vUv; void main() { vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }`,
  fragmentShader: `
    uniform sampler2D tDiffuse;
    uniform float uTime;
    uniform float uIntensity;
    varying vec2 vUv;
    // Hash-based noise
    float hash(vec2 p) {
      vec3 p3 = fract(vec3(p.xyx) * 0.1031);
      p3 += dot(p3, p3.yzx + 33.33);
      return fract((p3.x + p3.y) * p3.z);
    }
    void main() {
      vec4 color = texture2D(tDiffuse, vUv);
      float noise = hash(vUv * 1000.0 + uTime * 100.0) * 2.0 - 1.0;
      color.rgb += noise * uIntensity;
      gl_FragColor = color;
    }
  `
};
const grainPass = new ShaderPass(filmGrainShader);
```

Update `grainPass.uniforms.uTime.value = t;` in the animation loop.

### 3E. Enhance Existing Passes

**UnrealBloomPass — tighten for selective glow:**
```js
const bloomPass = new UnrealBloomPass(
  new THREE.Vector2(W, H),
  isMobile ? 0.25 : 0.4,  // strength — slightly reduced for cleaner look
  0.4,                      // radius — tighter bloom
  0.85                      // threshold — higher = only bright emissives bloom
);
```

**Vignette — enhance with colour tint:**
Update the vignette fragment shader to add a subtle purple tint to the darkened edges:
```glsl
void main() {
  vec4 c = texture2D(tDiffuse, vUv);
  float d = distance(vUv, vec2(0.5));
  float vignette = smoothstep(0.8, uOffset * 0.5, d * (uDarkness + uOffset));
  // Tint dark edges with very subtle purple
  vec3 tint = mix(vec3(0.02, 0.01, 0.04), vec3(1.0), vignette);
  c.rgb *= tint;
  c.rgb *= vignette;
  gl_FragColor = c;
}
```

---

## Phase 4: Geometry Detail Density

### 4A. Increase PCB Component Count with InstancedMesh

The PCB currently has just 9 gold traces. A real PCB has hundreds of components. Add instanced SMD components:

```js
// SMD capacitors — tiny rectangular components scattered on PCB
{
  const smdGeo = new THREE.BoxGeometry(0.012, 0.006, 0.005);
  const smdMat = new THREE.MeshPhysicalMaterial({
    color: 0x2a2a35, roughness: 0.3, metalness: 0.6,
    clearcoat: 0.4, clearcoatRoughness: 0.2,
  });
  const smdCount = isMobile ? 40 : 120;
  const smdMesh = new THREE.InstancedMesh(smdGeo, smdMat, smdCount);

  const dummy = new THREE.Object3D();
  const boardW = 0.7, boardH = 0.38;

  for (let i = 0; i < smdCount; i++) {
    dummy.position.set(
      (Math.random() - 0.5) * boardW * 0.85,
      0.035 + Math.random() * 0.003,    // just above board surface
      (Math.random() - 0.5) * boardH * 0.85
    );
    dummy.rotation.set(0, Math.random() * Math.PI, 0);
    dummy.scale.setScalar(0.8 + Math.random() * 0.4);
    dummy.updateMatrix();
    smdMesh.setMatrixAt(i, dummy.matrix);
  }
  smdMesh.instanceMatrix.needsUpdate = true;
  parts.smdComponents = smdMesh;
  device.add(smdMesh);
}
```

Add a second InstancedMesh for SMD resistors with different colour (tan/beige):
```js
{
  const resGeo = new THREE.BoxGeometry(0.01, 0.004, 0.005);
  const resMat = new THREE.MeshPhysicalMaterial({
    color: 0x8b7355, roughness: 0.5, metalness: 0.2,
    clearcoat: 0.2, clearcoatRoughness: 0.3,
  });
  const resCount = isMobile ? 30 : 80;
  const resMesh = new THREE.InstancedMesh(resGeo, resMat, resCount);
  // ... same placement pattern as above, slightly different positions
  parts.resistors = resMesh;
  device.add(resMesh);
}
```

Add solder pads (tiny metallic circles at trace endpoints):
```js
{
  const padGeo = new THREE.CylinderGeometry(0.004, 0.004, 0.001, 8);
  const padMat = new THREE.MeshPhysicalMaterial({
    color: 0xc0c0c0, roughness: 0.2, metalness: 0.9,
  });
  const padCount = isMobile ? 50 : 200;
  const padMesh = new THREE.InstancedMesh(padGeo, padMat, padCount);
  // ... scatter across board surface
  parts.solderPads = padMesh;
  device.add(padMesh);
}
```

### 4B. Increase Gold Trace Density

The current 9 traces are too sparse. Increase to 20+ horizontal + 12+ vertical traces with varying widths, creating a realistic circuit pattern. Add some diagonal traces too. Group them all in the existing `parts.traces` group.

### 4C. Add BGA Pads Under Neural Chip

Add a grid of tiny solder balls under the neural chip (Ball Grid Array):
```js
{
  const bgaGeo = new THREE.SphereGeometry(0.002, 6, 4);
  const bgaMat = new THREE.MeshPhysicalMaterial({
    color: 0xc0c0c0, roughness: 0.15, metalness: 0.95,
  });
  const gridSize = isMobile ? 5 : 8;
  const bgaCount = gridSize * gridSize;
  const bgaMesh = new THREE.InstancedMesh(bgaGeo, bgaMat, bgaCount);
  // Grid pattern under chip position
  const dummy = new THREE.Object3D();
  const spacing = 0.012;
  let idx = 0;
  for (let row = 0; row < gridSize; row++) {
    for (let col = 0; col < gridSize; col++) {
      dummy.position.set(
        -0.08 + (col - gridSize/2) * spacing,
        0.035,
        -0.02 + (row - gridSize/2) * spacing
      );
      dummy.updateMatrix();
      bgaMesh.setMatrixAt(idx++, dummy.matrix);
    }
  }
  bgaMesh.instanceMatrix.needsUpdate = true;
  parts.bgaPads = bgaMesh;
  device.add(bgaMesh);
}
```

### 4D. Add Connector Ports

Add small connector geometry to the pod edges (USB-C style port, SIM tray slot):
```js
{
  const connGroup = new THREE.Group();
  // USB-C port — small rectangle cutout on bottom edge
  const portGeo = new THREE.BoxGeometry(0.03, 0.008, 0.012);
  const portMat = new THREE.MeshPhysicalMaterial({
    color: 0x0a0a0a, roughness: 0.3, metalness: 0.8,
  });
  const port = new THREE.Mesh(portGeo, portMat);
  port.position.set(0, -0.01, RY * 0.72); // bottom edge of pod
  connGroup.add(port);

  // Inner pins (gold contacts)
  const pinGeo = new THREE.BoxGeometry(0.002, 0.003, 0.008);
  for (let i = 0; i < 6; i++) {
    const pin = new THREE.Mesh(pinGeo, goldTraceMat);
    pin.position.set(-0.01 + i * 0.004, -0.01, RY * 0.72);
    connGroup.add(pin);
  }
  parts.connectors = connGroup;
  device.add(connGroup);
}
```

---

## Phase 5: Custom Shader Effects

### 5A. Fresnel Edge Glow on Shell

Add a custom fresnel rim glow to the shell material using `onBeforeCompile`:

```js
shellMat.onBeforeCompile = (shader) => {
  shader.uniforms.uFresnelPower = { value: 3.0 };
  shader.uniforms.uFresnelColor = { value: new THREE.Color(0x6e5dfa) };
  shader.uniforms.uFresnelIntensity = { value: 0.0 }; // animate with scroll

  shader.fragmentShader = shader.fragmentShader.replace(
    '#include <output_fragment>',
    `
    // Fresnel rim glow
    vec3 viewDir = normalize(vViewPosition);
    vec3 worldNormal = normalize(vNormal);
    float fresnel = pow(1.0 - abs(dot(viewDir, worldNormal)), uFresnelPower);
    vec3 fresnelGlow = uFresnelColor * fresnel * uFresnelIntensity;
    outgoingLight += fresnelGlow;
    #include <output_fragment>
    `
  );

  shader.uniforms.uFresnelPower = { value: 3.0 };
  // Store reference for animation
  shellMat.userData.shader = shader;
};
```

Drive `uFresnelIntensity` from 0 → 0.6 as the shell opens (scroll units 10–20), then back to 0 during reassemble.

### 5B. Animated Data Flow on Gold Traces

Make gold traces pulse with animated emissive energy:

```js
goldTraceMat.onBeforeCompile = (shader) => {
  shader.uniforms.uTime = { value: 0 };
  shader.uniforms.uFlowIntensity = { value: 0.0 }; // animate with scroll

  shader.fragmentShader = shader.fragmentShader.replace(
    '#include <emissivemap_fragment>',
    `
    #include <emissivemap_fragment>
    // Animated data flow
    float flow = sin(vUv.x * 40.0 - uTime * 3.0) * 0.5 + 0.5;
    flow = pow(flow, 4.0); // sharpen pulses
    totalEmissiveRadiance += vec3(0.9, 0.7, 0.2) * flow * uFlowIntensity;
    `
  );

  goldTraceMat.userData.shader = shader;
};
```

Drive `uFlowIntensity` from 0 → 1.0 when the PCB is visible (scroll units 50–78), combined with bloom for glowing data flowing through circuits.

### 5C. LiDAR Pulse Effect

Make the LiDAR emitter pulse with emissive:

In the animation loop:
```js
if (lidarEmitterMat.emissiveIntensity !== undefined) {
  lidarEmitterMat.emissiveIntensity = 0.3 + Math.sin(t * 4.0) * 0.5;
}
```

---

## Phase 6: Enhanced Particle System

### 6A. Dual Particle Layers

Keep the existing ambient particles but add a second, denser particle layer that activates during the explosion:

**Inner particles (device-centred, activated on shell open):**
```js
{
  const INNER_COUNT = isMobile ? 200 : 800;
  const innerPos = new Float32Array(INNER_COUNT * 3);
  const innerSizes = new Float32Array(INNER_COUNT);
  for (let i = 0; i < INNER_COUNT; i++) {
    // Concentrated around device centre
    innerPos[i * 3]     = (Math.random() - 0.5) * 2.0;
    innerPos[i * 3 + 1] = (Math.random() - 0.5) * 2.0;
    innerPos[i * 3 + 2] = (Math.random() - 0.5) * 2.0;
    innerSizes[i] = Math.random() * 1.5 + 0.3;
  }

  // Shader with scroll-driven opacity and upward drift
  const innerParticleMat = new THREE.ShaderMaterial({
    transparent: true, depthWrite: false,
    uniforms: {
      uTime: { value: 0 },
      uOpacity: { value: 0 },     // driven by scroll — 0 when assembled, 0.5 when exploded
      uColor: { value: new THREE.Color(0xa78bfa) },  // purple — matches brand
    },
    vertexShader: `
      attribute float size;
      uniform float uTime;
      varying float vAlpha;
      void main() {
        vec3 p = position;
        // Gentle upward spiral
        float angle = uTime * 0.3 + position.y * 2.0;
        p.x += sin(angle) * 0.08;
        p.z += cos(angle) * 0.08;
        p.y += sin(uTime * 0.2 + position.x) * 0.12;
        vec4 mv = modelViewMatrix * vec4(p, 1.0);
        gl_Position = projectionMatrix * mv;
        gl_PointSize = size * (150.0 / -mv.z);
        vAlpha = smoothstep(5.0, 0.5, -mv.z);
      }
    `,
    fragmentShader: `
      uniform float uOpacity;
      uniform vec3 uColor;
      varying float vAlpha;
      void main() {
        float d = length(gl_PointCoord - 0.5);
        if (d > 0.5) discard;
        float a = smoothstep(0.5, 0.0, d) * uOpacity * vAlpha;
        gl_FragColor = vec4(uColor, a);
      }
    `
  });

  const innerGeo = new THREE.BufferGeometry();
  innerGeo.setAttribute('position', new THREE.Float32BufferAttribute(innerPos, 3));
  innerGeo.setAttribute('size', new THREE.Float32BufferAttribute(innerSizes, 1));

  const innerParticles = new THREE.Points(innerGeo, innerParticleMat);
  scene.add(innerParticles);
}
```

Drive `uOpacity` via GSAP:
- 0 during hero
- Fade to 0.5 as shell opens (units 10–18)
- Peak at 0.7 during full explode (units 78–92)
- Fade to 0 during reassemble

---

## Phase 7: Scroll Velocity Effects

### 7A. Velocity-Driven Dynamic Effects

In the animation loop, read scroll velocity and drive multiple effects simultaneously:

```js
// In animate():
const rawVelocity = Math.abs(lenis.velocity || 0);
const normVelocity = Math.min(rawVelocity / 2000, 1.0); // 0–1 normalised

// 1. Chromatic aberration scales with velocity
chromaticPass.uniforms.uIntensity.value = THREE.MathUtils.lerp(
  chromaticPass.uniforms.uIntensity.value,
  0.0008 + normVelocity * 0.004,
  0.1
);

// 2. Bloom intensity slightly increases during fast scroll
bloomPass.strength = THREE.MathUtils.lerp(
  bloomPass.strength,
  (isMobile ? 0.25 : 0.4) + normVelocity * 0.15,
  0.1
);

// 3. Film grain increases slightly during fast scroll
grainPass.uniforms.uIntensity.value = THREE.MathUtils.lerp(
  grainPass.uniforms.uIntensity.value,
  0.03 + normVelocity * 0.03,
  0.1
);
```

All values use `lerp` for smooth transitions — never jarring.

---

## Phase 8: Camera Path — Spline-Based

### 8A. Replace linear camera tweens with CatmullRomCurve3

Currently the camera jumps between 7 fixed positions via linear GSAP tweens. Replace with a smooth spline orbit:

```js
const cameraPath = new THREE.CatmullRomCurve3([
  new THREE.Vector3(0, 0.3, 3.8),       // Hero — front centre
  new THREE.Vector3(1.5, 0.6, 3.0),     // Shell open — slightly right, elevated
  new THREE.Vector3(2.2, 0.4, 2.0),     // Connectivity — orbit right, closer
  new THREE.Vector3(-1.8, -0.3, 2.5),   // Power — swing left, drop down
  new THREE.Vector3(0.5, 1.5, 2.8),     // Brain — elevated top-down
  new THREE.Vector3(1.8, 0.3, 2.5),     // Senses — orbit right
  new THREE.Vector3(0, 0.6, 4.5),       // Full explode — pull back wide
  new THREE.Vector3(0, 0.3, 3.8),       // Reassemble — return to start
], false, 'catmullrom', 0.5);            // tension 0.5 for smooth curves
```

Replace the camera position tweens in the master timeline with a single proxy animation:

```js
const cameraProxy = { progress: 0 };
masterTL.to(cameraProxy, { progress: 1, duration: 100, ease: 'none' }, 0);

// In animate():
const camPos = cameraPath.getPointAt(cameraProxy.progress);
camera.position.copy(camPos);
camera.lookAt(0, 0, 0);
```

This creates a silky-smooth orbital camera path instead of jarring position jumps.

---

## Phase 9: Visual Polish

### 9A. Label Line Connectors

Currently labels are just floating text. Add thin lines connecting each label to its component:

```html
<svg id="label-lines" style="position:fixed;inset:0;z-index:9;pointer-events:none;width:100%;height:100%"></svg>
```

In `updateLabels()`, for each visible label, draw an SVG line from the label dot to a screen-space projection of the component:
```js
const svgNS = 'http://www.w3.org/2000/svg';
const linesSvg = document.getElementById('label-lines');

function updateLabels() {
  // Clear previous lines
  while (linesSvg.firstChild) linesSvg.removeChild(linesSvg.firstChild);

  Object.entries(labelMap).forEach(([key, { el, obj }]) => {
    const vis = labelVisibility[key] || 0;
    if (vis < 0.05) { el.style.opacity = '0'; return; }

    obj.updateWorldMatrix(true, false);
    _projVec.setFromMatrixPosition(obj.matrixWorld);
    _projVec.project(camera);

    if (_projVec.z > 1) { el.style.opacity = '0'; return; }

    const x = (_projVec.x * 0.5 + 0.5) * W;
    const y = (-_projVec.y * 0.5 + 0.5) * H;

    // Position label with offset
    const labelX = x;
    const labelY = y - 30;
    el.style.transform = `translate(${labelX}px, ${labelY}px)`;
    el.style.opacity = String(vis);

    // Draw connector line
    if (vis > 0.1) {
      const line = document.createElementNS(svgNS, 'line');
      line.setAttribute('x1', String(x));
      line.setAttribute('y1', String(y));
      line.setAttribute('x2', String(labelX));
      line.setAttribute('y2', String(labelY + 12)); // to label dot
      line.setAttribute('stroke', `rgba(167, 139, 250, ${vis * 0.4})`);
      line.setAttribute('stroke-width', '1');
      line.setAttribute('stroke-dasharray', '2,3');
      linesSvg.appendChild(line);
    }
  });
}
```

### 9B. Scroll Progress Indicator

Add a minimal scroll progress bar on the right edge:

```html
<div id="scroll-progress" style="position:fixed;top:0;right:0;width:2px;height:100vh;z-index:20;pointer-events:none">
  <div id="scroll-fill" style="width:100%;height:0%;background:linear-gradient(180deg,#6e7bff,#a78bfa);transition:height .1s ease"></div>
</div>
```

Update in animation loop:
```js
document.getElementById('scroll-fill').style.height = `${scrollProgress * 100}%`;
```

### 9C. Enhanced Loading Screen

Add a rotating wireframe pod preview during loading:

Before the main scene initialises, render a simplified wireframe of the pod shape that rotates. When loading completes, crossfade from wireframe to the full scene. This gives immediate visual feedback.

---

## Phase 10: Performance Safeguards

### 10A. Mobile Quality Tiers

Add a quality tier system:

```js
const QUALITY = (() => {
  if (isMobile) return 'low';
  const gl = renderer.getContext();
  const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
  const gpuRenderer = debugInfo ? gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL) : '';
  // Integrated GPUs get medium, dedicated GPUs get high
  if (/Intel|Mali|Adreno/i.test(gpuRenderer)) return 'medium';
  return 'high';
})();
```

Conditionally disable expensive features:
```js
// Film grain — skip on low
if (QUALITY !== 'low') composer.addPass(grainPass);

// Chromatic aberration — skip on low
if (QUALITY !== 'low') composer.addPass(chromaticPass);

// Inner particles — reduce count
const INNER_COUNT = QUALITY === 'low' ? 100 : QUALITY === 'medium' ? 400 : 800;

// Shell segments
const shellSegs = QUALITY === 'low' ? 48 : QUALITY === 'medium' ? 64 : 72;

// SMD components
const smdCount = QUALITY === 'low' ? 20 : QUALITY === 'medium' ? 60 : 120;
```

### 10B. Resize Handler — debounce

Debounce the resize handler to prevent excessive re-renders:

```js
let resizeTimeout;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimeout);
  resizeTimeout = setTimeout(() => {
    W = window.innerWidth;
    H = window.innerHeight;
    camera.aspect = W / H;
    camera.updateProjectionMatrix();
    renderer.setSize(W, H);
    composer.setSize(W, H);
    bloomPass.resolution.set(W, H);
  }, 150);
});
```

---

## Implementation Order

Execute these phases in this exact order, committing after each:

1. **Phase 1** (Lighting) — biggest visual impact, foundation for all materials
2. **Phase 2** (Materials) — depends on Phase 1's environment map
3. **Phase 3** (Post-Processing) — depends on Phases 1–2 for correct input
4. **Phase 4** (Geometry Detail) — independent, adds density
5. **Phase 5** (Shader Effects) — depends on Phase 2 materials
6. **Phase 6** (Particles) — independent
7. **Phase 7** (Velocity Effects) — depends on Phase 3 passes
8. **Phase 8** (Camera Path) — independent
9. **Phase 9** (Visual Polish) — finishing touches
10. **Phase 10** (Performance) — final guard rails

After all phases, `git push origin main` and `vercel --yes --prod`.

---

## Quality Checklist

Before considering this complete, verify:

- [ ] Shell has visible environment reflections with clearcoat highlights
- [ ] Chips show iridescent rainbow at glancing angles
- [ ] Gold traces have specular highlights and animated data flow glow
- [ ] Battery shows brushed-metal anisotropic reflections
- [ ] Lens has visible refraction distortion and chromatic dispersion
- [ ] Copper heat spreader has bright orange-tinted specular highlights
- [ ] Inner purple particles appear when shell opens
- [ ] Chromatic aberration strengthens during fast scroll
- [ ] Film grain is visible but subtle
- [ ] Camera follows a smooth orbital path (no position jumps)
- [ ] Labels have dashed connector lines to their components
- [ ] LiDAR emitter pulses with emissive glow
- [ ] Vignette has subtle purple tint at edges
- [ ] All 16+ original components are preserved
- [ ] No visible performance degradation on desktop
- [ ] Mobile renders without crashes (reduced quality is fine)
- [ ] Scroll experience is smooth throughout (no janks)
- [ ] All 8 sections render and transition correctly
