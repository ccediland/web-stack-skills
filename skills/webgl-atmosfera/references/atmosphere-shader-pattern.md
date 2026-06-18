---
title: The atmosphere shader pattern
summary: How a full-screen fragment shader paints an atmosphere from fbm domain-warped noise, a copy-ready reference shader with declared precision and bounded loops, the references to learn it from, and the boundary where a CSS gradient is the better choice than WebGL.
last_updated: 2026-06-18
applies_to: WebGL2 GLSL ES 3.0 fragment shaders; a single full-screen pass
---

# The Atmosphere Shader Pattern

> Every pixel of the hero runs the same small program — the fragment shader — which turns the pixel position and a few uniforms into a color. An atmosphere is that color driven by layered, slowly drifting noise.

## Contents

- The mental model
- Noise, fbm, and domain warping
- The reference shader
- Uniforms the host drives
- The boundary — WebGL versus CSS
- Production caveats for shadertoy-style shaders
- References

## The mental model

A full-screen pass draws one triangle large enough to cover the viewport. The vertex shader does almost nothing; the work is in the fragment shader, which the GPU runs once per pixel, massively in parallel. The fragment shader receives the pixel's normalized coordinate and uniforms — a clock, the resolution, a scroll value — and returns a color. Because the GPU runs all pixels at once, per-pixel noise that would stall a CPU is cheap here. That parallelism is the entire reason WebGL is worth its cost for this effect.

## Noise, fbm, and domain warping

Three ideas compose the look. Noise is a smooth pseudo-random field — simplex or value noise — that varies continuously across the canvas. Fractional Brownian motion (fbm) sums several octaves of that noise at rising frequency and falling amplitude, which adds fine detail on top of broad shapes; keep the octave count low, typically four, because each octave is more work per pixel. Domain warping feeds one noise field into the coordinates of another, bending the pattern into the organic, flowing shapes that read as atmosphere rather than static grain. Animate by adding the time uniform to the noise input so the field drifts.

## The reference shader

This is a domain-warped fbm flow field in GLSL ES 3.0. Precision is declared, the loop is a fixed four octaves, and there is no dynamic branching, so it behaves on mobile GPUs. Replace the two color constants with values derived from the design tokens.

```glsl
#version 300 es
precision highp float;

in vec2 vUv;
out vec4 fragColor;

uniform float uTime;
uniform vec2 uResolution;
uniform float uScroll; // 0..1 progress, driven by the host

// Hash and value noise — compact, no texture lookups.
float hash(vec2 p) {
  p = fract(p * vec2(123.34, 456.21));
  p += dot(p, p + 45.32);
  return fract(p.x * p.y);
}

float noise(vec2 p) {
  vec2 i = floor(p);
  vec2 f = fract(p);
  vec2 u = f * f * (3.0 - 2.0 * f);
  float a = hash(i);
  float b = hash(i + vec2(1.0, 0.0));
  float c = hash(i + vec2(0.0, 1.0));
  float d = hash(i + vec2(1.0, 1.0));
  return mix(mix(a, b, u.x), mix(c, d, u.x), u.y);
}

float fbm(vec2 p) {
  float sum = 0.0;
  float amp = 0.5;
  for (int i = 0; i < 4; i++) { // bounded; do not raise on mobile
    sum += amp * noise(p);
    p *= 2.0;
    amp *= 0.5;
  }
  return sum;
}

void main() {
  vec2 uv = vUv;
  uv.x *= uResolution.x / uResolution.y; // keep the field square

  float t = uTime * 0.05 + uScroll * 0.6;

  // Domain warp: noise drives the coordinates of the next noise sample.
  vec2 q = vec2(fbm(uv + t), fbm(uv + vec2(5.2, 1.3) - t));
  float field = fbm(uv + q * 1.5);

  vec3 low = vec3(0.04, 0.06, 0.13);  // replace from tokens
  vec3 high = vec3(0.20, 0.45, 0.75); // replace from tokens
  vec3 color = mix(low, high, smoothstep(0.2, 0.8, field));

  fragColor = vec4(color, 1.0);
}
```

## Uniforms the host drives

| Uniform | Meaning | Source |
|---|---|---|
| uTime | Seconds since start | The render loop's timestamp |
| uResolution | Drawing-buffer width and height | Set on resize |
| uScroll | Scroll progress, 0 to 1 | A passive scroll listener, see the seam reference |

## The boundary — WebGL versus CSS

WebGL is justified only when the look genuinely needs per-pixel noise, turbulence, domain warping, or many blended layers animating at 60 fps. For a slow drift of two or three colors with no per-pixel detail, a CSS animated gradient is the correct choice — it is cheaper, adds no canvas, and brings no CSP or accessibility canvas surface to manage. State this honestly when scoping: if the design is a gentle color shift, ship a CSS gradient and skip this skill's machinery. Reserve the canvas for true flow-field atmospheres where CSS would thrash paint or the CPU.

## Production caveats for shadertoy-style shaders

Shaders copied from shadertoy or similar playgrounds are sketches, not production code. They assume high precision that some mobile GPUs do not honor, which causes banding under medium precision, so always declare precision explicitly. They often use high iteration counts or dynamic branching that melt mobile GPUs, so keep loops fixed and low and avoid branching on per-pixel data. Test on real mobile hardware, not only desktop, before shipping.

## References

- [Flowing WebGL gradient, deconstructed](https://alexharri.com/blog/webgl-gradients): the clearest open-source walkthrough of the noise-to-fbm-to-domain-warp pipeline.
- [The Book of Shaders](https://thebookofshaders.com/): foundational fragment-shader and noise concepts from basics.
- [glsl-noise gist by Patricio Gonzalez Vivo](https://gist.github.com/patriciogonzalezvivo/670c22f3966e662d2f83): canonical noise function implementations.
- [Stripe mesh gradient deconstruction](https://kevinhufnagl.com/how-to-stripe-website-gradient-effect/): the layered mesh-gradient alternative when the brief wants smooth color fields rather than a flow field.

## Limitations

- The reference shader is a starting point tuned for legibility, not a final look; the octave count, warp strength, and palette are the knobs to tune.
- Raising the octave count or adding branching to chase fidelity is the most common way to break mobile performance.
