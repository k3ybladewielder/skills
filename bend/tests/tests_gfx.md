# Graphics Rendering and Shaders Subskill (`bend/tests/gfx`)

## Overview
This subskill guides 2D/3D graphical applications, pixel shader rendering, and GPU display loops in pure Bend.

## Key Concepts
- **Pixel Shader Functions**: Pure mathematical mappings from pixel coordinates `(x, y)` to color values `(r, g, b, a)`.
- **Parallel Dispatch**: Shaders are mapped across screen resolutions using parallel calls (`f!(x)` or parallel loops).
- **Frame Buffers**: Screen states are maintained in single-owner mutable arrays or generated on the fly.

## Example Pattern from `bend/tests/gfx`
```python
import Base

def shade_pixel(+u: F32, +v: F32) -> U32:
  r = F32.to_u32(F32.mul(u, 255.0))
  g = F32.to_u32(F32.mul(v, 255.0))
  b = 128
  U32.add(U32.mul(r, 65536), U32.add(U32.mul(g, 256), b))
```

## Agent Checklist
- Keep pixel shader functions completely pure and allocation-free.
- Dispatch parallel rendering using CUDA GPU backend (`bend run -c`).
