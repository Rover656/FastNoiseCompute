# FastNoiseCompute
This is a library based on [FastNoise](https://github.com/Auburns/FastNoise) by Auburns implemented in Unity Compute shaders. I'm slowly adding the different noise types.

This should also support other compute shader platforms, as long as they use HLSL.

## API Reference

Note, this library port is a work in progress, this API reference will evolve yet.

### Types

- `FN_DECIMAL` - resolves to either float or double (depending on config).
- `FN_DECIMAL2` - resolves to either float2 or double2 (depending on config).
- `FN_DECIMAL3` - resolves to either float3 or double3 (depending on config).
- `FN_DECIMAL4` - resolves to either float4 or double4 (depending on config).

### Noise types

- `FN_NOISE_VALUE`
- `FN_NOISE_VALUE_FRACTAL`
- `FN_NOISE_PERLIN`
- `FN_NOISE_PERLIN_FRACTAL`
- `FN_NOISE_SIMPLEX`
- `FN_NOISE_SIMPLEX_FRACTAL` - Not implemented yet
- `FN_NOISE_CELLULAR` - Not implemented yet
- `FN_NOISE_WHITENOISE`
- `FN_NOISE_CUBIC` - Not implemented yet
- `FN_NOISE_CUBIC_FRACTAL` - Not implemented yet

### Fractal Types

- `FN_FRACTAL_FBM`
- `FN_FRACTAL_BILLOW`
- `FN_FRACTAL_RIGIDMULTI`

### Interpolation Methods

- `FN_INTERP_LINEAR`
- `FN_INTERP_HERMITE`
- `FN_INTERP_QUINTIC`

### Configuration Functions

- `void FNSetNoiseType(int type)` - Set the type of noise returned by `GetNoise` (default: `FN_NOISE_SIMPLEX`)
- `void FNSetSeed(int seed)` - Set the seed used by all noise functions (default: `1337`).
- `int FNGetSeed()` - Get the seed used by all noise functions
- `void FNSetInterp(int interp)` - Set the interpolation method (default: `FN_INTERP_QUINTIC`).
- `void FNSetFrequency(FN_DECIMAL frequency)` - Set the noise frequency (default: `0.001`).
- `void FNSetFractalOctaves(int octaves)` - Set the number of fractal octaves (default: `3`)
- `void FNSetFractalLacunarity(FN_DECIMAL lacunarity)` - Set the fractal lacunarity (default: `2.0`).
- `void FNSetFractalGain(FN_DECIMAL gain)` - Set the fractal gain (default: `0.5`).
- `void FNSetFractalType(int type)` - Set the fractal type (default: `FN_FRACTAL_FBM`).

### General noise functions

- `FN_DECIMAL FNGetNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)` - Gets 3d noise from the currently selected generator (See `FNSetNoiseType`)
- `FN_DECIMAL FNGetNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)` - Gets 2d noise from the currently selected generator (See `FNSetNoiseType`)

### White Noise Functions

- `FN_DECIMAL FNGetWhiteNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z, FN_DECIMAL w)`
- `FN_DECIMAL FNGetWhiteNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetWhiteNoise(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetWhiteNoiseInt(int x, int y, int z, int w)`
- `FN_DECIMAL FNGetWhiteNoiseInt(int x, int y, int z)`
- `FN_DECIMAL FNGetWhiteNoiseInt(int x, int y)`

### Value Noise Functions

- `FN_DECIMAL FNGetValue(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetValue(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetValueFractal(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)` - Uses the current fractal type.
- `FN_DECIMAL FNGetValueFractal(FN_DECIMAL3 pos)` - Uses the current fractal type.
- `FN_DECIMAL FNGetValue(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetValue(FN_DECIMAL2 pos)`
- `FN_DECIMAL FNGetValueFractal(FN_DECIMAL x, FN_DECIMAL y)` - Uses the current fractal type.
- `FN_DECIMAL FNGetValueFractal(FN_DECIMAL2 pos)` - Uses the current fractal type.

### Perlin Noise Functions

- `FN_DECIMAL FNGetPerlin(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)` - Uses the current fractal type.
- `FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL2 pos)`
- `FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL2 pos)`

### Simplex Noise Functions

- `FN_DECIMAL FNGetSimplex(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL2 pos)`
