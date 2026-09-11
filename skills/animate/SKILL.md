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

7. **Respect `prefers-reduced-motion`.** Always. No exceptions for "it's just a subtle fade." Wrap all non-essential motion. Users who opt out have a reason — vestibular disorders, seizure risk, focus needs.

```css
/* Global safety net — still include per-component overrides */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    scroll-behavior: auto !important;
  }
}
```

Framer Motion — `useReducedMotion()` hook returns `true` when the OS preference is set:
```tsx
import { useReducedMotion } from 'framer-motion'

const prefersReduced = useReducedMotion()
// Swap out the full animation variant for a no-op when true
const variants = prefersReduced
  ? { initial: {}, animate: {}, exit: {} }
  : { initial: { opacity: 0, y: 12 }, animate: { opacity: 1, y: 0 }, exit: { opacity: 0 } }
```

Reanimated 3:
```tsx
import { useReducedMotion } from 'react-native-reanimated'

const shouldReduce = useReducedMotion()
// Skip spring; snap directly to target value
scale.value = shouldReduce ? targetValue : withSpring(targetValue, { damping: 15 })
```

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

## Design-to-Code: Paper.design / Figma Motion Tokens

When Yan shares a Paper.design or Figma spec, extract these motion properties before writing code:

| Token | Where to find it | Maps to |
|-------|-----------------|---------|
| Duration | Prototype tab, transition duration | `duration` in ms |
| Easing | "Smart animate" / custom easing | cubic-bezier string or spring config |
| Delay | Per-layer delay in prototype | `withDelay(ms, ...)` or CSS `animation-delay` |
| Spring | If spring is specified: tension + friction or damping ratio | Reanimated `{ stiffness, damping }` |
| Move by | Translate delta on enter/exit | `translateY` / `translateX` initial value |

**Figma → Reanimated translation:**

| Figma spring | Reanimated equivalent |
|-------------|----------------------|
| Tension 300, Friction 30 | `{ stiffness: 300, damping: 30 }` |
| Damping ratio 0.7, Duration 400ms | `{ damping: 0.7 * 2 * Math.sqrt(stiffness), stiffness }` — or approximate with `{ damping: 18, stiffness: 250 }` |
| "Gentle" preset | `{ damping: 20, stiffness: 200 }` |
| "Bouncy" preset | `{ damping: 8, stiffness: 180 }` |
| "Quick" preset | `{ damping: 25, stiffness: 400 }` |

**JetBrains Mono variable font** — Yan's font of choice; animate weight axis for kinetic type:

```css
/* Pull in the variable font if hosted */
@font-face {
  font-family: 'JetBrains Mono';
  src: url('JetBrainsMono[wght].woff2') format('woff2-variations');
  font-weight: 100 800;
}

/* Animate weight on hover for terminal-aesthetic headers */
.terminal-heading {
  font-family: 'JetBrains Mono', monospace;
  font-variation-settings: 'wght' 400;
  transition: font-variation-settings 200ms ease-out;
}
.terminal-heading:hover {
  font-variation-settings: 'wght' 700;
}
```

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
  withSpring, withTiming, withSequence, withDelay, withRepeat,
  FadeIn, FadeOut, useAnimatedScrollHandler, runOnJS,
  LinearTransition, CurvedTransition } from 'react-native-reanimated'
import { Gesture, GestureDetector } from 'react-native-gesture-handler'

// Shared value → animated style
const scale = useSharedValue(1)
const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }))
const onPress = () => { scale.value = withSpring(0.95, { damping: 15 }) }

// Layout entry/exit
<Animated.View entering={FadeIn.duration(300)} exiting={FadeOut.duration(200)}>

// Sequence: animate through multiple values in order
scale.value = withSequence(
  withTiming(0.94, { duration: 80 }),   // press down
  withSpring(1, { damping: 12 })        // spring back
)

// Delay: wait before starting
opacity.value = withDelay(200, withTiming(1, { duration: 300 }))

// Repeat: loop (positive count, or -1 for infinite)
rotation.value = withRepeat(withTiming(2 * Math.PI, { duration: 1000 }), -1)

// runOnJS: call a JS function from a worklet (e.g. trigger navigation, setState)
const handleSnapEnd = (index: number) => setActiveIndex(index)
const pan = Gesture.Pan().onEnd(() => {
  'worklet'
  runOnJS(handleSnapEnd)(computedIndex)
})

// Scroll handler — track scroll position on the UI thread
const scrollY = useSharedValue(0)
const scrollHandler = useAnimatedScrollHandler({ onScroll: (e) => {
  scrollY.value = e.contentOffset.y
}})
const headerStyle = useAnimatedStyle(() => ({
  opacity: 1 - scrollY.value / 100,
  transform: [{ translateY: -scrollY.value * 0.3 }],
}))
// <Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16}>

// Layout animation (list reorder, add/remove items)
<Animated.View layout={LinearTransition.springify().damping(20)}>

// Gesture-driven (RNGH v2 — useAnimatedGestureHandler is deprecated)
const translateX = useSharedValue(0)
const pan = Gesture.Pan()
  .onUpdate((e) => { translateX.value = e.translationX })
  .onEnd(() => { translateX.value = withSpring(0) })

// In JSX:
<GestureDetector gesture={pan}>
  <Animated.View style={animatedStyle} />
</GestureDetector>

// Simultaneous gestures (pinch + pan)
const composed = Gesture.Simultaneous(Gesture.Pinch(), pan)
```

**Spring presets for common RN contexts:**

| Context | `damping` | `stiffness` | Feel |
|---------|-----------|-------------|------|
| Button tap feedback | 15 | 400 | Snappy, responsive |
| Drawer / bottom sheet | 20 | 200 | Smooth, physical |
| Card flip / expand | 18 | 300 | Confident, not bouncy |
| Playful / game UI | 8 | 180 | Bouncy, fun |
| Modal slide-up | 25 | 250 | Purposeful, settled |

**`interpolate` — map a shared value to an output range (essential for scroll-driven UIs):**

```tsx
import { interpolate, Extrapolation } from 'react-native-reanimated'

// Header fades + slides as user scrolls
const scrollY = useSharedValue(0)
const headerStyle = useAnimatedStyle(() => ({
  opacity: interpolate(scrollY.value, [0, 80], [1, 0], Extrapolation.CLAMP),
  transform: [{ translateY: interpolate(scrollY.value, [0, 80], [0, -20], Extrapolation.CLAMP) }],
}))
// Extrapolation.CLAMP: stays at boundary values outside range — almost always correct
// Extrapolation.EXTEND: keeps extrapolating beyond range (can run to ±∞ — avoid)
// Multi-stop (like a CSS @keyframe): interpolate(t, [0, 0.5, 1], [0, 1, 0], Extrapolation.CLAMP)

// Tab bar scale on scroll (grows when scrolling up, shrinks when scrolling down)
const tabStyle = useAnimatedStyle(() => ({
  transform: [{ scaleY: interpolate(scrollY.value, [-20, 0, 80], [1.05, 1, 0.9], Extrapolation.CLAMP) }],
}))
```

**`useAnimatedReaction` — worklet side-effect when a shared value crosses a threshold:**

```tsx
import { useAnimatedReaction, runOnJS } from 'react-native-reanimated'

useAnimatedReaction(
  () => scrollY.value,                    // selector — runs on UI thread, must be a worklet
  (current, previous) => {
    'worklet'
    if (current > 100 && (previous ?? 0) <= 100) {
      runOnJS(setHeaderCollapsed)(true)   // bridge to JS — all JS calls need runOnJS
    }
    if (current <= 100 && (previous ?? 0) > 100) {
      runOnJS(setHeaderCollapsed)(false)
    }
  }
)
// Use for: threshold triggers, haptics on snap, analytics events, setState from scroll
// Don't use for derived styles — that's useAnimatedStyle
```

**Haptics coordination — pair physical feedback with spring animation completion:**

```tsx
import * as Haptics from 'expo-haptics'

// Fire haptic exactly when spring settles (third arg to withSpring is a completion callback)
const snapToIndex = (index: number) => {
  translateX.value = withSpring(snapPoints[index], { damping: 20 }, (finished) => {
    'worklet'
    if (finished) runOnJS(Haptics.impactAsync)(Haptics.ImpactFeedbackStyle.Medium)
  })
}
// Feedback levels:
// Light  → hover, focus, soft arrival
// Medium → tap, select, card snap, list reorder
// Heavy  → drag release, confirm, destructive action
// Notification.Success / .Warning / .Error → system-level outcomes
```

**Expo Router screen transitions:**

```tsx
// app/_layout.tsx — custom stack animation
import { Stack } from 'expo-router'

<Stack screenOptions={{
  animation: 'slide_from_right',   // default iOS
  // 'slide_from_bottom' for modals
  // 'fade' for tab-like switches
  // 'none' to disable (then drive with Reanimated manually)
  gestureEnabled: true,
  gestureDirection: 'horizontal',
}} />

// Shared element transition (Expo Router v3+ with react-native-reanimated)
// Tag both elements with the same sharedTransitionTag
// Source screen:
<Animated.Image source={item.img} sharedTransitionTag={`image-${item.id}`} />
// Destination screen:
<Animated.Image source={item.img} sharedTransitionTag={`image-${item.id}`} />
// No extra config needed — Reanimated handles the morph automatically
```

### Moti (Expo — simpler Reanimated 3 wrapper)

```tsx
import { MotiView, MotiText } from 'moti'

<MotiView from={{ opacity: 0, translateY: 10 }} animate={{ opacity: 1, translateY: 0 }}
  transition={{ type: 'timing', duration: 300 }} />
```

### Supabase Realtime + Animation

When data arrives from a Supabase realtime subscription, animate the new item in rather than letting it pop. Pattern for React Native:

```tsx
import { useEffect } from 'react'
import { useSharedValue, withSpring, withDelay, useAnimatedStyle } from 'react-native-reanimated'
import { supabase } from '@/lib/supabase'

// Track a list and animate new arrivals
function useLiveItems<T extends { id: string }>(table: string) {
  const [items, setItems] = useState<T[]>([])
  const [newId, setNewId] = useState<string | null>(null)

  useEffect(() => {
    const channel = supabase.channel('realtime:' + table)
      .on('postgres_changes', { event: 'INSERT', schema: 'public', table },
        (payload) => {
          setItems(prev => [payload.new as T, ...prev])
          setNewId((payload.new as T).id)
        })
      .subscribe()
    return () => { supabase.removeChannel(channel) }
  }, [table])

  return { items, newId }
}

// Per-item animated row — fades + slides in only for the newest
function AnimatedRow({ id, isNew }: { id: string; isNew: boolean }) {
  const opacity = useSharedValue(isNew ? 0 : 1)
  const translateY = useSharedValue(isNew ? -12 : 0)
  
  useEffect(() => {
    if (isNew) {
      opacity.value = withDelay(50, withSpring(1))
      translateY.value = withDelay(50, withSpring(0, { damping: 18 }))
    }
  }, [isNew])

  const style = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ translateY: translateY.value }],
  }))
  return <Animated.View style={style}>{/* row content */}</Animated.View>
}
```

Key rule: **only animate the new item** — re-animating the whole list on each subscription event is jarring and looks broken.

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

#### Feedback Loop (Touch Designer style) — trails, echo, displacement

TD's signature aesthetic comes from **ping-pong render targets**: each frame reads the previous frame's output as a texture input, feeding it back with slight decay or distortion. This creates trails, smear/echo effects, and organic displacement.

**Three.js / r3f — ping-pong setup:**

```js
import * as THREE from 'three'

// Two render targets — swap each frame
const rtA = new THREE.WebGLRenderTarget(width, height)
const rtB = new THREE.WebGLRenderTarget(width, height)
let [read, write] = [rtA, rtB]

// Feedback material reads the previous frame
const feedbackMaterial = new THREE.ShaderMaterial({
  uniforms: {
    uPrev:  { value: read.texture },   // last frame
    uInput: { value: sourceTex },       // live webcam or scene
    uDecay: { value: 0.96 },            // <1 fades; 1.0 = infinite trail
    uDisplace: { value: 0.003 },        // UV shift per frame
    uTime:  { value: 0 },
  },
  vertexShader: /* glsl */`
    varying vec2 vUv;
    void main() { vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }
  `,
  fragmentShader: /* glsl */`
    uniform sampler2D uPrev;
    uniform sampler2D uInput;
    uniform float uDecay;
    uniform float uDisplace;
    uniform float uTime;
    varying vec2 vUv;

    vec2 noiseDisplace(vec2 uv, float t) {
      float nx = fract(sin(dot(uv + t * 0.1, vec2(12.98, 78.23))) * 43758.5);
      float ny = fract(sin(dot(uv + t * 0.1, vec2(93.98, 17.85))) * 43758.5);
      return vec2(nx, ny) * 2.0 - 1.0;
    }

    void main() {
      vec2 displaced = vUv + noiseDisplace(vUv, uTime) * uDisplace;
      vec4 prev  = texture2D(uPrev, displaced) * uDecay;
      vec4 live  = texture2D(uInput, vUv);
      gl_FragColor = max(prev, live * 0.8); // live source bleeds through
    }
  `,
})

// In your render loop:
function renderFeedback(renderer, scene, camera) {
  feedbackMaterial.uniforms.uPrev.value = read.texture
  feedbackMaterial.uniforms.uTime.value += 0.016

  renderer.setRenderTarget(write)
  renderer.render(feedbackScene, camera) // feedbackScene uses feedbackMaterial on a plane
  renderer.setRenderTarget(null)

  ;[read, write] = [write, read] // swap — next frame reads what we just wrote
}
```

**Key controls:**

| Uniform | Effect |
|---------|--------|
| `uDecay: 0.96` | Trails fade slowly — lower = faster fade |
| `uDecay: 1.0` | Infinite accumulation — no fade |
| `uDisplace: 0.003` | Subtle drift/smear — higher = more distortion |
| `uDisplace: 0.0` | Clean echo with no spatial drift |
| `max(prev, live)` | Additive brightness (glow buildup) |
| `mix(prev, live, 0.15)` | Smooth blend — softer transition |

**Reaction-diffusion (Gray-Scott)** — organic cellular/coral patterns, fully generative:

```glsl
// Ping-pong fragment — Gray-Scott reaction-diffusion
uniform sampler2D uState; // previous RD state (r=chemical A, g=chemical B)
uniform vec2 uResolution;

// GS parameters — tune for different patterns:
// Coral: f=0.0545, k=0.062 | Spots: f=0.025, k=0.06 | Worms: f=0.078, k=0.061
uniform float uFeed;  // f: feed rate of A
uniform float uKill;  // k: kill rate of B
const float dA = 1.0, dB = 0.5;

vec4 lap(vec2 uv) { // Laplacian (discrete diffusion)
  vec2 px = 1.0 / uResolution;
  return -4.0 * texture2D(uState, uv)
    + texture2D(uState, uv + vec2(px.x, 0)) + texture2D(uState, uv - vec2(px.x, 0))
    + texture2D(uState, uv + vec2(0, px.y)) + texture2D(uState, uv - vec2(0, px.y));
}

void main() {
  vec2 uv = gl_FragCoord.xy / uResolution;
  vec4 state = texture2D(uState, uv);
  float a = state.r, b = state.g;
  float reaction = a * b * b;
  float na = a + (dA * lap(uv).r - reaction + uFeed * (1.0 - a)) * 0.5;
  float nb = b + (dB * lap(uv).g + reaction - (uKill + uFeed) * b) * 0.5;
  gl_FragColor = vec4(clamp(na, 0.0, 1.0), clamp(nb, 0.0, 1.0), 0.0, 1.0);
}
```

**UV displacement from noise** — distort any texture (webcam, image, scene) without a ping-pong:

```glsl
uniform sampler2D uTexture;
uniform float uTime;
uniform float uStrength; // 0.02–0.08 for subtle; 0.1+ for glitchy

vec2 warpUV(vec2 uv, float t) {
  float nx = sin(uv.y * 8.0 + t) * cos(uv.x * 3.0 + t * 0.7);
  float ny = cos(uv.x * 6.0 + t * 1.3) * sin(uv.y * 4.0 - t * 0.5);
  return uv + vec2(nx, ny) * uStrength;
}

void main() {
  gl_FragColor = texture2D(uTexture, warpUV(vUv, uTime));
}
```

**r3f / React integration pattern:**

```tsx
import { useFBO, useFrame } from '@react-three/fiber'
import { useRef } from 'react'

function FeedbackEffect() {
  const targets = [useFBO(), useFBO()]
  const idx = useRef(0)

  useFrame(({ gl, scene, camera, clock }) => {
    const read = targets[idx.current]
    const write = targets[1 - idx.current]
    feedbackMat.uniforms.uPrev.value = read.texture
    feedbackMat.uniforms.uTime.value = clock.elapsedTime
    gl.setRenderTarget(write)
    gl.render(scene, camera)
    gl.setRenderTarget(null)
    idx.current = 1 - idx.current
  })

  return <mesh material={feedbackMat}><planeGeometry args={[2, 2]} /></mesh>
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
| Loading progress (determinate) | CSS `scaleX` or Reanimated | `withTiming(pct, { duration: 300, easing: Easing.out(Easing.quad) })` |
| Loading progress (indeterminate) | CSS `@keyframes` | sweeping gradient, 1.2s ease-in-out infinite |
| Glow / aura pulse | CSS animation | `box-shadow` pulse on `@keyframes`, 2–4s ease-in-out infinite |

**Loading progress bar patterns:**

```css
/* Indeterminate — no known completion time */
@keyframes sweep {
  0%   { transform: translateX(-100%) scaleX(0.4); }
  50%  { transform: translateX(0%)    scaleX(0.8); }
  100% { transform: translateX(100%)  scaleX(0.4); }
}
.progress-track { overflow: hidden; height: 3px; background: rgba(255,255,255,0.12); }
.progress-bar   { height: 100%; background: currentColor;
                  animation: sweep 1.2s ease-in-out infinite; }
```

```tsx
// Determinate (React) — Framer Motion width
import { motion } from 'framer-motion'
<div className="overflow-hidden h-[3px] bg-white/10 rounded-full">
  <motion.div className="h-full bg-white rounded-full"
    initial={{ width: '0%' }}
    animate={{ width: `${progress}%` }}
    transition={{ duration: 0.3, ease: [0.25, 1, 0.5, 1] }} />
</div>

// Determinate (React Native / Reanimated)
import { Easing } from 'react-native-reanimated'
const width = useSharedValue(0)
const setProgress = (pct: number) => {
  width.value = withTiming(pct, { duration: 300, easing: Easing.out(Easing.quad) })
}
const barStyle = useAnimatedStyle(() => ({ width: `${width.value}%` as any }))
```

**Glow / aura pulse (CSS — phosphor aesthetic):**

```css
@keyframes aura {
  0%, 100% { box-shadow: 0 0 12px 4px var(--glow-color, rgba(120,220,180,0.4)); }
  50%       { box-shadow: 0 0 28px 10px var(--glow-color, rgba(120,220,180,0.7)); }
}
.glowing { animation: aura 3s ease-in-out infinite; }

/* Dynamic color per element — set --glow-color via JS */
el.style.setProperty('--glow-color', `rgba(${r},${g},${b},0.5)`)
```

---

## Scroll-Snap Animation Coordination

Activate when the user describes full-viewport, one-section-per-screen layouts — quote viewers, presentation slides, product showcases, photo galleries. Each snap point is a "scene" and animations should feel cinematic: enter on snap, hold, exit on next snap.

### The pattern: Intersection Observer + scroll-snap

CSS handles snap; JS drives per-scene animation state via `IntersectionObserver`:

```css
.scroll-container {
  height: 100dvh;
  overflow-y: scroll;
  scroll-snap-type: y mandatory;
}
.scene {
  height: 100dvh;
  scroll-snap-align: start;
}
```

```js
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      entry.target.dataset.active = entry.isIntersecting ? 'true' : 'false'
      // trigger enter/exit CSS class or imperative animation
      if (entry.isIntersecting) entry.target.classList.add('in-view')
      else entry.target.classList.remove('in-view')
    })
  },
  { threshold: 0.8 } // 80% visible = "snapped"
)
document.querySelectorAll('.scene').forEach((s) => observer.observe(s))
```

```css
/* Scene content animates in when snapped */
.scene .content { opacity: 0; transform: translateY(16px); transition: opacity 400ms ease-out, transform 400ms ease-out; }
.scene.in-view .content { opacity: 1; transform: translateY(0); }
```

### React pattern (Framer Motion + scroll-snap)

```tsx
import { motion, useInView } from 'framer-motion'
import { useRef } from 'react'

function Scene({ children }: { children: React.ReactNode }) {
  const ref = useRef<HTMLDivElement>(null)
  const inView = useInView(ref, { amount: 0.8 }) // fires at 80% visibility

  return (
    <div ref={ref} className="h-dvh snap-start flex items-center justify-center">
      <motion.div
        initial={{ opacity: 0, y: 16 }}
        animate={inView ? { opacity: 1, y: 0 } : { opacity: 0, y: 16 }}
        transition={{ duration: 0.4, ease: 'easeOut' }}
      >
        {children}
      </motion.div>
    </div>
  )
}
```

### Staggered content inside a snap scene

When the snapped section has multiple elements (title, subtitle, CTA), stagger them:

```tsx
const container = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.08, delayChildren: 0.1 } },
}
const item = {
  hidden: { opacity: 0, y: 12 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.35, ease: 'easeOut' } },
}

<motion.div variants={container} initial="hidden" animate={inView ? 'visible' : 'hidden'}>
  <motion.h2 variants={item}>Quote text</motion.h2>
  <motion.p variants={item}>— Author, Book</motion.p>
</motion.div>
```

### Key rules for scroll-snap animation

- **Don't re-animate on snap-back** — detect direction if needed; or only animate `in-view` state, let exit be instant
- **`scroll-snap-stop: always`** on critical content prevents fast-scroll skip
- **`height: 100dvh`** not `100vh` — accounts for mobile browser chrome (Safari bottom bar)
- **Avoid layout animation inside snap scenes** — `motion.div layout` causes height recalculation which fights snap
- **Test on mobile** — scroll-snap inertia on iOS differs from desktop; `IntersectionObserver` timing can be off by one frame on fast flicks

---

## View Transitions API

Activate when: page-to-page navigation in Next.js / Astro / vanilla MPA, shared-element transitions between routes, seamless hero image or card → detail view transitions.

The View Transitions API is natively supported in all modern browsers (Chrome 111+, Safari 18+, Firefox 130+). No library required.

### Vanilla — wrapping a route change

```js
// Wrap any DOM mutation in a transition
async function navigate(url) {
  if (!document.startViewTransition) {
    // fallback: navigate without transition
    window.location.href = url
    return
  }
  const transition = document.startViewTransition(async () => {
    // update the DOM here — fetch new content, swap innerHTML, etc.
    const html = await fetch(url).then(r => r.text())
    document.body.innerHTML = parseHTML(html).body.innerHTML
    history.pushState({}, '', url)
  })
  await transition.finished
}
```

```css
/* Default cross-fade — override per element */
::view-transition-old(root) { animation: 200ms ease-out both fade-out; }
::view-transition-new(root) { animation: 300ms ease-out both fade-in; }

@keyframes fade-out { to { opacity: 0 } }
@keyframes fade-in  { from { opacity: 0 } }
```

### Shared element transition (card → detail)

Name the element in both pages with `view-transition-name`. The browser morphs between them automatically.

```css
/* On the list page — give the card a unique name */
.card[data-id="42"] { view-transition-name: card-42; }

/* On the detail page — same name on the hero */
.hero { view-transition-name: card-42; }
```

```css
/* Control the morphing animation */
::view-transition-group(card-42) {
  animation-duration: 400ms;
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
```

Dynamic name assignment via JS (for lists where name is data-driven):
```js
el.style.viewTransitionName = `card-${id}` // set before transition starts
// clear after transition to avoid name collisions
transition.finished.then(() => { el.style.viewTransitionName = '' })
```

### Next.js 15 App Router

Next.js 15 doesn't enable View Transitions natively, but you can wrap `router.push()`:

```tsx
'use client'
import { useRouter } from 'next/navigation'

export function useViewTransitionRouter() {
  const router = useRouter()
  
  const push = (href: string) => {
    if (!document.startViewTransition) {
      router.push(href)
      return
    }
    document.startViewTransition(() => {
      router.push(href)
    })
  }
  
  return { push }
}
```

Usage: `const { push } = useViewTransitionRouter()` → `<button onClick={() => push('/quote/42')}>`.

### When to prefer View Transitions over Framer Motion

| Situation | Reach for |
|-----------|----------|
| MPA / hard navigation, shared element morph | View Transitions API |
| SPA with complex state-driven animation | Framer Motion `layoutId` |
| Mixed SPA + server components (Next.js 15) | View Transitions wrapper around `router.push` |
| Animating within the same route | Framer Motion |

### `prefers-reduced-motion` with View Transitions

```css
@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(root), ::view-transition-new(root) {
    animation: none;
  }
}
```

---

## Audio-Reactive Animation

Activate when: "audio reactive", "sound visualizer", "beat-driven", "amplitude", "frequency", or any animation that responds to microphone or audio playback.

### Web — WebAudio API

```js
async function initAudio() {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
  const ctx = new AudioContext()
  const source = ctx.createMediaStreamSource(stream)
  const analyser = ctx.createAnalyser()
  analyser.fftSize = 256 // 128 frequency bins; lower = coarser but faster
  source.connect(analyser)

  const data = new Uint8Array(analyser.frequencyBinCount)

  function tick() {
    analyser.getByteFrequencyData(data) // 0–255 per bin
    const bass   = avg(data.slice(0, 4))   / 255 // sub-bass ~0–200hz
    const mid    = avg(data.slice(4, 32))  / 255 // mids
    const treble = avg(data.slice(32, 64)) / 255 // highs
    // use bass/mid/treble as 0–1 scalars to drive transforms, opacity, etc.
    requestAnimationFrame(tick)
  }
  tick()
}

const avg = (arr) => arr.reduce((a, b) => a + b, 0) / arr.length
```

**Driving CSS/DOM from audio:**
```js
// Scale an element with bass
el.style.transform = `scale(${1 + bass * 0.4})`

// Drive GLSL uniform
uniforms.uBass.value = bass

// Drive Framer Motion with `useMotionValue` + `set()`
bassMotion.set(bass)
```

**Driving a Three.js scene:**
```js
// In useFrame or rAF:
analyser.getByteFrequencyData(data)
mesh.scale.setScalar(1 + avg(data.slice(0, 8)) / 255 * 2)
material.uniforms.uBass.value = avg(data.slice(0, 4)) / 255
```

### React Native — Expo AV metering

Full FFT isn't available natively in RN. Use amplitude metering for beat-driven effects:

```tsx
import { Audio } from 'expo-av'

const { recording } = await Audio.Recording.createAsync(
  { ...Audio.RecordingOptionsPresets.HIGH_QUALITY, isMeteringEnabled: true }
)

recording.setOnRecordingStatusUpdate((status) => {
  if (!status.metering) return
  // metering is in dBFS, roughly -160 (silence) to 0 (max)
  const level = Math.max(0, (status.metering + 60) / 60) // normalize to 0–1
  amplitude.value = withSpring(level, { damping: 10 })
})
```

### Aesthetic principles for audio-reactive work

- **Lag/smoothing**: raw FFT data is jittery — smooth with `lerp(prev, current, 0.15)` or exponential moving average. Instant response feels broken.
- **Non-linear mapping**: `Math.pow(bass, 2)` makes quiet moments calm and loud moments explosive. Linear is boring.
- **Frequency-to-visual mapping**: bass → scale/position, mids → color/brightness, treble → detail/texture — this maps naturally to how humans perceive music.
- **Avoid full-screen flashing** — it's nauseating and inaccessible. Scale, position, and opacity are safer than flipping background colors.

---

## Real-Time Camera / Canvas Effects

Activate when: "camera filter", "greenscreen", "pixelate", "photobooth", "live video effect", "canvas from webcam".

```js
// Setup: stream webcam into a <video>, render to canvas each frame
const video = document.querySelector('video')
const canvas = document.querySelector('canvas')
const ctx = canvas.getContext('2d')
navigator.mediaDevices.getUserMedia({ video: true })
  .then(stream => { video.srcObject = stream; video.play() })

function tick() {
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height)
  const frame = ctx.getImageData(0, 0, canvas.width, canvas.height)
  applyEffect(frame)
  ctx.putImageData(frame, 0, 0)
  requestAnimationFrame(tick)
}
```

**Pixelation:**
```js
function pixelate(ctx, blockSize = 10) {
  const { width, height } = ctx.canvas
  ctx.imageSmoothingEnabled = false
  const tmpCanvas = document.createElement('canvas')
  tmpCanvas.width = width / blockSize
  tmpCanvas.height = height / blockSize
  const tmp = tmpCanvas.getContext('2d')
  tmp.drawImage(ctx.canvas, 0, 0, tmpCanvas.width, tmpCanvas.height)
  ctx.drawImage(tmpCanvas, 0, 0, width, height)
}
```

**Chroma key (greenscreen) — CPU-side:**
```js
function chromaKey(frame, keyR, keyG, keyB, threshold = 80) {
  const d = frame.data
  for (let i = 0; i < d.length; i += 4) {
    const dist = Math.sqrt(
      (d[i] - keyR) ** 2 + (d[i+1] - keyG) ** 2 + (d[i+2] - keyB) ** 2
    )
    if (dist < threshold) d[i+3] = 0 // set alpha to 0 (transparent)
  }
}
// Call after ctx.getImageData, before putImageData
// Layer over background image using two stacked <canvas> elements
```

**GLSL chromakey (GPU — better performance):**
```glsl
uniform sampler2D uVideo;
uniform vec3 uKeyColor;
uniform float uThreshold;

void main() {
  vec4 texel = texture2D(uVideo, vUv);
  float dist = length(texel.rgb - uKeyColor);
  float alpha = step(uThreshold, dist);
  gl_FragColor = vec4(texel.rgb, alpha);
}
```

**Speech-to-visible-text (Web Speech API):**
```js
const recognition = new webkitSpeechRecognition()
recognition.continuous = true
recognition.interimResults = true
recognition.onresult = (e) => {
  const transcript = [...e.results].map(r => r[0].transcript).join(' ')
  overlayEl.textContent = transcript
  // animate in with character split + scramble for a live-decode effect
}
recognition.start()
```

---

## Performance Checklist (always verify)

- [ ] Only `transform` + `opacity` being animated (not layout properties)
- [ ] `will-change: transform` added only on elements that animate frequently (remove after if possible)
- [ ] No JS animation on every scroll tick without `requestAnimationFrame` or ScrollTrigger
- [ ] Reanimated worklets run on the UI thread (no `.value` access in render — only in `useAnimatedStyle`)
- [ ] GLSL uniforms updated via `useFrame` ref, not React state
- [ ] Lottie/Rive assets are compressed and not blocking the main thread
- [ ] `prefers-reduced-motion` respected — non-essential animations disabled or instant for users who've opted out

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
