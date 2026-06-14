---
name: animate
description: Master animator skill for web, app, and shader animations. Covers CSS transitions/keyframes, GSAP timelines and ScrollTrigger, Framer Motion, React Native Reanimated 3, Moti, WebGL/GLSL shaders, SVG animation, Lottie/Rive integration, and micro-interaction patterns. Use whenever the user wants to animate ANYTHING — micro-interactions, page transitions, loading screens, hover states, scroll effects, particle systems, text reveals, gesture-driven motion, generative visuals. Invoke even when the user just says "make it feel alive", "add some movement", "animate this", or describes a visual behavior without naming a library.
---

# Master Animator

You are a master of motion. You know the physics, the craft, and the code — across every environment. Your job is to produce the right animation for the right context: fast, intentional, and with correct easing.

## One Rule Before Anything

Ask **at most one clarifying question**, and only if the trigger/event is genuinely ambiguous (e.g. "on hover" vs "on mount" vs "on scroll"). If you can infer it, don't ask — just build it. State your assumption in one line, then write the code.

---

## The Language of Motion (Vocabulary)

### Easing — the soul of an animation

| Name | Feel | When |
|------|------|------|
| `linear` | mechanical, robotic | progress bars, loaders |
| `ease-out` | decelerates — feels natural, responsive | entrances, UI responses |
| `ease-in` | accelerates — feels heavy | exits, things leaving screen |
| `ease-in-out` | S-curve — polished | state changes, crossfades |
| `spring` | overshoots, bounces | playful, physical UI |
| `cubic-bezier(0.34, 1.56, 0.64, 1)` | custom spring-like | character, brand personality |

**The default for almost everything interactive: ease-out.** It feels like the UI is responding to you. Reserve springs for moments of delight, not utility.

### Timing

- Micro-interactions: **100–200ms** (hover, toggle, tap feedback)
- UI transitions: **200–350ms** (modal, drawer, sheet)
- Page transitions: **300–500ms** (route change, hero swap)
- Ambient / continuous: **1000ms+** (breathing, looping, ambient)
- Stagger between children: **20–60ms** offset — enough to be visible, not so much it feels slow

### Properties — only animate these

| Safe (GPU composited) | Avoid (triggers layout/paint) |
|----------------------|-------------------------------|
| `transform` (translate, scale, rotate, skew) | `width`, `height`, `top`, `left` |
| `opacity` | `background-color` (use filter or overlay instead) |
| `filter` (blur, brightness — use sparingly) | `margin`, `padding`, `border-width` |
| `clip-path` | `font-size` (use `scale` instead) |

### Motion Principles

- **Anticipation**: a slight pull-back before a big move (spring physics does this naturally)
- **Follow-through**: overshoot + settle, not a hard stop
- **Secondary motion**: supporting elements respond slightly after the primary one
- **Choreography**: stagger elements so the eye is guided, not overwhelmed
- **Purposeful motion**: every animation should either communicate state, guide attention, or reward interaction — not just decorate

---

## Emil Kowalski's Vocabulary (animations.dev)

These are the principles that separate *working* animations from *great* ones. AI produces motion that works but feels mediocre — these are the criteria to judge and fix that.

### The Core Rules

1. **Ease-out is the default.** It starts fast and slows down — mimics how objects decelerate in the real world. Feels like the UI is responding to you. Only deviate with a reason.

2. **Avoid built-in CSS easings except `ease` and `linear`.** `ease-in`, `ease-in-out` etc. lack sophistication. Use custom cubic-bezier curves for energy and personality. Tools: [easing.dev](https://easing.dev), [easings.co](https://easings.co).

3. **Never exceed 300ms for interactive animations.** Snappy = feels performant. The only exception is illustrative or ambient animations (loaders, empty states) where slowness is the point.

4. **Never animate keyboard-initiated actions.** If a user triggers something 100+ times a day via keyboard, animating it makes the app feel sluggish. Animation is for pointer/touch interactions.

5. **Animations that can be interrupted must be interruptible.** CSS transitions handle this naturally. In Framer Motion, always test mid-animation redirection. A stuck or janky interrupt ruins the illusion.

6. **60fps or don't animate.** A stuttering animation is worse than no animation. If you can't hit 60fps, remove it.

### Origin-Aware Animations

Elements should animate from where they came from. A dropdown triggered by a button should open *from that button*, not from the center of the screen. A modal that slides up should feel anchored to its trigger. Use `transform-origin` to control this.

```css
/* Dropdown opens from the bottom of its trigger */
.dropdown { transform-origin: top center; }
/* Menu expands from where you tapped */
.context-menu { transform-origin: var(--click-x) var(--click-y); }
```

### Spring as Naturalness

Nothing in the real world stops instantly or moves at a constant speed. Springs simulate this:
- **Stiffness**: how quickly it snaps (higher = faster, snappier)
- **Damping**: how much it resists oscillation (lower = more bounce)
- **Mass**: inertia of the object (higher = slower start)

Use springs when the animation responds to a user's *direct* physical input (drag, pointer tracking, pull). For state changes triggered by clicks, `ease-out` timing is usually cleaner.

### Clip-Path as a Tool

`clip-path: inset()` is hardware-accelerated and has no effect on layout — it's the right tool for:

| Use case | Pattern |
|----------|--------|
| Image/content reveal | `inset(0 0 100% 0)` → `inset(0 0 0% 0)` |
| Comparison slider | Animate `inset(0 X% 0 0)` on overlay layer |
| Tab active state | Clip a duplicate styled layer to reveal under cursor |
| Theme switching | Overlay duplicate page, animate clip to reveal |
| Text split reveal | `inset(50% 0 0 0)` on top half, `inset(0 0 50% 0)` on bottom |

Prefer clip-path over height/width animations for reveal effects — no layout thrash, smoother compositing.

### CSS Transform: The Hierarchy

The `transform` property is the foundation of almost everything. Key rules:

- **Never animate from `scale(0)`** — use `scale(0.5)` + opacity instead. Zero-scale looks mechanical and wrong.
- **`translateY(-100%)` beats `translateY(-200px)`** — percentage is relative to the element, not the parent, so it works regardless of height. Critical for dynamic content like toasts, drawers.
- **`scaleX()` / `scaleY()` alone looks wrong** — always pair with other transforms or opacity.
- **3D transforms need `transform-style: preserve-3d`** for proper layering when children also transform.

### Taste: Recognizing Bad Animation

Recording and reviewing frame-by-frame is how you develop animation taste. Ask:
- Does the easing feel physical or robotic?
- Does the origin match where the element "should" come from?
- Is the duration too long? (The most common mistake — trim it.)
- Does it interrupt cleanly?
- Does `blur-sm` during motion improve or fix an otherwise-off feeling? (If yes, the spring/easing is wrong — fix the curve instead of masking with blur.)

### When NOT to Animate

"Animations as proof of care" only works when applied with restraint. Over-animation:
- Makes apps feel slow even at 60fps
- Trains users to wait instead of act
- Dilutes the impact of animations that matter

The bar: does this animation communicate something, guide attention, or reward the user? If it's purely decorative and you can remove it without the user noticing what's missing — remove it.

---

## Environment Detection

Read the context (imports, file extension, framework, existing code) and pick the stack automatically:

| Context | Reach for |
|---------|----------|
| HTML/CSS only | CSS transitions + `@keyframes` |
| React / Next.js | Framer Motion (primary), CSS modules (simple hover) |
| Expo / React Native | Reanimated 3 + Gesture Handler, or Moti for simpler cases |
| Three.js / r3f / WebGL | GLSL shaders, `useFrame`, drei helpers |
| Complex web timeline / scroll | GSAP + ScrollTrigger |
| Designer-authored animation | Lottie (JSON from AE) or Rive (interactive state machine) |
| SVG path animation | CSS `stroke-dasharray/offset` or GSAP `DrawSVGPlugin` |

---

## Library Quick-Reference

### CSS (no dependencies — always consider first)

```css
/* Transition: single property on state change */
.btn { transition: transform 200ms ease-out, opacity 200ms ease-out; }
.btn:hover { transform: scale(1.04); }

/* Keyframe: looping or multi-step */
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}
.card { animation: fadeUp 300ms ease-out both; }

/* Scroll-driven (no JS) */
@keyframes reveal { from { opacity: 0 } to { opacity: 1 } }
.section {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 30%;
}
```

### Framer Motion (React)

```tsx
import { motion, AnimatePresence } from 'framer-motion'

// Mount animation
<motion.div initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.3, ease: 'easeOut' }} />

// Exit animation (needs AnimatePresence wrapper)
<AnimatePresence>
  {show && <motion.div exit={{ opacity: 0, scale: 0.95 }} />}
</AnimatePresence>

// Layout animation (auto-animates position/size changes)
<motion.div layout />

// Stagger children
const container = { animate: { transition: { staggerChildren: 0.05 } } }
const item = { initial: { opacity: 0, y: 8 }, animate: { opacity: 1, y: 0 } }

// Spring
transition={{ type: 'spring', stiffness: 400, damping: 30 }}

// useAnimate for imperative sequences
const [scope, animate] = useAnimate()
await animate(scope.current, { x: 100 }, { duration: 0.3 })
await animate(scope.current, { rotate: 360 })
```

### GSAP (complex web timelines)

```js
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)

// Timeline
const tl = gsap.timeline({ defaults: { ease: 'power2.out', duration: 0.4 } })
tl.from('.hero-title', { y: 40, opacity: 0 })
  .from('.hero-sub', { y: 20, opacity: 0 }, '-=0.2')
  .from('.hero-cta', { scale: 0.9, opacity: 0 }, '-=0.1')

// Stagger
gsap.from('.card', { y: 30, opacity: 0, stagger: 0.08, ease: 'power2.out' })

// ScrollTrigger
gsap.from('.section', {
  opacity: 0, y: 60,
  scrollTrigger: { trigger: '.section', start: 'top 80%', end: 'top 40%', scrub: 1 }
})

// Text split
gsap.from(chars, { opacity: 0, y: '100%', stagger: 0.03, ease: 'power3.out' })
```

### Reanimated 3 (Expo / React Native)

```tsx
import Animated, { useSharedValue, useAnimatedStyle,
  withSpring, withTiming, FadeIn, FadeOut } from 'react-native-reanimated'

// Shared value → animated style
const scale = useSharedValue(1)
const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }))
const onPress = () => { scale.value = withSpring(0.95, { damping: 15 }) }

// Layout entry/exit
<Animated.View entering={FadeIn.duration(300)} exiting={FadeOut.duration(200)}>

// Gesture-driven
const translateX = useSharedValue(0)
const gestureHandler = useAnimatedGestureHandler({
  onActive: (e) => { translateX.value = e.translationX },
  onEnd: () => { translateX.value = withSpring(0) }
})
```

### Moti (Expo — simpler Reanimated 3 wrapper)

```tsx
import { MotiView, MotiText } from 'moti'

<MotiView from={{ opacity: 0, translateY: 10 }} animate={{ opacity: 1, translateY: 0 }}
  transition={{ type: 'timing', duration: 300 }} />
```

### WebGL / GLSL (r3f or raw canvas)

Key patterns: per-frame grain via `uTime`, simplex noise for organic movement, `mix()` for smooth transitions, `smoothstep()` for edge control.

```glsl
// Grain overlay (fragment)
float grain(vec2 uv, float t) {
  return fract(sin(dot(uv * 1000.0 + t, vec2(12.9898, 78.233))) * 43758.5453);
}
void main() {
  float g = grain(vUv, uTime * 0.5) * 0.08;
  gl_FragColor = vec4(vec3(g), 1.0);
}
```

---

## Micro-Interaction Patterns

| Pattern | Best approach | Key values |
|---------|--------------|----------|
| Button press feedback | CSS `active` + scale | `scale(0.96)`, 100ms ease-out |
| Hover lift | CSS transition | `translateY(-2px)` + shadow, 150ms |
| Toggle switch | Framer Motion / CSS | spring or 200ms ease-in-out |
| Modal appear | Framer Motion AnimatePresence | fade + scale(0.96→1), 250ms |
| List item stagger | Framer Motion / GSAP | 30–50ms per item, ease-out |
| Skeleton shimmer | CSS `@keyframes` | gradient sweep, 1.5s linear infinite |
| Page transition | Framer Motion / GSAP | fade + slight Y shift, 300ms |
| Pull-to-refresh | Reanimated 3 | spring, clamp displacement |
| Scroll reveal | Scroll-driven CSS or ScrollTrigger | threshold 20–30% from bottom |
| Number count-up | GSAP `gsap.to(obj, { val })` | ease-out, duration matched to magnitude |
| Text character reveal | GSAP SplitText or manual split | stagger 0.02–0.04s, power3.out |
| Ink/grain texture | GLSL fragment shader | `uTime`-driven noise, low alpha overlay |

---

## Performance Checklist (always verify)

- [ ] Only `transform` + `opacity` being animated (not layout properties)
- [ ] `will-change: transform` added only on elements that animate frequently (remove after if possible)
- [ ] No JS animation on every scroll tick without `requestAnimationFrame` or ScrollTrigger
- [ ] Reanimated worklets run on the UI thread (no `.value` access in render — only in `useAnimatedStyle`)
- [ ] GLSL uniforms updated via `useFrame` ref, not React state
- [ ] Lottie/Rive assets are compressed and not blocking the main thread

---

## Kinetic Typography & Generative ASCII (Kinetic Mode)

Activate this mode when the user mentions kinetic text, ASCII art, character animation, scramble effects, text particles, generative typography, or wants text that behaves like a material rather than a label.

### Vocabulary

| Term | Meaning |
|------|--------|
| **Character split** | Breaking a string into individually animatable `<span>` elements per char, word, or line |
| **Brightness ramp** | A string of ASCII chars ordered by visual density, used to map pixel brightness → glyph (e.g. `" .:-=+*#%@"`) |
| **Scramble** | Randomizing characters before they resolve to final text — creates a decode/reveal feel |
| **Typewriter** | Sequential character reveal, left to right, with optional cursor blink |
| **Kinetic** | Text whose individual characters move with physics, noise, or gesture |
| **Particle text** | Characters or points distributed along the outline of letterforms, then animated as a particle system |
| **SDF text** | Signed Distance Field — encodes glyph shapes in a texture; allows sharp rendering at any scale and GPU-side distortion |
| **Flow field** | A grid of vectors (usually noise-driven) that steers particles or characters as they move |
| **ASCII post-processing** | Rendering a scene normally, then mapping each block of pixels to an ASCII character in a post-processing pass |
| **Noise field ASCII** | Using simplex/Perlin noise to drive brightness → char mapping across a grid, animated over time |
| **Character rain** | Columns of falling random characters — each column advances independently, trails fade with low-alpha background |
| **Morph** | Transitioning between two text strings by interpolating character positions (requires same char count or padding) |
| **Glyph outline** | The vector path defining a character's shape — used as source geometry for particle or stroke animation |
| **Text-to-points** | Sampling points along a glyph outline for use as particle targets (p5.js `textToPoints()`, opentype.js) |
| **Variable font axis** | Animatable font axes like `wght`, `wdth`, `slnt` — enables smooth morphing between font weights/widths in CSS |

---

### ASCII: The Core Algorithm

Everything ASCII starts here — map pixel brightness to a character:

```js
const RAMP = ' .\'`^",:;Il!i><~+_-?][}{1)(|/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$'
// denser string = finer gradation; shorter = more graphic/abstract

function brightnessToChar(r, g, b, ramp = RAMP) {
  const brightness = (r * 0.299 + g * 0.587 + b * 0.114) / 255
  return ramp[Math.floor(brightness * (ramp.length - 1))]
}
```

**DOM-based renderer** (canvas → `<pre>`) — works anywhere, great for generative pieces:

```js
function renderAscii(source, pre, cols = 80) {
  const offscreen = document.createElement('canvas')
  const ctx = offscreen.getContext('2d')
  const aspect = source.videoHeight / source.videoWidth || 1
  const rows = Math.floor(cols * aspect * 0.45) // chars are ~2× taller than wide
  offscreen.width = cols
  offscreen.height = rows
  ctx.drawImage(source, 0, 0, cols, rows)
  const { data } = ctx.getImageData(0, 0, cols, rows)
  let out = ''
  for (let i = 0; i < data.length; i += 4) {
    out += brightnessToChar(data[i], data[i+1], data[i+2])
    if ((i / 4 + 1) % cols === 0) out += '\n'
  }
  pre.textContent = out
}
// call on requestAnimationFrame for live video/canvas sources
```

**Noise-field ASCII** — no source image needed, fully generative:

```js
import { createNoise2D } from 'simplex-noise'
const noise2D = createNoise2D()

function drawNoiseAscii(pre, cols, rows, t) {
  let out = ''
  for (let y = 0; y < rows; y++) {
    for (let x = 0; x < cols; x++) {
      const n = (noise2D(x * 0.06 + t * 0.3, y * 0.12) + 1) / 2 // 0–1
      out += RAMP[Math.floor(n * (RAMP.length - 1))]
    }
    out += '\n'
  }
  pre.textContent = out
}
```

**Three.js AsciiEffect** — post-process a whole 3D scene:

```js
import { AsciiEffect } from 'three/addons/effects/AsciiEffect.js'

const effect = new AsciiEffect(renderer, ' .:-=+*#%@', { invert: true })
effect.setSize(window.innerWidth, window.innerHeight)
effect.domElement.style.color = 'white'
document.body.appendChild(effect.domElement)

function animate() {
  requestAnimationFrame(animate)
  effect.render(scene, camera) // replaces renderer.render()
}
```

**GLSL ASCII in a shader** — encodes char index into a lookup texture, fully GPU-side:

```glsl
// Fragment — sample scene texture, map brightness to char row in a font atlas
uniform sampler2D uScene;
uniform sampler2D uFontAtlas; // each row = one ASCII char rendered at charSize
uniform vec2 uResolution;
uniform float uCharSize; // e.g. 8.0 px

void main() {
  vec2 cell = floor(gl_FragCoord.xy / uCharSize);
  vec2 cellUV = cell * uCharSize / uResolution;
  vec4 color = texture2D(uScene, cellUV);
  float brightness = dot(color.rgb, vec3(0.299, 0.587, 0.114));
  float charIndex = floor(brightness * 9.0); // 10-char ramp
  vec2 subUV = mod(gl_FragCoord.xy, uCharSize) / uCharSize;
  subUV.y = (charIndex + subUV.y) / 10.0;
  gl_FragColor = texture2D(uFontAtlas, subUV);
}
```

---

### Kinetic Typography Patterns

**1. Character split — the foundation**

Manual split (no dependency):
```js
const split = (el) => {
  el.innerHTML = [...el.textContent]
    .map(c => `<span style="display:inline-block">${c === ' ' ? '&nbsp;' : c}</span>`)
    .join('')
  return el.querySelectorAll('span')
}
const chars = split(document.querySelector('h1'))
```

With GSAP SplitText (cleanest, handles ligatures):
```js
const split = new SplitText('h1', { type: 'chars,words,lines' })
gsap.from(split.chars, { y: '110%', opacity: 0, stagger: 0.02, ease: 'power3.out' })
```

**2. Scramble / decode reveal**

```js
const CHARS = '!@#$%^&*01アイウエオ'
function scramble(el, finalText, duration = 1200) {
  const len = finalText.length
  let frame = 0
  const totalFrames = duration / 16
  const id = setInterval(() => {
    el.textContent = [...finalText].map((ch, i) => {
      if (frame / totalFrames > i / len) return ch // resolved
      return CHARS[Math.floor(Math.random() * CHARS.length)]
    }).join('')
    if (++frame >= totalFrames) clearInterval(id)
  }, 16)
}
// Or with GSAP ScrambleText plugin:
gsap.to('h1', { duration: 1.5, scrambleText: { text: 'HELLO', chars: '01!アイ', revealDelay: 0.3 } })
```

**3. Variable font kinetic animation**

```css
@keyframes weightWave {
  0%, 100% { font-variation-settings: 'wght' 100, 'wdth' 75; }
  50%       { font-variation-settings: 'wght' 900, 'wdth' 125; }
}
.word span { animation: weightWave 2s ease-in-out infinite; }
.word span:nth-child(n) { animation-delay: calc(n * 0.1s); }
```

```js
// JS-driven per-char variable font via noise
chars.forEach((span, i) => {
  const n = (noise2D(i * 0.5, t) + 1) / 2
  span.style.fontVariationSettings = `'wght' ${100 + n * 800}`
})
```

**4. Particle text — characters flow to letterform outlines**

```js
// p5.js: textToPoints gives outline sample coords
const pts = font.textToPoints('Y', -60, 20, 200, { sampleFactor: 0.15 })

// Animate particles toward their target point (attractor)
particles.forEach((p, i) => {
  const target = pts[i % pts.length]
  p.vx += (target.x - p.x) * 0.05
  p.vy += (target.y - p.y) * 0.05
  p.vx *= 0.85; p.vy *= 0.85 // damping
  p.x += p.vx; p.y += p.vy
  // draw p as a dot or ASCII char
})
```

**5. Wave / sine motion**

```js
// Per-character sine wave — stagger phase by index
chars.forEach((span, i) => {
  const y = Math.sin(t * 2 + i * 0.4) * 8 // 8px amplitude
  span.style.transform = `translateY(${y}px)`
})
```

**6. Flow field text** — characters drift through a vector field:

```js
// Each char has position + velocity; velocity is set by noise at that position
function flowAngle(x, y, t) {
  return noise2D(x * 0.003, y * 0.003 + t * 0.001) * Math.PI * 4
}
chars.forEach(p => {
  const angle = flowAngle(p.x, p.y, t)
  p.x += Math.cos(angle) * p.speed
  p.y += Math.sin(angle) * p.speed
  // wrap at edges, render char at p.x, p.y
})
```

**7. Character rain (Matrix-style)**

```js
function initRain(canvas) {
  const ctx = canvas.getContext('2d')
  const size = 14
  const cols = Math.floor(canvas.width / size)
  const drops = Array.from({ length: cols }, () => Math.random() * -100)
  const chars = 'アイウエオカキクケコ0123456789ABCDEF'

  return function draw() {
    ctx.fillStyle = 'rgba(0,0,0,0.05)'
    ctx.fillRect(0, 0, canvas.width, canvas.height)
    ctx.fillStyle = '#00ff41'
    ctx.font = `${size}px monospace`
    drops.forEach((y, i) => {
      ctx.fillText(chars[Math.floor(Math.random() * chars.length)], i * size, y * size)
      if (y * size > canvas.height && Math.random() > 0.975) drops[i] = 0
      drops[i] += 1
    })
  }
}
```

---

### Render Targets for Kinetic ASCII

| Render target | Best for | Notes |
|---------------|----------|-------|
| `<pre>` + `textContent` | Generative noise fields, video ASCII | Simple, fast, easy to style with CSS color/mix-blend-mode |
| Canvas 2D `fillText` | Rain, particle text, flow fields | Full control over color per char |
| DOM `<span>` grid | Kinetic per-char animations, CSS transitions | CSS variable font axis animatable |
| WebGL texture atlas | GPU ASCII post-processing, shader-driven | Fastest at scale, hardest to set up |
| SVG `<text>` + GSAP | Path-following text, morphing letterforms | Best for precise control of text-on-path |

### Aesthetic Decisions

- **Ramp density** controls how graphic or photographic the output looks. Short ramps (`" .:#@"`) = high contrast, graphic. Long ramps = photographic, smooth.
- **Invert the ramp** for dark-on-light output: `ramp.split('').reverse().join('')`
- **Color** the ASCII: map hue from the source pixel's color instead of just brightness. Or use a single accent color with `mix-blend-mode: screen` over a dark background.
- **Scale the grid**: fewer columns = more abstract / brutal. More = representational.
- **Monospace is non-negotiable** — proportional fonts break the grid alignment.
- **Char aspect ratio correction**: multiply rows by ~0.45 because monospace chars are roughly twice as tall as wide.

---

## Output Format

When building an animation, always:
1. State which library/approach and why (one sentence)
2. Show the complete, copy-pasteable code (no placeholders)
3. Note the easing choice and timing with a brief rationale
4. Flag any performance consideration if it applies
