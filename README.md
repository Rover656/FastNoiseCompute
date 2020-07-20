# FastNoiseCompute
This is a library based on FastNoise C# implemented in Unity Compute shaders. I'm slowly adding the different noise types.

## API Reference

Note, this library port is a work in progress, this API reference will evolve yet.

### Types
- `FN_DECIMAL` - resolves to either float or double (depending on config).
- `FN_DECIMAL2` - resolves to either float2 or double2 (depending on config).
- `FN_DECIMAL3` - resolves to either float3 or double3 (depending on config).
- `FN_DECIMAL4` - resolves to either float4 or double4 (depending on config).

### Noise types
- `FN_NOISE_VALUE`
- `FN_NOISE_VALUE_FRACTAL` - Not implemented yet
- `FN_NOISE_PERLIN`
- `FN_NOISE_PERLIN_FRACTAL` - Not implemented yet
- `FN_NOISE_SIMPLEX`
- `FN_NOISE_SIMPLEX_FRACTAL` - Not implemented yet
- `FN_NOISE_CELLULAR` - Not implemented yet
- `FN_NOISE_WHITENOISE` - Not implemented yet
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
- `int FNGetNoiseType()` - Get the type of noise returned by `GetNoise`
- `void FNSetSeed(int seed)` - Set the seed used by all noise functions (default: `1337`).
- `int FNGetSeed()` - Get the seed used by all noise functions
- `void FNSetInterp(int interp)` - Set the interpolation method (default: `FN_INTERP_QUINTIC`).
- `int FNGetInterp()` - Get the interpolation method.
- `void FNSetFrequency(FN_DECIMAL frequency)` - Set the noise frequency (default: `0.001`).
- `FN_DECIMAL FNGetFrequency()` - Get the noise frequency

### Noise Functions
- `FN_DECIMAL FNGetValue(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetValue(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetValue(FN_DECIMAL2 pos)`
- `FN_DECIMAL FNGetValue(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL2 pos)`
- `FN_DECIMAL FNGetPerlin(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL3 pos)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL2 pos)`
- `FN_DECIMAL FNGetSimplex(FN_DECIMAL x, FN_DECIMAL y)`
- `FN_DECIMAL FNGetNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)` - Gets 3d noise from the currently selected generator (See `FNSetNoiseType`)
- `FN_DECIMAL FNGetNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z)` - Gets 2d noise from the currently selected generator (See `FNSetNoiseType`)
