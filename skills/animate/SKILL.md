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
- **Contemplative / reading contexts** (quote apps, journals, question curation): intentionally use **400–600ms** with `ease-in-out` or `cubic-bezier(0.4, 0, 0.2, 1)` — the slowness creates breathing room and signals "this is not a task app." Don't mistake it for a perf problem. Empty states in these contexts should breathe, not pop.

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

**`@starting-style` — animate elements entering from `display:none` without any JS:**

Supported in Chrome 117+, Safari 17.5+, Firefox 129+. The correct way to animate dialogs, popovers, and conditionally-rendered elements without Framer Motion's AnimatePresence.

```css
/* Animating a <dialog> or popover that goes display:none → display:block */
dialog {
  opacity: 0;
  transform: scale(0.95) translateY(4px);
}

dialog[open] {
  opacity: 1;
  transform: scale(1) translateY(0);
  /* transition-behavior: allow-discrete lets the transition fire even though
     'display' is a discrete property — without it, the enter animation won't play */
  transition: opacity 200ms ease-out, transform 200ms ease-out,
              display 200ms allow-discrete,
              overlay 200ms allow-discrete; /* overlay = backing-layer participation */
}

/* @starting-style sets where the enter transition starts from (before [open] applies) */
@starting-style {
  dialog[open] {
    opacity: 0;
    transform: scale(0.95) translateY(4px);
  }
}
```

```css
/* Same pattern for a CSS popover (Popover API) */
[popover] { opacity: 0; transform: translateY(-4px); }
[popover]:popover-open {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 150ms ease-out, transform 150ms ease-out,
              display 150ms allow-discrete, overlay 150ms allow-discrete;
}
@starting-style {
  [popover]:popover-open { opacity: 0; transform: translateY(-4px); }
}
```

When to use `@starting-style` vs `AnimatePresence`:
- `@starting-style` = CSS-only enter from `display:none`, no JS, works on native `<dialog>` and `popover` API
- `AnimatePresence` = React tree mounts/unmounts, richer sequencing, cross-platform
- If you already have a `<dialog>` or `popover`, reach for `@starting-style` first — it's zero dependency

**`@property` — typed custom properties that CSS can actually interpolate:**

Without `@property`, CSS transitions on `--custom-vars` snap instead of interpolate. With it, any property with a declared type becomes a first-class animatable value — enabling smooth hue shifts, glow pulses, and gradient morphs with zero JS.

```css
/* Declare types so the browser knows how to interpolate */
@property --hue {
  syntax: '<angle>';
  initial-value: 160deg;
  inherits: false;
}
@property --glow-alpha {
  syntax: '<number>';
  initial-value: 0;
  inherits: false;
}

/* These custom properties now animate smoothly in transitions */
.neon {
  --hue: 160deg;
  --glow-alpha: 0;
  color: hsl(var(--hue) 80% 65%);
  box-shadow: 0 0 20px hsla(var(--hue) 80% 65% / var(--glow-alpha));
  transition: --hue 600ms ease-out, --glow-alpha 300ms ease-out;
}
.neon:hover {
  --hue: 280deg;
  --glow-alpha: 0.7;
}

/* Looping gradient animation using @property — no JS, GPU-composited */
@property --gradient-angle {
  syntax: '<angle>';
  initial-value: 0deg;
  inherits: false;
}
@keyframes rotate-gradient {
  to { --gradient-angle: 360deg; }
}
.gradient-ring {
  background: conic-gradient(from var(--gradient-angle), #0ff, #f0f, #ff0, #0ff);
  animation: rotate-gradient 4s linear infinite;
}
```

Support: Chrome 85+, Safari 16.4+, Firefox 128+. For the phosphor/neon aesthetic, this replaces opacity-only glow animations with true color-shift animations.

### Next.js App Router — animation constraints

Every Framer Motion, Reanimated, or motion-hook call requires a client component. In Next.js App Router:

- Mark any file using `motion.*`, `AnimatePresence`, `useAnimate`, `useMotionValue` with `'use client'` at the top — the build will fail or silently break otherwise.
- `AnimatePresence` for **page transitions** must live in a client component that wraps the route segment — put it in a dedicated `<LayoutClient>` component imported by your server `layout.tsx`:

```tsx
// app/layout-client.tsx
'use client'
import { AnimatePresence, motion } from 'framer-motion'
import { usePathname } from 'next/navigation'

export function LayoutClient({ children }: { children: React.ReactNode }) {
  const key = usePathname()
  return (
    <AnimatePresence mode="wait">
      <motion.div key={key} initial={{ opacity: 0 }} animate={{ opacity: 1 }}
        exit={{ opacity: 0 }} transition={{ duration: 0.2 }}>
        {children}
      </motion.div>
    </AnimatePresence>
  )
}

// app/layout.tsx (Server Component — fine to import from here)
import { LayoutClient } from './layout-client'
export default function RootLayout({ children }) {
  return <html><body><LayoutClient>{children}</LayoutClient></body></html>
}
```

- **Prefer View Transitions API** (native, no JS overhead) for cross-route morphs — see the View Transitions section below. Reserve Framer Motion `AnimatePresence` for within-route show/hide.
- **`motion.div layout` + RSC**: if a component uses the `layout` prop and its parent is a Server Component, hydration mismatches can occur. Keep layout-animated trees entirely inside client components.

---

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

// withDecay — physics-based fling after a pan gesture (swipe-to-dismiss, momentum scroll)
import { withDecay } from 'react-native-reanimated'

const translateX = useSharedValue(0)
const fling = Gesture.Pan()
  .onUpdate((e) => { translateX.value = e.translationX })
  .onEnd((e) => {
    translateX.value = withDecay({
      velocity: e.velocityX,        // hand off exact finger velocity
      rubberBandEffect: true,       // elastic bounce at clamp edges
      clamp: [-SCREEN_WIDTH * 0.5, SCREEN_WIDTH * 0.5], // hard limits
    })
  })
// withDecay reads like the object is still moving when your finger lifts — no spring bounce.
// Use it for: swipe-to-dismiss drawers, momentum carousel, physics-based scroll overscroll.
// For snapping to a grid after decay, combine with onEnd checking final position + withSpring.
```

**`useAnimatedKeyboard` — keyboard-aware layouts without KeyboardAvoidingView:**

```tsx
import { useAnimatedKeyboard, useAnimatedStyle, KeyboardState } from 'react-native-reanimated'

function CommentInput() {
  const keyboard = useAnimatedKeyboard()

  // Translate the input up by exactly the keyboard height as it animates in
  const containerStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: -keyboard.height.value }],
  }))

  return (
    <Animated.View style={[styles.inputBar, containerStyle]}>
      <TextInput placeholder="Add a reflection…" />
    </Animated.View>
  )
}
// keyboard.state.value: KeyboardState.OPEN | CLOSING | OPENING | CLOSED
// keyboard.height.value: animated height — already interpolated, just use it
// Works on both iOS and Android; no LayoutAnimation or KeyboardAvoidingView needed.
// Note: requires Reanimated 3.1+ and enableExperimentalWebImplementation for web.
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

**`useDerivedValue` — compute a derived animated value on the UI thread:**

```tsx
import { useDerivedValue } from 'react-native-reanimated'

// Derive a value from shared values — runs on UI thread, returns a read-only shared value
const rotation = useSharedValue(0)
const radians = useDerivedValue(() => (rotation.value * Math.PI) / 180)

// Use in useAnimatedStyle like any shared value
const style = useAnimatedStyle(() => ({
  transform: [{ rotate: `${radians.value}rad` }],
}))

// More practical: compute once, share across multiple components
const scrollY = useSharedValue(0)
const headerOpacity = useDerivedValue(() =>
  Math.min(1, Math.max(0, scrollY.value / 80))
)
const headerScale = useDerivedValue(() =>
  interpolate(scrollY.value, [0, 80], [1, 0.96], Extrapolation.CLAMP)
)
// Pass headerOpacity and headerScale to any child — no repeated interpolation logic
```

Rules:
- `useDerivedValue` = compute a derived *animated value* (read on UI thread). Use when the same derived expression would appear in multiple `useAnimatedStyle` calls.
- `useAnimatedStyle` = map animated values → style object. Can read `useDerivedValue` results directly.
- `useAnimatedReaction` = side effects when a value crosses a threshold (haptics, `runOnJS`).
- Never call `useDerivedValue` just to trigger a JS effect — that produces silent worklet errors.

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

**GLSL glitch effects — chromatic aberration, RGB split, scanlines:**

These are the primary tools for a Touch Designer / photobooth / dystopian aesthetic. GPU-side, zero layout cost.

```glsl
// Chromatic aberration (RGB split) — offset R, G, B channels by different UV amounts
uniform sampler2D uTexture;
uniform float uStrength;  // 0.005–0.02 for subtle; 0.03+ for glitchy
uniform float uTime;
varying vec2 vUv;

void main() {
  // Directional split — subtle circular aberration
  vec2 dir = vUv - 0.5;
  float dist = length(dir);
  vec2 offset = normalize(dir) * uStrength * dist;

  float r = texture2D(uTexture, vUv + offset).r;
  float g = texture2D(uTexture, vUv).g;
  float b = texture2D(uTexture, vUv - offset).b;
  gl_FragColor = vec4(r, g, b, 1.0);
}
```

```glsl
// Digital glitch — horizontal slice displacement driven by noise + time
uniform sampler2D uTexture;
uniform float uTime;
uniform float uIntensity; // 0.0 = off, 1.0 = heavy glitch
varying vec2 vUv;

float rand(float n) { return fract(sin(n) * 43758.5453); }

void main() {
  float sliceY = floor(vUv.y * 40.0) / 40.0;          // quantize into horizontal bands
  float noise  = rand(sliceY + floor(uTime * 12.0));   // changes ~12× per second
  float active = step(0.92, noise) * uIntensity;       // only ~8% of bands glitch at once
  float shift  = (rand(sliceY * 7.3 + uTime) * 2.0 - 1.0) * 0.04 * active;

  vec2 uv = vUv + vec2(shift, 0.0);
  gl_FragColor = texture2D(uTexture, uv);
}
```

```glsl
// Scanlines overlay — dark horizontal bands, TV/CRT look
uniform float uResolutionY;  // canvas height in px
varying vec2 vUv;

void main() {
  float line = mod(vUv.y * uResolutionY, 2.0);  // alternating 1px rows
  float scanline = line < 1.0 ? 0.85 : 1.0;    // darken every other row
  // Multiply over your base color/texture:
  gl_FragColor = vec4(vec3(scanline), 1.0);
  // In practice, mix this into your final composite:
  // gl_FragColor = baseColor * scanline;
}
```

Compositing in r3f (Three.js EffectComposer pattern):
```tsx
// Use @react-three/postprocessing for clean effect stacking
// Or run glitch as a final pass over the feedback render target
// uIntensity can be driven from audio (bass), a Reanimated shared value, or a JS ref
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
| Sticky header on scroll-past | IntersectionObserver + `AnimatePresence` / `interpolate` | appears when hero exits viewport; fade-in 200ms + `-8px` → `0` translateY |
| Number count-up | GSAP `gsap.to(obj, { val })` | ease-out, duration matched to magnitude |
| Text character reveal | GSAP SplitText or manual split | stagger 0.02–0.04s, power3.out |
| Ink/grain texture | GLSL fragment shader | `uTime`-driven noise, low alpha overlay |
| Loading progress (determinate) | CSS `scaleX` or Reanimated | `withTiming(pct, { duration: 300, easing: Easing.out(Easing.quad) })` |
| Loading progress (indeterminate) | CSS `@keyframes` | sweeping gradient, 1.2s ease-in-out infinite |
| Glow / aura pulse | CSS animation | `box-shadow` pulse on `@keyframes`, 2–4s ease-in-out infinite |
| Shake / error feedback | CSS `@keyframes` or Reanimated `withSequence` | alternating `translateX`, ~500ms, 6 keyframes |
| Tab / segment switch | Framer Motion `layoutId` pill or Reanimated `withSpring` on indicator | pill slides under active tab; 200ms spring, not a hard jump |
| Hover reveal card | CSS `position:absolute` + `transform` | scale + opacity, set `transform-origin` to edge nearest trigger |
| Simulated async progress | Framer Motion `useMotionValue` / Reanimated `withTiming` | asymptote to 95%, snap to 100% on complete |
| Optimistic UI state | Framer Motion `AnimatePresence` + local state | animate instantly, rollback in catch |

**Number count-up — animate a displayed number from 0 to target:**

```tsx
// React (Framer Motion useMotionValue + useTransform)
'use client'
import { useEffect } from 'react'
import { useMotionValue, useTransform, animate, motion } from 'framer-motion'

function CountUp({ to, duration = 1.2, decimals = 0 }: { to: number; duration?: number; decimals?: number }) {
  const count = useMotionValue(0)
  const rounded = useTransform(count, (v) => v.toFixed(decimals))

  useEffect(() => {
    const controls = animate(count, to, { duration, ease: [0.25, 1, 0.5, 1] })
    return controls.stop
  }, [to])

  return <motion.span>{rounded}</motion.span>
}
// Usage: <CountUp to={847} duration={1.5} /> → animates "0" → "847"
// For a score out of 10: <CountUp to={7.4} duration={0.9} decimals={1} />
```

```tsx
// React Native (Reanimated 3) — drives a Text via useAnimatedProps
import Animated, { useSharedValue, withTiming, useAnimatedProps, Easing } from 'react-native-reanimated'
import { TextInput } from 'react-native'

const AnimatedTextInput = Animated.createAnimatedComponent(TextInput)

function CountUp({ to, duration = 1200 }: { to: number; duration?: number }) {
  const count = useSharedValue(0)

  useEffect(() => {
    count.value = withTiming(to, { duration, easing: Easing.out(Easing.cubic) })
  }, [to])

  const animatedProps = useAnimatedProps(() => ({
    text: String(Math.round(count.value)),
    defaultValue: '0',
  }))

  return (
    <AnimatedTextInput
      animatedProps={animatedProps}
      editable={false}
      style={{ color: '#fff', fontSize: 48, fontFamily: 'JetBrainsMono' }}
    />
  )
}
```

Duration rule: **match duration to magnitude** — a score out of 10 deserves ~600ms; a large stat like 12,450 deserves 1.5–2s. Don't use the same duration for both.

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

**Simulated async progress — for ops with no real progress signal (like an API call):**

Drive toward 95% asymptotically while the call is running; snap to 100% on completion. Never stall at 100% — brief hold, then reset. This is the right pattern for Motamo's "publishing" flow, Supabase writes, and any server action that takes 2–8s with no incremental signal.

```tsx
// React (Framer Motion useMotionValue)
'use client'
import { useEffect } from 'react'
import { motion, useMotionValue, animate } from 'framer-motion'

function useSimulatedProgress(isRunning: boolean) {
  const progress = useMotionValue(0)

  useEffect(() => {
    let controls: ReturnType<typeof animate> | null = null
    if (isRunning) {
      progress.set(0)
      // Asymptotic: fast start, slows to near-stop before 95% (won't complete in ~8s)
      controls = animate(progress, 0.95, { duration: 8, ease: [0.1, 0.4, 0.6, 0.9] })
    } else if (progress.get() > 0) {
      controls?.stop()
      // Snap to 100%, hold 400ms, then clear
      animate(progress, 1, { duration: 0.2 }).then(() => {
        setTimeout(() => animate(progress, 0, { duration: 0 }), 400)
      })
    }
    return () => controls?.stop()
  }, [isRunning])

  return progress
}

// Usage
function PublishButton({ onPublish }: { onPublish: () => Promise<void> }) {
  const [running, setRunning] = useState(false)
  const progress = useSimulatedProgress(running)

  const handleClick = async () => {
    setRunning(true)
    await onPublish()
    setRunning(false) // triggers snap-to-100 + clear
  }

  return (
    <button onClick={handleClick} disabled={running}>
      <div className="overflow-hidden h-[3px] bg-white/10 rounded-full">
        <motion.div className="h-full bg-white rounded-full origin-left"
          style={{ scaleX: progress }} />
      </div>
      {running ? 'Publishing…' : 'Publish'}
    </button>
  )
}
```

```tsx
// React Native — Reanimated 3 version
import { useSharedValue, withTiming, withSequence, withDelay, Easing } from 'react-native-reanimated'

function useSimulatedProgress(isRunning: boolean) {
  const progress = useSharedValue(0)

  useEffect(() => {
    if (isRunning) {
      progress.value = 0
      // Slow asymptote — won't reach 0.95 before ~8s
      progress.value = withTiming(0.95, { duration: 8000, easing: Easing.out(Easing.cubic) })
    } else if (progress.value > 0) {
      progress.value = withSequence(
        withTiming(1, { duration: 200 }),
        withDelay(400, withTiming(0, { duration: 0 })),
      )
    }
  }, [isRunning])

  return progress
}
```

Key rule: the asymptote target is **95%, not 100%** — never animate to 100% before the call actually completes. Seeing a bar stuck at 100% while still loading is more frustrating than one that slowly crawls toward 95%.

---

**Optimistic UI animation — animate success immediately, roll back on error:**

For dashboard interactions (publish, approve, delete) where you want instant feel without waiting for the server:

```tsx
'use client'
import { useState } from 'react'
import { motion, AnimatePresence } from 'framer-motion'

function PublishableRow({ quote, onPublish }: { quote: Quote; onPublish: (id: string) => Promise<void> }) {
  // Optimistic: immediately show published state
  const [optimisticPublished, setOptimisticPublished] = useState(quote.published)
  const [error, setError] = useState(false)

  const handlePublish = async () => {
    const prev = optimisticPublished
    setOptimisticPublished(true) // immediate — animation fires now
    setError(false)
    try {
      await onPublish(quote.id)
    } catch {
      setOptimisticPublished(prev) // rollback
      setError(true)
    }
  }

  return (
    <motion.div
      animate={{ opacity: optimisticPublished ? 1 : 0.6 }}
      transition={{ duration: 0.2 }}
    >
      <AnimatePresence mode="wait">
        {optimisticPublished ? (
          <motion.span key="pub"
            initial={{ opacity: 0, scale: 0.9 }} animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0 }} transition={{ duration: 0.15 }}
          >PUBLISHED</motion.span>
        ) : (
          <motion.button key="draft" onClick={handlePublish}
            initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
          >PUBLISH</motion.button>
        )}
      </AnimatePresence>
      {error && <span className="text-red-500 text-xs">Failed — try again</span>}
    </motion.div>
  )
}
```

Rollback rule: always store the previous state before the optimistic update — `const prev = currentState` — so you can restore it in the catch block. The error state triggers its own micro-animation (shake, color change, or inline message — not a toast).

---

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

**Shake / error feedback (CSS + Reanimated):**

Use for: wrong password, magic 8-ball reveal, invalid input, game feedback. The rhythm — wide, narrow, wide, settle — reads as physical.

```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  15%       { transform: translateX(-8px) rotate(-1deg); }
  30%       { transform: translateX(8px)  rotate(1deg); }
  45%       { transform: translateX(-6px); }
  60%       { transform: translateX(6px); }
  75%       { transform: translateX(-3px); }
  90%       { transform: translateX(3px); }
}
.shake { animation: shake 0.5s ease-out; }
/* Re-trigger: remove class, force reflow, add back */
el.classList.remove('shake')
void el.offsetWidth // reflow
el.classList.add('shake')
```

```tsx
// Reanimated 3 — shake a React Native element
const shakeX = useSharedValue(0)
const shakeStyle = useAnimatedStyle(() => ({ transform: [{ translateX: shakeX.value }] }))
const shake = () => {
  shakeX.value = withSequence(
    withTiming(-10, { duration: 55 }),
    withTiming( 10, { duration: 55 }),
    withTiming( -8, { duration: 55 }),
    withTiming(  8, { duration: 55 }),
    withTiming( -4, { duration: 55 }),
    withTiming(  0, { duration: 55 }),
  )
}
// Also fire a haptic on the first frame: runOnJS(Haptics.notificationAsync)(Haptics.NotificationFeedbackType.Error)
```

**Tab / segment switch — animated indicator that slides under the active tab:**

```tsx
// React — Framer Motion layoutId pill (web)
import { motion } from 'framer-motion'

const tabs = ['Quotes', 'Books']
const [active, setActive] = useState('Quotes')

<div className="flex gap-1 relative">
  {tabs.map((tab) => (
    <button key={tab} onClick={() => setActive(tab)}
      className="relative px-4 py-1.5 text-sm font-medium text-white/60 transition-colors hover:text-white"
      style={{ color: active === tab ? '#fff' : undefined }}
    >
      {active === tab && (
        <motion.span
          layoutId="tab-pill"  // same id across all tabs — Framer morphs the single element
          className="absolute inset-0 rounded-full bg-white/10"
          transition={{ type: 'spring', stiffness: 400, damping: 30 }}
        />
      )}
      <span className="relative z-10">{tab}</span>
    </button>
  ))}
</div>
```

```tsx
// React Native — Reanimated 3 underline indicator
import { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated'

const TAB_WIDTH = 80
const indicatorX = useSharedValue(0)

const indicatorStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: indicatorX.value }],
}))

const handleTabPress = (index: number) => {
  indicatorX.value = withSpring(index * TAB_WIDTH, { damping: 20, stiffness: 300 })
  setActiveTab(index)
}

// JSX: a <Animated.View style={indicatorStyle}> positioned absolute below the tab row,
// width = TAB_WIDTH, height = 2, backgroundColor = activeColor
```

Key: never toggle visibility of the indicator between tabs — let one indicator *slide* to the active position. A blinking-in indicator looks broken.

---

**Hover reveal card (scale up from trigger edge):**

The pattern for a tooltip/info card that appears 25% larger than its base state — used when the card IS the revealed content, not a separate floating element (Motamo author card pattern). Key: `transform-origin` must point to the edge/corner closest to the trigger so the card grows "toward" the reader, not away from the trigger.

```css
.citation { position: relative; display: inline-block; cursor: default; }

.reveal-card {
  position: absolute;
  bottom: calc(100% + 8px); /* or top / left / right based on available space */
  left: 50%;
  transform: translateX(-50%) scale(0.88);
  transform-origin: bottom center; /* grows upward from the citation line */
  opacity: 0;
  pointer-events: none;
  transition: transform 200ms cubic-bezier(0.34, 1.56, 0.64, 1),
              opacity 180ms ease-out;
  /* slight spring overshoot on scale makes it feel alive */
}

.citation:hover .reveal-card,
.citation:focus-within .reveal-card {
  transform: translateX(-50%) scale(1);
  opacity: 1;
  pointer-events: auto;
}
```

```tsx
// React — Framer Motion hover card (layoutId optional for shared element)
import { motion, AnimatePresence } from 'framer-motion'
import { useState } from 'react'

function CitationCard({ author, bio, photo }: { author: string; bio: string; photo?: string }) {
  const [open, setOpen] = useState(false)
  return (
    <span className="relative inline-block" onMouseEnter={() => setOpen(true)} onMouseLeave={() => setOpen(false)}>
      <span className="cursor-default underline decoration-dotted">{author}</span>
      <AnimatePresence>
        {open && (
          <motion.div
            className="absolute bottom-full left-1/2 -translate-x-1/2 mb-2 w-56 rounded-lg bg-neutral-900 p-3 shadow-xl"
            initial={{ opacity: 0, scale: 0.88, y: 4 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.92, y: 2 }}
            transition={{ type: 'spring', stiffness: 400, damping: 28 }}
            style={{ transformOrigin: 'bottom center' }}
          >
            {photo && <img src={photo} alt={author} className="mb-2 h-12 w-12 rounded-full object-cover" />}
            <p className="text-xs text-neutral-300">{bio}</p>
          </motion.div>
        )}
      </AnimatePresence>
    </span>
  )
}
// Note: on mobile (touch), swap hover for tap-to-toggle — onMouseEnter/Leave won't fire
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

## Scroll-Past Sticky Header

Activate when: a page has a hero element (book cover, profile image, product photo) at the top, and a condensed header should appear only after the user has scrolled past it. Common in book/quote pages, profile pages, product pages.

### Web — IntersectionObserver + Framer Motion AnimatePresence

```tsx
'use client'
import { useEffect, useRef, useState } from 'react'
import { AnimatePresence, motion } from 'framer-motion'

function BookPage({ book, quotes }: { book: Book; quotes: Quote[] }) {
  const heroRef = useRef<HTMLDivElement>(null)
  const [pastHero, setPastHero] = useState(false)

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => setPastHero(!entry.isIntersecting),
      { threshold: 0, rootMargin: '-64px 0px 0px 0px' } // adjust for any fixed nav above
    )
    if (heroRef.current) observer.observe(heroRef.current)
    return () => observer.disconnect()
  }, [])

  return (
    <>
      {/* Sticky header — mounts when hero exits the viewport */}
      <AnimatePresence>
        {pastHero && (
          <motion.header
            className="fixed top-0 inset-x-0 z-50 bg-neutral-950/95 backdrop-blur-sm border-b border-white/8"
            initial={{ opacity: 0, y: -8 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -4 }}
            transition={{ duration: 0.2, ease: 'easeOut' }}
          >
            <div className="flex items-center h-14 px-4 gap-3">
              <span className="text-sm font-medium truncate">{book.title}</span>
            </div>
          </motion.header>
        )}
      </AnimatePresence>

      {/* Hero — observed for exit */}
      <div ref={heroRef}>
        {/* book cover, title, etc. */}
      </div>

      {quotes.map(q => <QuoteRow key={q.id} quote={q} />)}
    </>
  )
}
```

Key: `!entry.isIntersecting` — header appears when the hero is NOT intersecting (scrolled off the top). `rootMargin: '-64px 0px 0px 0px'` accounts for any fixed nav above; match to your layout. Exit animation is subtler than entrance — `-4px` vs `-8px` — because abrupt disappearance reads as a glitch.

### Combining sticky header with a single-quote horizontal scroll

When the sticky header itself should show one quote at a time (e.g. MOTAMO book page), embed a horizontal scroll-snap inside it:

```tsx
// Inside the sticky header motion.header above:
<div className="flex items-center gap-3 h-14 px-4">
  <span className="text-xs text-white/40 shrink-0">{book.title}</span>
  {/* Single-quote carousel — horizontal snap, no visible scrollbar */}
  <div className="flex-1 overflow-x-auto snap-x snap-mandatory scroll-smooth no-scrollbar">
    <div className="flex">
      {quotes.map((q) => (
        <div key={q.id} className="snap-start shrink-0 w-full px-2">
          <p className="text-xs text-white/80 truncate">{q.text}</p>
        </div>
      ))}
    </div>
  </div>
</div>
```

```css
/* Hide scrollbar but keep scrollability */
.no-scrollbar { scrollbar-width: none; }
.no-scrollbar::-webkit-scrollbar { display: none; }
```

Drive the scroll position programmatically from the main page's active snap index, or let the user swipe independently. For a programmatic link between the main scroll and the sticky header's quote, use `scrollIntoView` on the quote element:

```tsx
// When the user snaps to quote i on the main page, advance the header carousel
useEffect(() => {
  headerQuoteRefs.current[activeIndex]?.scrollIntoView({ behavior: 'smooth', inline: 'start', block: 'nearest' })
}, [activeIndex])
```

### React Native — `useAnimatedScrollHandler` + `interpolate`

```tsx
import { useSharedValue, useAnimatedScrollHandler, useAnimatedStyle, interpolate, Extrapolation } from 'react-native-reanimated'
import Animated from 'react-native-reanimated'

const HERO_HEIGHT = 320 // px — height of book cover section

function BookScreen({ book, quotes }: { book: Book; quotes: Quote[] }) {
  const scrollY = useSharedValue(0)
  const scrollHandler = useAnimatedScrollHandler({ onScroll: (e) => { scrollY.value = e.contentOffset.y } })

  const stickyStyle = useAnimatedStyle(() => ({
    opacity: interpolate(scrollY.value, [HERO_HEIGHT - 60, HERO_HEIGHT], [0, 1], Extrapolation.CLAMP),
    transform: [{ translateY: interpolate(scrollY.value, [HERO_HEIGHT - 60, HERO_HEIGHT], [-8, 0], Extrapolation.CLAMP) }],
  }))

  return (
    <>
      <Animated.View style={[styles.stickyHeader, stickyStyle]} pointerEvents="box-none">
        <Text numberOfLines={1} style={styles.stickyTitle}>{book.title}</Text>
      </Animated.View>

      <Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16}>
        <View style={{ height: HERO_HEIGHT }}>{/* book cover */}</View>
        {quotes.map(q => <QuoteRow key={q.id} quote={q} />)}
      </Animated.ScrollView>
    </>
  )
}

const styles = StyleSheet.create({
  stickyHeader: {
    position: 'absolute', top: 0, left: 0, right: 0, zIndex: 50,
    height: 56, paddingHorizontal: 16,
    flexDirection: 'row', alignItems: 'center',
    backgroundColor: 'rgba(10, 10, 10, 0.95)',
  },
  stickyTitle: { fontSize: 14, fontWeight: '600', color: '#fff', flex: 1 },
})
```

Interpolating over the last 60px of the hero (`HERO_HEIGHT - 60` → `HERO_HEIGHT`) ties the opacity + slide to scroll position — it feels physically attached to the hero leaving screen. Never use a hard blink (`pointerEvents` toggle only) — it reads as a flash.

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

**Photobooth capture flash + countdown animation:**

The two animation moments every photobooth needs: a countdown before capture, and a flash on shutter.

```js
// Capture flash — white overlay that appears instantly and fades out fast
// Pure CSS + JS class toggle: no library needed
function triggerFlash(overlayEl) {
  overlayEl.style.opacity = '1'
  // Force reflow so the transition fires from opacity:1, not from wherever it was
  void overlayEl.offsetWidth
  overlayEl.style.transition = 'opacity 400ms ease-out'
  overlayEl.style.opacity = '0'
}
// CSS: .flash-overlay { position:fixed; inset:0; background:#fff; opacity:0; pointer-events:none; z-index:999; }
```

```tsx
// React — Framer Motion flash
import { useAnimate } from 'framer-motion'

function useFlash() {
  const [scope, animate] = useAnimate()
  const flash = async () => {
    await animate(scope.current, { opacity: 1 }, { duration: 0 })      // instant white
    await animate(scope.current, { opacity: 0 }, { duration: 0.4, ease: 'easeOut' })
  }
  return { scope, flash }
}
// <div ref={scope} className="fixed inset-0 bg-white pointer-events-none z-50 opacity-0" />
```

```tsx
// Countdown timer — 3…2…1…GO with scale pop per tick
import { useState, useEffect } from 'react'
import { motion, AnimatePresence } from 'framer-motion'

function Countdown({ from = 3, onComplete }: { from?: number; onComplete: () => void }) {
  const [count, setCount] = useState(from)

  useEffect(() => {
    if (count === 0) { onComplete(); return }
    const t = setTimeout(() => setCount(c => c - 1), 1000)
    return () => clearTimeout(t)
  }, [count])

  return (
    <AnimatePresence mode="popLayout">
      <motion.div
        key={count}
        initial={{ scale: 1.6, opacity: 0 }}
        animate={{ scale: 1,   opacity: 1 }}
        exit={{    scale: 0.6, opacity: 0 }}
        transition={{ type: 'spring', stiffness: 500, damping: 25 }}
        className="text-8xl font-bold text-white tabular-nums"
      >
        {count === 0 ? '📸' : count}
      </motion.div>
    </AnimatePresence>
  )
}
```

React Native countdown uses the same pattern with `MotiView` or `Animated.View` + `withSpring`.

Key: use `mode="popLayout"` not `mode="wait"` — `popLayout` removes the exiting element from layout immediately so the incoming number pops in cleanly without waiting.

**Dominant color extraction from a video/canvas frame — for aura and palette effects:**

Samples a grid of pixels, skips near-black and near-white (background/highlights), averages the rest. Fast enough to run every few frames without a worker.

```js
// Extract dominant color from a canvas context
// Returns [r, g, b] (0–255 each)
function extractDominantColor(ctx, width, height, sampleStep = 12) {
  const { data } = ctx.getImageData(0, 0, width, height)
  let r = 0, g = 0, b = 0, count = 0
  for (let i = 0; i < data.length; i += 4 * sampleStep) {
    const brightness = (data[i] + data[i + 1] + data[i + 2]) / 3
    // Skip near-black (background) and near-white (blown-out highlights)
    if (brightness < 24 || brightness > 230) continue
    r += data[i]; g += data[i + 1]; b += data[i + 2]
    count++
  }
  if (count === 0) return [120, 220, 180] // fallback teal
  return [Math.round(r / count), Math.round(g / count), Math.round(b / count)]
}
```

**Aura pipeline — live video → dominant color → animated CSS glow:**

Pairs the extraction above with the tick loop from the webcam setup. `lerp` smooths color transitions so the glow feels alive rather than flickering:

```js
let currentAura = [120, 220, 180]
let targetAura  = [120, 220, 180]
let frameCount  = 0

function lerpColor([r1, g1, b1], [r2, g2, b2], t) {
  return [
    Math.round(r1 + (r2 - r1) * t),
    Math.round(g1 + (g2 - g1) * t),
    Math.round(b1 + (b2 - b1) * t),
  ]
}

function auraTick(video, canvas, ctx, auraEl) {
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height)

  // Re-sample color every 20 frames (≈3× per second at 60fps) — cheaper than every frame
  if (frameCount % 20 === 0) {
    targetAura = extractDominantColor(ctx, canvas.width, canvas.height)
  }
  frameCount++

  // Ease current color toward target — t=0.04 gives a ~25-frame settle
  currentAura = lerpColor(currentAura, targetAura, 0.04)
  const [r, g, b] = currentAura

  // Push to the glow element — this is the same --glow-color var used in the aura pulse section
  auraEl.style.setProperty('--glow-color', `rgba(${r},${g},${b},0.6)`)

  requestAnimationFrame(() => auraTick(video, canvas, ctx, auraEl))
}

// Start it after getUserMedia resolves
navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
  video.srcObject = stream
  video.play().then(() => auraTick(video, canvas, ctx, auraEl))
})
```

```css
/* The glow element — reads the JS-driven --glow-color CSS variable */
.aura-halo {
  position: absolute;
  inset: -40px;
  border-radius: 50%;
  filter: blur(40px);
  background: var(--glow-color, rgba(120, 220, 180, 0.5));
  transition: background 300ms ease-out; /* smooth even between rAF updates */
  pointer-events: none;
}
```

Key rule: **sample step ≥ 10** — sampling every pixel at 640×480 is 300K iterations per frame. A step of 12 reads ~2 200 pixels and takes < 2ms. Use `filter: blur()` on the glow element, not on the video — blurring the whole camera feed tanks performance.

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
