---
title: Rive state machine inputs, listeners, and triggers on the web
summary: How to drive Rive state-machine inputs from JavaScript, how editor Listeners handle hover and click internally, the native-first scroll wiring, and the event-handling flag.
last_updated: 2026-06-18
applies_to: @rive-app/canvas@2.38.1
---

# Rive state machine inputs, listeners, and triggers on the web

The state machine is the contract between the designer and the developer. The designer names inputs in the editor; the developer reads and sets them at runtime. There are two interaction routes: Listeners authored in the editor for pointer interactions, and inputs driven from JavaScript for everything else.

## Contents

- Input types and the web API
- Editor Listeners for hover and click
- Native-first scroll wiring
- Event handling and OpenUrl
- Common gotchas

## Input types and the web API

A state machine exposes three input types: Number, Boolean, and Trigger. On the web you read and set them on the input objects returned by stateMachineInputs.

```ts
const inputs = rive.stateMachineInputs('Hero');
const hovered = inputs.find((i) => i.name === 'isHovered');   // boolean
const progress = inputs.find((i) => i.name === 'scroll');     // number
const burst = inputs.find((i) => i.name === 'burst');         // trigger

hovered.value = true;     // boolean and number use .value
progress.value = 0.5;
burst.fire();             // triggers use .fire()
```

Each input has name, type, a value getter and setter, and fire for triggers. For inputs on nested artboards use the path variants setBooleanStateAtPath, setNumberStateAtPath, and fireStateAtPath with a path string such as parentArtboard or group/nested. The methods setNumberState, setBooleanState, and fireState are Android and Unity APIs, not the web runtime; on the web, set value or call fire on the input object.

## Editor Listeners for hover and click

Rive Listeners authored in the editor (Pointer Enter, Pointer Exit, Pointer Down, Pointer Up, Pointer Move, Click) are dispatched by the runtime against the canvas automatically. Hover and click that change an input or fire a trigger need no JavaScript wiring; the designer wires them in the editor and the runtime handles pointer events. This is the preferred route for pointer interaction because it keeps the logic in the `.riv`.

The parameters shouldDisableRiveListeners and dispatchPointerExit control this behavior; isTouchScrollEnabled lets page scrolling continue when a touch or drag starts on the canvas on touch devices.

## Native-first scroll wiring

For scroll-driven motion, write a Number input from a passive scroll listener throttled with requestAnimationFrame. No animation library is needed for a single value; set the input and let the state machine interpolate through its blend states.

```ts
let ticking = false;
addEventListener('scroll', () => {
  if (ticking) return;
  ticking = true;
  requestAnimationFrame(() => {
    const max = document.documentElement.scrollHeight - innerHeight;
    progress.value = max > 0 ? scrollY / max : 0;
    ticking = false;
  });
}, { passive: true });
```

Do not pull in GSAP for this. The motion skill owns GSAP; a single Number input is set directly. The passive listener keeps scrolling smooth, and the rAF throttle coalesces writes to one per frame.

## Event handling and OpenUrl

Set automaticallyHandleEvents to false, which is the default, so a Rive Event such as OpenUrl cannot trigger navigation implicitly during the render loop. If the animation needs to act on events, subscribe explicitly and handle them yourself.

```ts
import { EventType } from '@rive-app/canvas';
rive.on(EventType.RiveEvent, (event) => {
  // inspect event.data and act explicitly; do not auto-open URLs
});
```

## Common gotchas

- stateMachineInputs returns undefined until the file is loaded. Query inputs inside the onLoad callback or after riveLoaded is true.
- stateMachines must be supplied at construction or Rive plays the first linear animation and the inputs do not exist.
- Boolean and Number inputs hold state; Trigger inputs are momentary and fire once. Use a Number to track continuous progress, a Boolean for an on or off state, a Trigger for a one-shot transition.
- If autoBind is enabled, data-bound view-model values can update the graphic at runtime; this is the route for dynamic text or color without re-authoring the input set.
