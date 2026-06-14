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

## Output Format

When building an animation, always:
1. State which library/approach and why (one sentence)
2. Show the complete, copy-pasteable code (no placeholders)
3. Note the easing choice and timing with a brief rationale
4. Flag any performance consideration if it applies
