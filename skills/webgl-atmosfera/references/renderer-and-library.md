---
title: Renderer and library for a full-screen shader hero
summary: Why the default renderer is a vendored raw WebGL2 helper rather than a library, when OGL or twgl earn their bytes, why WebGPU is out for a single decorative draw call, and the copy-ready helper itself.
last_updated: 2026-06-18
applies_to: Astro 6 plus Cloudflare; WebGL2; a single full-screen fragment-shader pass
---

# Renderer and Library for a Full-Screen Shader Hero

> A hero atmosphere is one full-screen quad running one fragment shader. That is the smallest workload in graphics, so the renderer decision is about defensibility and bytes, not features.

## Contents

- The decision in one table
- Why raw WebGL2 is the default
- When to reach for OGL
- When to reach for twgl
- Why WebGPU is out
- The vendored raw WebGL2 helper

## The decision in one table

| Option | Bytes for one shader | Maintenance risk | When to pick |
|---|---|---|---|
| Raw WebGL2 helper (vendored) | Zero library bytes; the helper is first-party code | None — WebGL2 is a frozen browser API, effectively universal | Default. A single decorative pass that will not grow |
| OGL minimal subset (vendored) | A fraction of OGL Core, itself about 8 kB minzipped | Stale — `ogl@1.0.11` is the latest, roughly a year without a publish, README still says alpha; zero runtime dependencies | When a second pass, render target, or texture is likely, and shader-first ergonomics matter |
| twgl `7.0.0` | A few kB | Low — actively maintained, zero dependencies | When you want a thin maintained helper and do not want to carry first-party WebGL code |

Raw WebGL2 is the neutral technical winner for one shader: it is the purest expression of native-first, where the platform is the renderer and no layer is added. OGL and twgl are real alternatives, documented here so a reader with different constraints can choose deliberately.

## Why raw WebGL2 is the default

WebGL2 is Baseline and present in effectively every browser the stack targets. A full-screen fragment-shader pass needs only a context, one buffer holding a viewport-covering triangle, one compiled program, a render loop, and a few uniforms. A library abstracts plumbing that, for this single case, you write once and never touch again. Carrying zero dependency removes supply-chain and abandonment risk entirely, lets the CI JS budget see exact bytes, and ages perfectly because the underlying API does not change. The cost is more setup code, which the vendored helper below absorbs once.

## When to reach for OGL

OGL is pleasant for shader-first work: a `Program` takes vertex and fragment GLSL strings plus a uniforms object, and a `Triangle` primitive is purpose-built for full-screen passes. Reach for it when the hero is likely to grow a second render pass, a render target, or texture sampling, where hand-rolled plumbing starts to cost more than the dependency. If chosen, pin `ogl@1.0.11` exactly with no caret range, and vendor only the Core subset actually used — Renderer, Triangle, Program, Mesh — not Math or Extras. The README weight table lists Core at about 8 kB minzipped; the screen-shader subset is a strict subset of that, and the exact tree-shaken figure is unpublished, so measure it in the build before writing the budget line. Note the persistent alpha label: the API has been stable in practice for a long time despite it, but the label plus the year-long publish gap is the reason to vendor rather than depend.

## When to reach for twgl

twgl reduces WebGL verbosity without wrapping the API in an object model; its canonical example is exactly a full-screen quad with time and resolution uniforms. It is the healthiest of the three on maintenance — current `7.0.0` line, many dependents, zero dependencies. Pick it when you want a maintained helper rather than first-party WebGL code, and the few extra kB are acceptable against the budget.

## Why WebGPU is out

WebGPU is out for this use case on fitness grounds, not maturity. A decorative full-screen quad is a single draw call that uses none of WebGPU's advantages — general-purpose compute, cheap many-object rendering, render bundles, off-main-thread command encoding. OGL is WebGL-only regardless, and WebGL2 is the universal floor. Status is a secondary point and is genuinely contested: industry sources claim WebGPU reached Baseline in January 2026, while MDN as of May 2026 still labels it not Baseline because it does not work in some widely used browsers, and the W3C gpuweb implementation status shows the gaps behind that label — Firefox on Linux and Android still pending, Safari gated to the newest operating systems. The decision does not hinge on this because the workload does not want WebGPU. Keep it only as a review-gate: revisit if the hero ever needs compute-class effects and WebGPU is Baseline by MDN's definition.

## The vendored raw WebGL2 helper

Ship this as first-party code. It creates the context, builds a viewport-covering triangle, compiles the program, clamps device pixel ratio, handles resize, exposes uniforms, and surfaces context-loss hooks. The render loop, visibility pausing, and the scroll uniform are wired by the caller — see `perf-and-scroll-seam.md`. Loss and restore handling is in `fallback-and-a11y.md`.

```js
// atmosphere-gl.js — first-party, zero dependency, WebGL2 only.
export function createAtmosphere(canvas, fragmentSource, { dprCap = 1.5 } = {}) {
  const gl = canvas.getContext('webgl2', {
    antialias: false,
    alpha: true,
    powerPreference: 'low-power',
    failIfMajorPerformanceCaveat: false,
  });
  if (!gl) return null; // caller swaps in the static fallback

  // Full-viewport triangle: three vertices covering clip space, UVs spanning 0..1.
  const vertexSource = `#version 300 es
    in vec2 position;
    out vec2 vUv;
    void main() {
      vUv = position * 0.5 + 0.5;
      gl_Position = vec4(position, 0.0, 1.0);
    }`;
  const program = link(gl, vertexSource, fragmentSource);
  if (!program) return null;

  const buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 3, -1, -1, 3]), gl.STATIC_DRAW);
  const loc = gl.getAttribLocation(program, 'position');
  gl.enableVertexAttribArray(loc);
  gl.vertexAttribPointer(loc, 2, gl.FLOAT, false, 0, 0);

  const u = {
    time: gl.getUniformLocation(program, 'uTime'),
    resolution: gl.getUniformLocation(program, 'uResolution'),
    scroll: gl.getUniformLocation(program, 'uScroll'),
  };

  function resize() {
    const dpr = Math.min(window.devicePixelRatio || 1, dprCap);
    const w = Math.floor(canvas.clientWidth * dpr);
    const h = Math.floor(canvas.clientHeight * dpr);
    if (canvas.width !== w || canvas.height !== h) {
      canvas.width = w;
      canvas.height = h;
      gl.viewport(0, 0, w, h);
    }
  }

  function render(timeSeconds, scroll) {
    resize();
    gl.useProgram(program);
    gl.uniform1f(u.time, timeSeconds);
    gl.uniform2f(u.resolution, canvas.width, canvas.height);
    gl.uniform1f(u.scroll, scroll);
    gl.drawArrays(gl.TRIANGLES, 0, 3);
  }

  return { gl, program, render, resize, canvas };
}

function link(gl, vs, fs) {
  const compile = (type, src) => {
    const s = gl.createShader(type);
    gl.shaderSource(s, src);
    gl.compileShader(s);
    if (!gl.getShaderParameter(s, gl.COMPILE_STATUS)) {
      console.warn(gl.getShaderInfoLog(s));
      return null;
    }
    return s;
  };
  const v = compile(gl.VERTEX_SHADER, vs);
  const f = compile(gl.FRAGMENT_SHADER, fs);
  if (!v || !f) return null;
  const p = gl.createProgram();
  gl.attachShader(p, v);
  gl.attachShader(p, f);
  gl.linkProgram(p);
  if (!gl.getProgramParameter(p, gl.LINK_STATUS)) {
    console.warn(gl.getProgramInfoLog(p));
    return null;
  }
  return p;
}
```

## Limitations

- The helper covers a single program and a single pass. A second pass, a render target, or texture sampling is the signal to switch to OGL rather than extend this.
- The exact minzipped figure for an OGL minimal subset is unpublished; the 8 kB Core ceiling is the defensible bound until measured in the build.
