# JBUFO - Absurd Surreal Biological Flight Engine v4

A high-performance real-time Canvas 2D rendering demonstration featuring organic physics simulation, advanced particle systems, and GPU-optimized compositing. Models a biological entity in flight with dynamic thrust, physiological response, and emergent visual effects.

## 🎨 Project Overview

**JBUFO** (Just Before Unfathomable Functional Obsolescence) is an experimental WebGL-alternative that explores biological aesthetics through pure Canvas 2D implementation. It combines:

- **Organic tissue rendering** with procedural vascular networks
- **Real-time particle propulsion** using object pooling
- **Physics-driven visuals** responsive to G-force and acceleration
- **GPU-optimized layer composition** via off-screen rendering
- **Dynamic HUD telemetry** synced to flight parameters

## 🏗️ Technical Architecture

### Rendering Pipeline

| Layer | Purpose | Implementation |
|-------|---------|-----------------|
| **Particle System** | Exhaust thrust effect | 600-particle object pool + sprite atlas + Screen blend mode |
| **Off-screen Body** | Tissue texture cache | Pre-rendered Canvas (250×450px) with veins & pores |
| **Specular Layer** | Dynamic highlights | Second Canvas for rim lighting, gloss, and caustics |
| **Physics Simulation** | Motion & acceleration | G-force, blood pressure pulse, camera shake |
| **HUD Telemetry** | Real-time metrics | FPS, Mach speed, G-force, vascular pressure warnings |

### Key Systems

#### 1. **High-Performance Particle Pool** (Lines 77–109)
```javascript
class BetterParticlePool {
  - 600 pre-allocated particles
  - Flame & smoke sprite rendering
  - Radial gradient atlas (128×128px each)
  - Screen composite mode for additive blending
  - Life-based alpha fade & size scaling
}
```
**Critical Variables:**
- `p.life`, `p.maxLife` → fade opacity
- `p.size`, `p.maxInitalSize` → particle expansion (smoke grows, flame shrinks)
- `p.vx`, `p.vy` → velocity vector relative to entity rotation

#### 2. **Organic Tissue Rendering** (Lines 118–255)
**Core Body Canvas (`offscreenBody`):**
- Ellipse-based torso with bilateral symmetry
- Sfumato shadow gradients (smooth tonal transitions)
- Procedural vascular network (deep veins + surface arteries)
- Micro-scale pore texture via noise pattern overlay

**Specular Highlights Canvas (`offscreenSpecular`):**
- Bottom engine rim light (engine exhaust caustics)
- Central shaft gloss (wet, reflective surface)
- Glans rim reflection (apex highlight)
- All constrained to body boundary via `source-in` blend mode

**Critical Gradients:**
```javascript
testicleGrad    // Deep shadow (#421d17 → #ebbaa9)
shaftGrad       // Shaft midtone (#542a20 → #f5ccbe)
glansGrad       // Apex flushed (#ff8a8a → #3b0a0a)
specTesticle{L,R}  // Engine bottom heat (#fff5c8 → transparent)
specShaft       // Centerline gloss (razor-thin white line)
specGlans       // Rim light reflection (orange to transparent)
```

#### 3. **Physics & Motion** (Lines 260–287)
```javascript
entity {
  x, y              // Position (screen space)
  vy                // Vertical velocity (-2.5 px/frame upward)
  angle             // Rotation (±0.04 rad from sin wave)
  baseSpeed         // Thrust magnitude (2.4 px)
  gForce            // Acceleration (3.5 ± 2.5·sin(t·0.001))
  bloodPressure     // Physiological response (0–1)
  noiseOffset       // Lateral drift seed
}
```

**Physics Pipeline:**
1. Update position: `entity.y += entity.vy`
2. Apply drift: `entity.x += sin(t + offset) * 2.2`
3. Calculate rotation: `angle = sin(t * 0.006) * 0.04`
4. Compute G-force: `gForce = 3.5 + sin(t * 0.001) * 2.5`
5. Pulse blood pressure: `bloodPressure = (sin(t * 0.005 * gForce) + 1) / 2`
6. Spawn particles at dual engine points (L/R offset by rotation matrix)
7. Apply camera shake proportional to G-force

#### 4. **Dynamic Compositing** (Lines 299–338)

**Frame Render Order:**
```javascript
1. Clear background (#050508)
2. Apply camera shake (translate)
3. Render particle exhaust (Screen blend mode)
4. Translate to entity position & rotate by entity.angle
5. Draw pre-rendered body texture (cached)
6. Apply blood pressure mask:
   - Only if gForce > 2
   - color-dodge blend mode
   - Radial gradient centered on entity
   - Alpha = bloodPressure * 0.4 * (gForce / 6)
7. Render specular highlights:
   - Screen blend mode (additive)
   - Alpha modulated by flame flicker (0.5 + random * 0.5)
   - Syncs optical response to thrust particle density
8. Restore transforms
```

#### 5. **HUD Telemetry** (Lines 17–33, 289–297)

**Real-time Bindings:**

| Display | Formula | Update Rate |
|---------|---------|------------|
| `FPS` | `frames/sec` | 1000ms |
| `TARGET_SPEED` | `baseSpeed + gForce*0.1` | 1000ms |
| `DYNAMIC_G` | `gForce.toFixed(2)` | 1000ms |
| `WARNING` | `gForce > 5.5 ? visible : hidden` | Per-frame |

**Styling:**
- Matrix terminal aesthetic (Courier New mono)
- Green glow text shadow (#00ff00 at 5px blur)
- Warning text in red (#ff3333) with 8px glow
- Positioned absolute, pointer-events: none

## 📊 Visual Effect Breakdown

### Particle Exhaust
- **Flame**: 20–45px radius, lifespan 28 frames, shrinks as it fades
- **Smoke**: 30–80px radius, lifespan 55 frames, expands as it fades
- **Spawn Rate**: 3 flame + 1–2 smoke per physics tick per engine
- **Velocity**: Relative to entity rotation, ±2.5 lateral jitter, +6 px/frame thrust

### Tissue Engorging
When `gForce > 2`:
- Radial gradient (red core #ff3232 → dark red #640000)
- Bleeds outward up to 140px radius
- **Alpha** = `bloodPressure * 0.4 * (gForce / 6)`
  - At gForce=6: max 40% visibility
  - Pulsates 5× per second in rhythm with blood pressure
- **Blend Mode**: `color-dodge` (brightens underlying tissue)

### Highlight Flicker
- **Specular Alpha**: `0.5 + Math.random() * 0.5` per frame
- **Effect**: High-frequency shimmer synced to flame particle emission
- **Blend Mode**: `screen` (additive)
- **Perceptual**: Makes tissue appear wet, reflective, and thermally reactive

### Camera Shake
```javascript
cameraShake = gForce > 5 ? 5.5 + random*2 : 2.5 + sin(t*0.05)
// Apply: ctx.translate((random - 0.5) * cameraShake, (random - 0.5) * cameraShake)
```
- **High G state** (>5): ±3.5 px violent jitter
- **Normal flight** (<5): ±1.25 px subtle vibration
- Creates visceral sense of acceleration stress

## 🎮 Interaction Model

Currently **non-interactive** (read-only visualization). Input vectors available for extension:
- `entity.x` → mouse/touch horizontal control
- `entity.vy` → acceleration throttle
- `entity.angle` → yaw steering
- `entity.baseSpeed` → thrust magnitude

## ⚙️ Performance Optimization

### Memory Efficiency
- **Object Pool**: 600 particles pre-allocated, reused per frame
- **Off-screen Caching**: Body/specular rendered once on init, reused every frame
- **Sprite Atlas**: All particle sprites baked to single 512×256 Canvas

### GPU Optimization
- **Composite Operations**: `source-atop`, `color-dodge`, `screen` for layer blending
- **Transform Batching**: Single `ctx.translate()` + `ctx.rotate()` before draw
- **Gradient Caching**: Radials recomputed per-frame (unavoidable for dynamic pulse)

### Canvas Quality
- Full viewport size, resized on window resize
- `globalCompositeOperation` switches minimize state changes
- `ctx.save()` / `ctx.restore()` bracket complex transforms

## 📝 Code Structure

```
index.html
├── Particle Atlas (Lines 52–75)
│   ├── Flame gradient (radial, 128×128)
│   └── Smoke gradient (radial, 128×128)
├── Particle Pool (Lines 77–109)
│   ├── spawn(), updateAndDraw()
│   └── Life cycle & rendering
├── Off-screen Assets (Lines 111–255)
│   ├── Body texture (veins, pores, sfumato shadows)
│   └── Specular highlights (rim light, gloss, caustics)
├── Physics Engine (Lines 260–287)
│   ├── Position, velocity, rotation
│   ├── G-force & blood pressure calculation
│   └── Engine point displacement
├── Render Loop (Lines 289–342)
│   ├── FPS meter
│   ├── Physics update
│   ├── Composite pipeline
│   └── requestAnimationFrame recursion
└── HUD Display (Lines 17–33)
    ├── FPS, Speed, G-force
    └── Vascular pressure warning
```

## 🚀 Deployment

1. Clone or download `index.html`
2. Open in any modern browser (Chrome 90+, Firefox 88+, Safari 15+)
3. Runs immediately at full screen resolution
4. No external dependencies or build step required

**Browser Support:**
- Canvas 2D API
- `requestAnimationFrame()`
- CSS transforms & compositing
- ES6 class syntax

## 🎓 Educational Value

### Demonstrates:
- **Particle Systems**: Object pooling, sprite atlasing, batch rendering
- **Off-screen Rendering**: Canvas caching, layer composition, performance optimization
- **Physics Simulation**: Kinematics, G-force modeling, procedural animation
- **Gradient Rendering**: Radial/linear gradients, color stops, Sfumato shading
- **Blend Modes**: `screen`, `color-dodge`, `source-atop`, `source-in` semantics
- **Transform Pipelines**: Matrix composition, hierarchical transforms, GPU acceleration

### Advanced Concepts:
- Vascular system dynamics (pressure-based pulse)
- Thermal visualization (caustics, specular highlighting)
- Procedural organic geometry (Bézier curves, symmetry)
- Frame-locked animation timing
- Composite operation state management

## 📄 License

Unlicensed / Public Domain

## 🔗 References

- [MDN Canvas 2D API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Composite Operations Spec](https://www.w3.org/TR/2dcontext/#compositing)
- [requestAnimationFrame Timing](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames)
- [Sfumato Technique](https://en.wikipedia.org/wiki/Sfumato)

---

**Created**: June 2026 | **Status**: Stable | **FPS Target**: 60
