// FastNoise.compute
//
// MIT License
//
// Copyright(c) 2017 Jordan Peck, 2020 Reece Mackie
//
// Permission is hereby granted, free of charge, to any person obtaining a copy
// of this software and associated documentation files(the "Software"), to deal
// in the Software without restriction, including without limitation the rights
// to use, copy, modify, merge, publish, distribute, sublicense, and / or sell
// copies of the Software, and to permit persons to whom the Software is
// furnished to do so, subject to the following conditions :
//
// The above copyright notice and this permission notice shall be included in all
// copies or substantial portions of the Software.
//
// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.IN NO EVENT SHALL THE
// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
// SOFTWARE.
//
// The developer's email is reezixce@nerzixdthings.dev (for great email, take
// off every 'zix'.)
//

// Uncomment the line below to swap all the inputs/outputs/calculations of FastNoise to doubles instead of floats
//#define FN_USE_DOUBLES

// Type configuration
#if defined(FN_USE_DOUBLES)
#define FN_DECIMAL double
#define FN_DECIMAL2 double2
#define FN_DECIMAL3 double3
#define FN_DECIMAL4 double4
#else
#define FN_DECIMAL float
#define FN_DECIMAL2 float2
#define FN_DECIMAL3 float3
#define FN_DECIMAL4 float4
#endif

// Interp Methods
#define FN_INTERP_LINEAR 0
#define FN_INTERP_HERMITE 1
#define FN_INTERP_QUINTIC 2

// Fractal Types
#define FN_FRACTAL_FBM 0
#define FN_FRACTAL_BILLOW 1
#define FN_FRACTAL_RIGIDMULTI 2

// Noise Types (If it is commented, it isn't implemented. I'll add more if they are needed by someone utilizing the library)
#define FN_NOISE_VALUE 0
#define FN_NOISE_VALUE_FRACTAL 1
#define FN_NOISE_PERLIN 2
#define FN_NOISE_PERLIN_FRACTAL 3
#define FN_NOISE_SIMPLEX 4
#define FN_NOISE_SIMPLEX_FRACTAL 5
//#define FN_NOISE_CELLULAR 6
#define FN_NOISE_WHITENOISE 7
#define FN_NOISE_CUBIC 8
#define FN_NOISE_CUBIC_FRACTAL 9

// Configuration variables
static int _fn_seed = 1337;
static FN_DECIMAL _fn_frequency = (FN_DECIMAL)0.01;
static int _fn_interp = FN_INTERP_QUINTIC;
static int _fn_noiseType = FN_NOISE_SIMPLEX;

static int _fn_octaves = 3;
static FN_DECIMAL _fn_lacunarity = (FN_DECIMAL) 2.0;
static FN_DECIMAL _fn_gain = (FN_DECIMAL) 0.5;
static int _fn_fractalType = FN_FRACTAL_FBM;

static FN_DECIMAL _fn_fractalBounding;

static const FN_DECIMAL2 FN_GRAD_2D[] = {
    FN_DECIMAL2(-1,-1), FN_DECIMAL2( 1,-1), FN_DECIMAL2(-1, 1), FN_DECIMAL2( 1, 1),
    FN_DECIMAL2( 0,-1), FN_DECIMAL2(-1, 0), FN_DECIMAL2( 0, 1), FN_DECIMAL2( 1, 0)
};

static const FN_DECIMAL3 FN_GRAD_3D[] = {
    FN_DECIMAL3( 1, 1, 0), FN_DECIMAL3(-1, 1, 0), FN_DECIMAL3( 1,-1, 0), FN_DECIMAL3(-1,-1, 0),
    FN_DECIMAL3( 1, 0, 1), FN_DECIMAL3(-1, 0, 1), FN_DECIMAL3( 1, 0,-1), FN_DECIMAL3(-1, 0,-1),
    FN_DECIMAL3( 0, 1, 1), FN_DECIMAL3( 0,-1, 1), FN_DECIMAL3( 0, 1,-1), FN_DECIMAL3( 0,-1,-1),
    FN_DECIMAL3( 1, 1, 0), FN_DECIMAL3( 0,-1, 1), FN_DECIMAL3(-1, 1, 0), FN_DECIMAL3( 0,-1,-1),
};

int _FNFastFloor(FN_DECIMAL f) { return (f >= 0 ? (int)f : (int) f - 1); }

int _FNFastRound(FN_DECIMAL f) { return (f >= 0) ? (int)(f + (FN_DECIMAL)0.5) : (int)(f - (FN_DECIMAL)0.5); }

FN_DECIMAL _FNLerp(FN_DECIMAL a, FN_DECIMAL b, FN_DECIMAL t) { return a + t * (b - a); }

FN_DECIMAL _FNInterpHermiteFunc(FN_DECIMAL t) { return t * t * (3 - 2 * t); }

FN_DECIMAL _FNInterpQuinticFunc(FN_DECIMAL t) { return t * t * t * (t * (t * 6 - 15) + 10); }

FN_DECIMAL _FNCubicLerp(FN_DECIMAL a, FN_DECIMAL b, FN_DECIMAL c, FN_DECIMAL d, FN_DECIMAL t) {
    FN_DECIMAL p = (d - c) - (a - b);
    return t * t * t * p + t * t * ((a - b) - p) + t * (c - a) + b;
}

void _FNCalculateFractalBounding() {
    FN_DECIMAL amp = _fn_gain;
    FN_DECIMAL ampFractal = 1;
    for (int i = 1; i < _fn_octaves; i++) {
        ampFractal += amp;
        amp *= _fn_gain;
    }
    _fn_fractalBounding = 1 / ampFractal;
}

// Hashing
static const int X_PRIME = 1619;
static const int Y_PRIME = 31337;
static const int Z_PRIME = 6971;
static const int W_PRIME = 1013;

int _FNHash2D(int seed, int x, int y) {
    int hash = seed;
    hash ^= X_PRIME * x;
    hash ^= Y_PRIME * y;
    
    hash = hash * hash * hash * 60493;
    hash = (hash >> 13) ^ hash;
    
    return hash;
}

int _FNHash3D(int seed, int x, int y, int z) {
    int hash = seed;
    hash ^= X_PRIME * x;
    hash ^= Y_PRIME * y;
    hash ^= Z_PRIME * z;

    hash = hash * hash * hash * 60493;
    hash = (hash >> 13) ^ hash;
    return hash;
}

int _FNHash4D(int seed, int x, int y, int z, int w) {
    int hash = seed;
    hash ^= X_PRIME * x;
    hash ^= Y_PRIME * y;
    hash ^= Z_PRIME * z;
    hash ^= W_PRIME * w;

    hash = hash * hash * hash * 60493;
    hash = (hash >> 13) ^ hash;
    
    return hash;
}

FN_DECIMAL _FNValCoord2D(int seed, int x, int y) {
    int n = seed;
    n ^= X_PRIME * x;
    n ^= Y_PRIME * y;

    return (n * n * n * 60493) / (FN_DECIMAL)2147483648.0;
}

FN_DECIMAL _FNValCoord3D(int seed, int x, int y, int z) {
    int n = seed;
    n ^= X_PRIME * x;
    n ^= Y_PRIME * y;
    n ^= Z_PRIME * z;

    return (n * n * n * 60493) / (FN_DECIMAL)2147483648.0;
}

FN_DECIMAL _FNValCoord4D(int seed, int x, int y, int z, int w) {
    int n = seed;
    n ^= X_PRIME * x;
    n ^= Y_PRIME * y;
    n ^= Z_PRIME * z;
    n ^= W_PRIME * w;

    return (n * n * n * 60493) / (FN_DECIMAL)2147483648.0;
}

FN_DECIMAL _FNGradCoord2D(int seed, int x, int y, FN_DECIMAL xd, FN_DECIMAL yd) {
    int hash = seed;
    hash ^= X_PRIME * x;
    hash ^= Y_PRIME * y;
    
    hash = hash * hash * hash * 60493;
    hash = (hash >> 13) ^ hash;
    
    FN_DECIMAL2 g = FN_GRAD_2D[hash & 7];
    return xd * g.x + yd * g.y;
}

FN_DECIMAL _FNGradCoord3D(int seed, int x, int y, int z, FN_DECIMAL xd, FN_DECIMAL yd, FN_DECIMAL zd) {
    int hash = seed;
    hash ^= X_PRIME * x;
    hash ^= Y_PRIME * y;
    hash ^= Z_PRIME * z;
    
    hash = hash * hash * hash * 60493;
    hash = (hash >> 13) ^ hash;
    
    FN_DECIMAL3 g = FN_GRAD_3D[hash & 15];
    
    return xd * g.x + yd * g.y + zd * g.z;
}

FN_DECIMAL _FNGradCoord4D(int seed, int x, int y, int z, int w, FN_DECIMAL xd, FN_DECIMAL yd, FN_DECIMAL zd, FN_DECIMAL wd) {
    int hash = seed;
    hash ^= X_PRIME * x;
    hash ^= Y_PRIME * y;
    hash ^= Z_PRIME * z;
    hash ^= W_PRIME * w;

    hash = hash * hash * hash * 60493;
    hash = (hash >> 13) ^ hash;

    hash &= 31;
    FN_DECIMAL a = yd, b = zd, c = wd;            // X,Y,Z
    switch (hash >> 3) { // OR, DEPENDING ON HIGH ORDER 2 BITS:
        case 1: a = wd; b = xd; c = yd; break;     // W,X,Y
        case 2: a = zd; b = wd; c = xd; break;     // Z,W,X
        case 3: a = yd; b = zd; c = wd; break;     // Y,Z,W
    }
    return ((hash & 4) == 0 ? -a : a) + ((hash & 2) == 0 ? -b : b) + ((hash & 1) == 0 ? -c : c);
}

// White noise
int _FNFloatCast2Int(FN_DECIMAL f) {
    uint low;
    uint high;
    asuint(f, low, high);
    return (int) (low ^ high);
}

FN_DECIMAL FNGetWhiteNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z, FN_DECIMAL w) {
    int xi = _FNFloatCast2Int(x);
    int yi = _FNFloatCast2Int(y);
    int zi = _FNFloatCast2Int(z);
    int wi = _FNFloatCast2Int(w);

    return _FNValCoord4D(_fn_seed, xi, yi, zi, wi);
}

FN_DECIMAL FNGetWhiteNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int xi = _FNFloatCast2Int(x);
    int yi = _FNFloatCast2Int(y);
    int zi = _FNFloatCast2Int(z);

    return _FNValCoord3D(_fn_seed, xi, yi, zi);
}

FN_DECIMAL FNGetWhiteNoise(FN_DECIMAL x, FN_DECIMAL y) {
    int xi = _FNFloatCast2Int(x);
    int yi = _FNFloatCast2Int(y);

    return _FNValCoord2D(_fn_seed, xi, yi);
}

FN_DECIMAL FNGetWhiteNoiseInt(int x, int y, int z, int w) {
    return _FNValCoord4D(_fn_seed, x, y, z, w);
}

FN_DECIMAL FNGetWhiteNoiseInt(int x, int y, int z) {
    return _FNValCoord3D(_fn_seed, x, y, z);
}

FN_DECIMAL FNGetWhiteNoiseInt(int x, int y) {
    return _FNValCoord2D(_fn_seed, x, y);
}

// Value noise
FN_DECIMAL _FNSingleValue(int seed, FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int x0 = _FNFastFloor(x);
    int y0 = _FNFastFloor(y);
    int z0 = _FNFastFloor(z);
    int x1 = x0 + 1;
    int y1 = y0 + 1;
    int z1 = z0 + 1;

    FN_DECIMAL xs, ys, zs;
    switch (_fn_interp) {
        default:
        case FN_INTERP_LINEAR:
            xs = x - x0;
            ys = y - y0;
            zs = z - z0;
            break;
        case FN_INTERP_HERMITE:
            xs = _FNInterpHermiteFunc(x - x0);
            ys = _FNInterpHermiteFunc(y - y0);
            zs = _FNInterpHermiteFunc(z - z0);
            break;
        case FN_INTERP_QUINTIC:
            xs = _FNInterpQuinticFunc(x - x0);
            ys = _FNInterpQuinticFunc(y - y0);
            zs = _FNInterpQuinticFunc(z - z0);
            break;
    }

    FN_DECIMAL xf00 = _FNLerp(_FNValCoord3D(seed, x0, y0, z0), _FNValCoord3D(seed, x1, y0, z0), xs);
    FN_DECIMAL xf10 = _FNLerp(_FNValCoord3D(seed, x0, y1, z0), _FNValCoord3D(seed, x1, y1, z0), xs);
    FN_DECIMAL xf01 = _FNLerp(_FNValCoord3D(seed, x0, y0, z1), _FNValCoord3D(seed, x1, y0, z1), xs);
    FN_DECIMAL xf11 = _FNLerp(_FNValCoord3D(seed, x0, y1, z1), _FNValCoord3D(seed, x1, y1, z1), xs);

    FN_DECIMAL yf0 = _FNLerp(xf00, xf10, ys);
    FN_DECIMAL yf1 = _FNLerp(xf01, xf11, ys);

    return _FNLerp(yf0, yf1, zs);
}

FN_DECIMAL FNGetValue(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    return _FNSingleValue(_fn_seed, x * _fn_frequency, y * _fn_frequency, z * _fn_frequency);
}

FN_DECIMAL FNGetValue(FN_DECIMAL3 pos) {
    return _FNSingleValue(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency, pos.z * _fn_frequency);
}

FN_DECIMAL _FNSingleValueFractalFBM(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSingleValue(seed, x, y, z);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSingleValue(++seed, x, y, z) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleValueFractalBillow(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSingleValue(seed, x, y, z)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSingleValue(++seed, x, y, z)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleValueFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSingleValue(seed, x, y, z));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSingleValue(++seed, x, y, z))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetValueFractal(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    z *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSingleValueFractalFBM(x, y, z);
        case FN_FRACTAL_BILLOW:
            return _FNSingleValueFractalBillow(x, y, z);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSingleValueFractalRigidMulti(x, y, z);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetValueFractal(FN_DECIMAL3 pos) {
    return FNGetValueFractal(pos.x, pos.y, pos.z);
}

FN_DECIMAL _FNSingleValue(int seed, FN_DECIMAL x, FN_DECIMAL y) {
    int x0 = _FNFastFloor(x);
    int y0 = _FNFastFloor(y);
    int x1 = x0 + 1;
    int y1 = y0 + 1;

    FN_DECIMAL xs, ys;
    switch (_fn_interp) {
        default:
        case FN_INTERP_LINEAR:
            xs = x - x0;
            ys = y - y0;
            break;
        case FN_INTERP_HERMITE:
            xs = _FNInterpHermiteFunc(x - x0);
            ys = _FNInterpHermiteFunc(y - y0);
            break;
        case FN_INTERP_QUINTIC:
            xs = _FNInterpQuinticFunc(x - x0);
            ys = _FNInterpQuinticFunc(y - y0);
            break;
    }

    FN_DECIMAL xf0 = _FNLerp(_FNValCoord2D(seed, x0, y0), _FNValCoord2D(seed, x1, y0), xs);
    FN_DECIMAL xf1 = _FNLerp(_FNValCoord2D(seed, x0, y1), _FNValCoord2D(seed, x1, y1), xs);

    return _FNLerp(xf0, xf1, ys);
}

FN_DECIMAL FNGetValue(FN_DECIMAL x, FN_DECIMAL y) {
    return _FNSingleValue(_fn_seed, x * _fn_frequency, y * _fn_frequency);
}

FN_DECIMAL FNGetValue(FN_DECIMAL2 pos) {
    return _FNSingleValue(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency);
}

FN_DECIMAL _FNSingleValueFractalFBM(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSingleValue(seed, x, y);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSingleValue(++seed, x, y) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleValueFractalBillow(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSingleValue(seed, x, y)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        amp *= _fn_gain;
        sum += (abs(_FNSingleValue(++seed, x, y)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleValueFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSingleValue(seed, x, y));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSingleValue(++seed, x, y))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetValueFractal(FN_DECIMAL x, FN_DECIMAL y) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSingleValueFractalFBM(x, y);
        case FN_FRACTAL_BILLOW:
            return _FNSingleValueFractalBillow(x, y);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSingleValueFractalRigidMulti(x, y);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetValueFractal(FN_DECIMAL2 pos) {
    return FNGetValueFractal(pos.x, pos.y);
}

// Perlin noise
FN_DECIMAL _FNSinglePerlin(int seed, FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int x0 = _FNFastFloor(x);
    int y0 = _FNFastFloor(y);
    int z0 = _FNFastFloor(z);
    int x1 = x0 + 1;
    int y1 = y0 + 1;
    int z1 = z0 + 1;
    
    FN_DECIMAL xs, ys, zs;
    
    switch (_fn_interp) {
        default:
        case FN_INTERP_LINEAR:
            xs = x - x0;
            ys = y - y0;
            zs = z - z0;
            break;
        case FN_INTERP_HERMITE:
            xs = _FNInterpHermiteFunc(x - x0);
            ys = _FNInterpHermiteFunc(y - y0);
            zs = _FNInterpHermiteFunc(z - z0);
            break;
        case FN_INTERP_QUINTIC:
            xs = _FNInterpQuinticFunc(x - x0);
            ys = _FNInterpQuinticFunc(y - y0);
            zs = _FNInterpQuinticFunc(z - z0);
            break; 
    }
    
    FN_DECIMAL xd0 = x - x0;
    FN_DECIMAL yd0 = y - y0;
    FN_DECIMAL zd0 = z - z0;
    FN_DECIMAL xd1 = xd0 - 1;
    FN_DECIMAL yd1 = yd0 - 1;
    FN_DECIMAL zd1 = zd0 - 1;
    
    FN_DECIMAL xf00 = _FNLerp(_FNGradCoord3D(seed, x0, y0, z0, xd0, yd0, zd0), _FNGradCoord3D(seed, x1, y0, z0, xd1, yd0, zd0), xs);
    FN_DECIMAL xf10 = _FNLerp(_FNGradCoord3D(seed, x0, y1, z0, xd0, yd1, zd0), _FNGradCoord3D(seed, x1, y1, z0, xd1, yd1, zd0), xs);
    FN_DECIMAL xf01 = _FNLerp(_FNGradCoord3D(seed, x0, y0, z1, xd0, yd0, zd1), _FNGradCoord3D(seed, x1, y0, z1, xd1, yd0, zd1), xs);
    FN_DECIMAL xf11 = _FNLerp(_FNGradCoord3D(seed, x0, y1, z1, xd0, yd1, zd1), _FNGradCoord3D(seed, x1, y1, z1, xd1, yd1, zd1), xs);
    
    FN_DECIMAL yf0 = _FNLerp(xf00, xf10, ys);
    FN_DECIMAL yf1 = _FNLerp(xf01, xf11, ys);
    
    return _FNLerp(yf0, yf1, zs);
}

FN_DECIMAL FNGetPerlin(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    return _FNSinglePerlin(_fn_seed, x * _fn_frequency, y * _fn_frequency, z * _fn_frequency);
}

FN_DECIMAL FNGetPerlin(FN_DECIMAL3 pos) {
    return _FNSinglePerlin(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency, pos.z * _fn_frequency);
}

FN_DECIMAL _FNSinglePerlinFractalFBM(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSinglePerlin(seed, x, y, z);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSinglePerlin(++seed, x, y, z) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSinglePerlinFractalBillow(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSinglePerlin(seed, x, y, z)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSinglePerlin(++seed, x, y, z)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSinglePerlinFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSinglePerlin(seed, x, y, z));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSinglePerlin(++seed, x, y, z))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    z *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSinglePerlinFractalFBM(x, y, z);
        case FN_FRACTAL_BILLOW:
            return _FNSinglePerlinFractalBillow(x, y, z);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSinglePerlinFractalRigidMulti(x, y, z);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL3 pos) {
    return FNGetPerlinFractal(pos.x, pos.y, pos.z);
}

FN_DECIMAL _FNSinglePerlin(int seed, FN_DECIMAL x, FN_DECIMAL y) {
    int x0 = _FNFastFloor(x);
    int y0 = _FNFastFloor(y);
    int x1 = x0 + 1;
    int y1 = y0 + 1;

    FN_DECIMAL xs, ys;
    switch (_fn_interp) {
        default:
        case FN_INTERP_LINEAR:
            xs = x - x0;
            ys = y - y0;
            break;
        case FN_INTERP_HERMITE:
            xs = _FNInterpHermiteFunc(x - x0);
            ys = _FNInterpHermiteFunc(y - y0);
            break;
        case FN_INTERP_QUINTIC:
            xs = _FNInterpQuinticFunc(x - x0);
            ys = _FNInterpQuinticFunc(y - y0);
            break;
    }

    FN_DECIMAL xd0 = x - x0;
    FN_DECIMAL yd0 = y - y0;
    FN_DECIMAL xd1 = xd0 - 1;
    FN_DECIMAL yd1 = yd0 - 1;

    FN_DECIMAL xf0 = _FNLerp(_FNGradCoord2D(seed, x0, y0, xd0, yd0), _FNGradCoord2D(seed, x1, y0, xd1, yd0), xs);
    FN_DECIMAL xf1 = _FNLerp(_FNGradCoord2D(seed, x0, y1, xd0, yd1), _FNGradCoord2D(seed, x1, y1, xd1, yd1), xs);

    return _FNLerp(xf0, xf1, ys);
}

FN_DECIMAL FNGetPerlin(FN_DECIMAL2 pos) {
    return _FNSinglePerlin(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency);
}

FN_DECIMAL FNGetPerlin(FN_DECIMAL x, FN_DECIMAL y) {
    return _FNSinglePerlin(_fn_seed, x * _fn_frequency, y * _fn_frequency);
}

FN_DECIMAL _FNSinglePerlinFractalFBM(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSinglePerlin(seed, x, y);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSinglePerlin(++seed, x, y) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSinglePerlinFractalBillow(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSinglePerlin(seed, x, y)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSinglePerlin(++seed, x, y)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSinglePerlinFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSinglePerlin(seed, x, y));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++) {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSinglePerlin(++seed, x, y))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL x, FN_DECIMAL y) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSinglePerlinFractalFBM(x, y);
        case FN_FRACTAL_BILLOW:
            return _FNSinglePerlinFractalBillow(x, y);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSinglePerlinFractalRigidMulti(x, y);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetPerlinFractal(FN_DECIMAL2 pos) {
    return FNGetPerlinFractal(pos.x, pos.y);
}

// Simplex noise
static const FN_DECIMAL F3 = (FN_DECIMAL)(1.0 / 3.0);
static const FN_DECIMAL G3 = (FN_DECIMAL)(1.0 / 6.0);
static const FN_DECIMAL G33 = G3 * 3 - 1;

FN_DECIMAL _FNSingleSimplex(int seed, FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    FN_DECIMAL t = (x + y + z) * F3;
    int i = _FNFastFloor(x + t);
    int j = _FNFastFloor(y + t);
    int k = _FNFastFloor(z + t);

    t = (i + j + k) * G3;
    FN_DECIMAL x0 = x - (i - t);
    FN_DECIMAL y0 = y - (j - t);
    FN_DECIMAL z0 = z - (k - t);

    int i1, j1, k1;
    int i2, j2, k2;

    if (x0 >= y0) {
        if (y0 >= z0) {
            i1 = 1; j1 = 0; k1 = 0; i2 = 1; j2 = 1; k2 = 0;
        } else if (x0 >= z0) {
            i1 = 1; j1 = 0; k1 = 0; i2 = 1; j2 = 0; k2 = 1;
        } else { // x0 < z0
            i1 = 0; j1 = 0; k1 = 1; i2 = 1; j2 = 0; k2 = 1;
        }
    } else { // x0 < y0
        if (y0 < z0) {
            i1 = 0; j1 = 0; k1 = 1; i2 = 0; j2 = 1; k2 = 1;
        } else if (x0 < z0) {
            i1 = 0; j1 = 1; k1 = 0; i2 = 0; j2 = 1; k2 = 1;
        } else { // x0 >= z0
            i1 = 0; j1 = 1; k1 = 0; i2 = 1; j2 = 1; k2 = 0;
        }
    }

    FN_DECIMAL x1 = x0 - i1 + G3;
    FN_DECIMAL y1 = y0 - j1 + G3;
    FN_DECIMAL z1 = z0 - k1 + G3;
    FN_DECIMAL x2 = x0 - i2 + F3;
    FN_DECIMAL y2 = y0 - j2 + F3;
    FN_DECIMAL z2 = z0 - k2 + F3;
    FN_DECIMAL x3 = x0 + G33;
    FN_DECIMAL y3 = y0 + G33;
    FN_DECIMAL z3 = z0 + G33;

    FN_DECIMAL n0, n1, n2, n3;

    t = (FN_DECIMAL)0.6 - x0 * x0 - y0 * y0 - z0 * z0;
    if (t < 0) n0 = 0;
    else {
        t *= t;
        n0 = t * t * _FNGradCoord3D(seed, i, j, k, x0, y0, z0);
    }

    t = (FN_DECIMAL)0.6 - x1 * x1 - y1 * y1 - z1 * z1;
    if (t < 0) n1 = 0;
    else {
        t *= t;
        n1 = t * t * _FNGradCoord3D(seed, i + i1, j + j1, k + k1, x1, y1, z1);
    }

    t = (FN_DECIMAL)0.6 - x2 * x2 - y2 * y2 - z2 * z2;
    if (t < 0) n2 = 0;
    else {
        t *= t;
        n2 = t * t * _FNGradCoord3D(seed, i + i2, j + j2, k + k2, x2, y2, z2);
    }

    t = (FN_DECIMAL)0.6 - x3 * x3 - y3 * y3 - z3 * z3;
    if (t < 0) n3 = 0;
    else {
        t *= t;
        n3 = t * t * _FNGradCoord3D(seed, i + 1, j + 1, k + 1, x3, y3, z3);
    }

    return 32 * (n0 + n1 + n2 + n3);
}

FN_DECIMAL FNGetSimplex(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    return _FNSingleSimplex(_fn_seed, x * _fn_frequency, y * _fn_frequency, z * _fn_frequency);
}

FN_DECIMAL FNGetSimplex(FN_DECIMAL3 pos) {
    return _FNSingleSimplex(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency, pos.z * _fn_frequency);
}

FN_DECIMAL _FNSingleSimplexFractalFBM(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSingleSimplex(seed, x, y, z);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSingleSimplex(++seed, x, y, z) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleSimplexFractalBillow(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSingleSimplex(seed, x, y, z)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSingleSimplex(++seed, x, y, z)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleSimplexFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSingleSimplex(seed, x, y, z));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSingleSimplex(++seed, x, y, z))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetSimplexFractal(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    z *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSingleSimplexFractalFBM(x, y, z);
        case FN_FRACTAL_BILLOW:
            return _FNSingleSimplexFractalBillow(x, y, z);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSingleSimplexFractalRigidMulti(x, y, z);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetSimplexFractal(FN_DECIMAL3 pos) {
    return FNGetSimplexFractal(pos.x, pos.y, pos.z);
}

static const FN_DECIMAL SQRT3 = (FN_DECIMAL)1.7320508075688772935274463415059;
static const FN_DECIMAL F2 = (FN_DECIMAL)0.5 * (SQRT3 - (FN_DECIMAL)1.0);
static const FN_DECIMAL G2 = ((FN_DECIMAL)3.0 - SQRT3) / (FN_DECIMAL)6.0;

FN_DECIMAL _FNSingleSimplex(int seed, FN_DECIMAL x, FN_DECIMAL y) {
    FN_DECIMAL t = (x + y) * F2;
    int i = _FNFastFloor(x + t);
    int j = _FNFastFloor(y + t);

    t = (i + j) * G2;
    FN_DECIMAL X0 = i - t;
    FN_DECIMAL Y0 = j - t;

    FN_DECIMAL x0 = x - X0;
    FN_DECIMAL y0 = y - Y0;

    int i1, j1;
    if (x0 > y0) {
        i1 = 1; j1 = 0;
    } else {
        i1 = 0; j1 = 1;
    }

    FN_DECIMAL x1 = x0 - i1 + G2;
    FN_DECIMAL y1 = y0 - j1 + G2;
    FN_DECIMAL x2 = x0 - 1 + 2*G2;
    FN_DECIMAL y2 = y0 - 1 + 2*G2;

    FN_DECIMAL n0, n1, n2;

    t = (FN_DECIMAL)0.5 - x0 * x0 - y0 * y0;
    if (t < 0) n0 = 0;
    else {
         t *= t;
         n0 = t * t * _FNGradCoord2D(seed, i, j, x0, y0);
    }

    t = (FN_DECIMAL)0.5 - x1 * x1 - y1 * y1;
    if (t < 0) n1 = 0;
    else {
        t *= t;
        n1 = t * t * _FNGradCoord2D(seed, i + i1, j + j1, x1, y1);
    }

    t = (FN_DECIMAL)0.5 - x2 * x2 - y2 * y2;
    if (t < 0) n2 = 0;
    else {
        t *= t;
        n2 = t * t * _FNGradCoord2D(seed, i + 1, j + 1, x2, y2);
    }

    return 50 * (n0 + n1 + n2);
}

FN_DECIMAL FNGetSimplex(FN_DECIMAL x, FN_DECIMAL y) {
    return _FNSingleSimplex(_fn_seed, x * _fn_frequency, y * _fn_frequency);
}

FN_DECIMAL FNGetSimplex(FN_DECIMAL2 pos) {
    return _FNSingleSimplex(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency);
}

FN_DECIMAL _FNSingleSimplexFractalFBM(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSingleSimplex(seed, x, y);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSingleSimplex(++seed, x, y) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleSimplexFractalBillow(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSingleSimplex(seed, x, y)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSingleSimplex(++seed, x, y)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleSimplexFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSingleSimplex(seed, x, y));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSingleSimplex(++seed, x, y))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetSimplexFractal(FN_DECIMAL x, FN_DECIMAL y) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSingleSimplexFractalFBM(x, y);
        case FN_FRACTAL_BILLOW:
            return _FNSingleSimplexFractalBillow(x, y);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSingleSimplexFractalRigidMulti(x, y);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetSimplexFractal(FN_DECIMAL2 pos) {
    return FNGetSimplexFractal(pos.x, pos.y);
}

// TODO: Simplex 4D

// Cubic Noise
static const FN_DECIMAL CUBIC_3D_BOUNDING = 1 / (FN_DECIMAL)(1.5 * 1.5 * 1.5);

FN_DECIMAL _FNSingleCubic(int seed, FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int x1 = _FNFastFloor(x);
    int y1 = _FNFastFloor(y);
    int z1 = _FNFastFloor(z);

    int x0 = x1 - 1;
    int y0 = y1 - 1;
    int z0 = z1 - 1;
    int x2 = x1 + 1;
    int y2 = y1 + 1;
    int z2 = z1 + 1;
    int x3 = x1 + 2;
    int y3 = y1 + 2;
    int z3 = z1 + 2;

    FN_DECIMAL xs = x - (FN_DECIMAL)x1;
    FN_DECIMAL ys = y - (FN_DECIMAL)y1;
    FN_DECIMAL zs = z - (FN_DECIMAL)z1;

    return _FNCubicLerp(
        _FNCubicLerp(
        _FNCubicLerp(_FNValCoord3D(seed, x0, y0, z0), _FNValCoord3D(seed, x1, y0, z0), _FNValCoord3D(seed, x2, y0, z0), _FNValCoord3D(seed, x3, y0, z0), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y1, z0), _FNValCoord3D(seed, x1, y1, z0), _FNValCoord3D(seed, x2, y1, z0), _FNValCoord3D(seed, x3, y1, z0), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y2, z0), _FNValCoord3D(seed, x1, y2, z0), _FNValCoord3D(seed, x2, y2, z0), _FNValCoord3D(seed, x3, y2, z0), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y3, z0), _FNValCoord3D(seed, x1, y3, z0), _FNValCoord3D(seed, x2, y3, z0), _FNValCoord3D(seed, x3, y3, z0), xs),
            ys),
        _FNCubicLerp(
        _FNCubicLerp(_FNValCoord3D(seed, x0, y0, z1), _FNValCoord3D(seed, x1, y0, z1), _FNValCoord3D(seed, x2, y0, z1), _FNValCoord3D(seed, x3, y0, z1), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y1, z1), _FNValCoord3D(seed, x1, y1, z1), _FNValCoord3D(seed, x2, y1, z1), _FNValCoord3D(seed, x3, y1, z1), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y2, z1), _FNValCoord3D(seed, x1, y2, z1), _FNValCoord3D(seed, x2, y2, z1), _FNValCoord3D(seed, x3, y2, z1), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y3, z1), _FNValCoord3D(seed, x1, y3, z1), _FNValCoord3D(seed, x2, y3, z1), _FNValCoord3D(seed, x3, y3, z1), xs),
            ys),
        _FNCubicLerp(
        _FNCubicLerp(_FNValCoord3D(seed, x0, y0, z2), _FNValCoord3D(seed, x1, y0, z2), _FNValCoord3D(seed, x2, y0, z2), _FNValCoord3D(seed, x3, y0, z2), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y1, z2), _FNValCoord3D(seed, x1, y1, z2), _FNValCoord3D(seed, x2, y1, z2), _FNValCoord3D(seed, x3, y1, z2), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y2, z2), _FNValCoord3D(seed, x1, y2, z2), _FNValCoord3D(seed, x2, y2, z2), _FNValCoord3D(seed, x3, y2, z2), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y3, z2), _FNValCoord3D(seed, x1, y3, z2), _FNValCoord3D(seed, x2, y3, z2), _FNValCoord3D(seed, x3, y3, z2), xs),
            ys),
        _FNCubicLerp(
        _FNCubicLerp(_FNValCoord3D(seed, x0, y0, z3), _FNValCoord3D(seed, x1, y0, z3), _FNValCoord3D(seed, x2, y0, z3), _FNValCoord3D(seed, x3, y0, z3), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y1, z3), _FNValCoord3D(seed, x1, y1, z3), _FNValCoord3D(seed, x2, y1, z3), _FNValCoord3D(seed, x3, y1, z3), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y2, z3), _FNValCoord3D(seed, x1, y2, z3), _FNValCoord3D(seed, x2, y2, z3), _FNValCoord3D(seed, x3, y2, z3), xs),
        _FNCubicLerp(_FNValCoord3D(seed, x0, y3, z3), _FNValCoord3D(seed, x1, y3, z3), _FNValCoord3D(seed, x2, y3, z3), _FNValCoord3D(seed, x3, y3, z3), xs),
            ys),
        zs) * CUBIC_3D_BOUNDING;
}

FN_DECIMAL FNGetCubic(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    return _FNSingleCubic(_fn_seed, x * _fn_frequency, y * _fn_frequency, z * _fn_frequency);
}

FN_DECIMAL FNGetCubic(FN_DECIMAL3 pos) {
    return _FNSingleCubic(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency, pos.z * _fn_frequency);
}

FN_DECIMAL _FNSingleCubicFractalFBM(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSingleCubic(seed, x, y, z);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSingleCubic(++seed, x, y, z) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleCubicFractalBillow(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSingleCubic(seed, x, y, z)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSingleCubic(++seed, x, y, z)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleCubicFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSingleCubic(seed, x, y, z));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;
        z *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSingleCubic(++seed, x, y, z))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetCubicFractal(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    z *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSingleCubicFractalFBM(x, y, z);
        case FN_FRACTAL_BILLOW:
            return _FNSingleCubicFractalBillow(x, y, z);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSingleCubicFractalRigidMulti(x, y, z);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetCubicFractal(FN_DECIMAL3 pos) {
    return FNGetCubicFractal(pos.x, pos.y, pos.z);
}

static const FN_DECIMAL CUBIC_2D_BOUNDING = 1 / (FN_DECIMAL)(1.5 * 1.5);

FN_DECIMAL _FNSingleCubic(int seed, FN_DECIMAL x, FN_DECIMAL y) {
    int x1 = _FNFastFloor(x);
    int y1 = _FNFastFloor(y);

    int x0 = x1 - 1;
    int y0 = y1 - 1;
    int x2 = x1 + 1;
    int y2 = y1 + 1;
    int x3 = x1 + 2;
    int y3 = y1 + 2;

    FN_DECIMAL xs = x - (FN_DECIMAL)x1;
    FN_DECIMAL ys = y - (FN_DECIMAL)y1;

    return _FNCubicLerp(
                    _FNCubicLerp(_FNValCoord2D(seed, x0, y0), _FNValCoord2D(seed, x1, y0), _FNValCoord2D(seed, x2, y0), _FNValCoord2D(seed, x3, y0),
                        xs),
                    _FNCubicLerp(_FNValCoord2D(seed, x0, y1), _FNValCoord2D(seed, x1, y1), _FNValCoord2D(seed, x2, y1), _FNValCoord2D(seed, x3, y1),
                        xs),
                    _FNCubicLerp(_FNValCoord2D(seed, x0, y2), _FNValCoord2D(seed, x1, y2), _FNValCoord2D(seed, x2, y2), _FNValCoord2D(seed, x3, y2),
                        xs),
                    _FNCubicLerp(_FNValCoord2D(seed, x0, y3), _FNValCoord2D(seed, x1, y3), _FNValCoord2D(seed, x2, y3), _FNValCoord2D(seed, x3, y3),
                        xs),
                    ys) * CUBIC_2D_BOUNDING;
}

FN_DECIMAL FNGetCubic(FN_DECIMAL x, FN_DECIMAL y) {
    return _FNSingleCubic(_fn_seed, x * _fn_frequency, y * _fn_frequency);
}

FN_DECIMAL FNGetCubic(FN_DECIMAL2 pos) {
    return _FNSingleCubic(_fn_seed, pos.x * _fn_frequency, pos.y * _fn_frequency);
}

FN_DECIMAL _FNSingleCubicFractalFBM(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = _FNSingleCubic(seed, x, y);
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += _FNSingleCubic(++seed, x, y) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleCubicFractalBillow(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = abs(_FNSingleCubic(seed, x, y)) * 2 - 1;
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum += (abs(_FNSingleCubic(++seed, x, y)) * 2 - 1) * amp;
    }

    return sum * _fn_fractalBounding;
}

FN_DECIMAL _FNSingleCubicFractalRigidMulti(FN_DECIMAL x, FN_DECIMAL y) {
    int seed = _fn_seed;
    FN_DECIMAL sum = 1 - abs(_FNSingleCubic(seed, x, y));
    FN_DECIMAL amp = 1;

    for (int i = 1; i < _fn_octaves; i++)
    {
        x *= _fn_lacunarity;
        y *= _fn_lacunarity;

        amp *= _fn_gain;
        sum -= (1 - abs(_FNSingleCubic(++seed, x, y))) * amp;
    }

    return sum;
}

FN_DECIMAL FNGetCubicFractal(FN_DECIMAL x, FN_DECIMAL y) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    
    switch (_fn_fractalType) {
        case FN_FRACTAL_FBM:
            return _FNSingleCubicFractalFBM(x, y);
        case FN_FRACTAL_BILLOW:
            return _FNSingleCubicFractalBillow(x, y);
        case FN_FRACTAL_RIGIDMULTI:
            return _FNSingleCubicFractalRigidMulti(x, y);
        default:
            return 0;
    }
}

FN_DECIMAL FNGetCubicFractal(FN_DECIMAL2 pos) {
    return FNGetCubicFractal(pos.x, pos.y);
}

// TODO: Cellular Noise

// Public API

/*
 * Init function if you don't change the seed.
 */
void FNInit() { _FNCalculateFractalBounding(); }
void FNSetNoiseType(int type) { _fn_noiseType = type; }
void FNSetSeed(int seed) { _fn_seed = seed; _FNCalculateFractalBounding(); } // This must be called before any GetNoise calls to ensure the fractal bounding is set.
int FNGetSeed() { return _fn_seed; }
void FNSetInterp(int interp) { _fn_interp = interp; }
void FNSetFrequency(FN_DECIMAL frequency) { _fn_frequency = frequency; }
void FNSetFractalOctaves(int octaves) { _fn_octaves = octaves; _FNCalculateFractalBounding(); }
void FNSetFractalLacunarity(FN_DECIMAL lacunarity) { _fn_lacunarity = lacunarity; }
void FNSetFractalGain(FN_DECIMAL gain) { _fn_gain = gain; _FNCalculateFractalBounding(); }
void FNSetFractalType(int type) { _fn_fractalType = type; }

FN_DECIMAL FNGetNoise(FN_DECIMAL x, FN_DECIMAL y, FN_DECIMAL z) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    z *= _fn_frequency;
    
    switch (_fn_noiseType) {
        case FN_NOISE_VALUE:
            return _FNSingleValue(_fn_seed, x, y, z);
        case FN_NOISE_VALUE_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSingleValueFractalFBM(x, y, z);
                case FN_FRACTAL_BILLOW:
                    return _FNSingleValueFractalBillow(x, y, z);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSingleValueFractalRigidMulti(x, y, z);
                default:
                    return 0;
            }
        case FN_NOISE_PERLIN:
            return _FNSinglePerlin(_fn_seed, x, y, z);
        case FN_NOISE_PERLIN_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSinglePerlinFractalFBM(x, y, z);
                case FN_FRACTAL_BILLOW:
                    return _FNSinglePerlinFractalBillow(x, y, z);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSinglePerlinFractalRigidMulti(x, y, z);
                default:
                    return 0;
            }
        case FN_NOISE_SIMPLEX:
            return _FNSingleSimplex(_fn_seed, x, y, z);
        case FN_NOISE_SIMPLEX_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSingleSimplexFractalFBM(x, y, z);
                case FN_FRACTAL_BILLOW:
                    return _FNSingleSimplexFractalBillow(x, y, z);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSingleSimplexFractalRigidMulti(x, y, z);
                default:
                    return 0;
            }
        case FN_NOISE_WHITENOISE:
            return FNGetWhiteNoise(x, y, z);
        case FN_NOISE_CUBIC:
            return _FNSingleCubic(_fn_seed, x, y, z);
        case FN_NOISE_CUBIC_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSingleCubicFractalFBM(x, y, z);
                case FN_FRACTAL_BILLOW:
                    return _FNSingleCubicFractalBillow(x, y, z);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSingleCubicFractalRigidMulti(x, y, z);
                default:
                    return 0;
            }
        default:
            return 0;
    }
}

FN_DECIMAL FNGetNoise(FN_DECIMAL x, FN_DECIMAL y) {
    x *= _fn_frequency;
    y *= _fn_frequency;
    
    switch (_fn_noiseType) {
        case FN_NOISE_VALUE:
            return _FNSingleValue(_fn_seed, x, y);
        case FN_NOISE_VALUE_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSingleValueFractalFBM(x, y);
                case FN_FRACTAL_BILLOW:
                    return _FNSingleValueFractalBillow(x, y);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSingleValueFractalRigidMulti(x, y);
                default:
                    return 0;
            }
        case FN_NOISE_PERLIN:
            return _FNSinglePerlin(_fn_seed, x, y);
        case FN_NOISE_PERLIN_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSinglePerlinFractalFBM(x, y);
                case FN_FRACTAL_BILLOW:
                    return _FNSinglePerlinFractalBillow(x, y);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSinglePerlinFractalRigidMulti(x, y);
                default:
                    return 0;
            }
        case FN_NOISE_SIMPLEX:
            return _FNSingleSimplex(_fn_seed, x, y);
        case FN_NOISE_SIMPLEX_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSingleSimplexFractalFBM(x, y);
                case FN_FRACTAL_BILLOW:
                    return _FNSingleSimplexFractalBillow(x, y);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSingleSimplexFractalRigidMulti(x, y);
                default:
                    return 0;
            }
        case FN_NOISE_WHITENOISE:
            return FNGetWhiteNoise(x, y);
        case FN_NOISE_CUBIC:
            return _FNSingleCubic(_fn_seed, x, y);
        case FN_NOISE_CUBIC_FRACTAL:
            switch (_fn_fractalType) {
                case FN_FRACTAL_FBM:
                    return _FNSingleCubicFractalFBM(x, y);
                case FN_FRACTAL_BILLOW:
                    return _FNSingleCubicFractalBillow(x, y);
                case FN_FRACTAL_RIGIDMULTI:
                    return _FNSingleCubicFractalRigidMulti(x, y);
                default:
                    return 0;
            }
        default:
            return 0;
    }
}