# go-images

[![License](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)](https://github.com/go-images/images/blob/main/LICENSE)
[![Pure Go](https://img.shields.io/badge/pure%20Go-CGO%3D0-00ADD8?logo=go&logoColor=white)](https://github.com/go-images/images)
[![Go Reference](https://pkg.go.dev/badge/github.com/go-images/images.svg)](https://pkg.go.dev/github.com/go-images/images)

**Pictures in Go, with no C library underneath and no picture thrown away.**

Go's standard library decodes four formats, and its decoders answer one question:
*did this decode?* A stream that stopped halfway is an error and nothing else — so a
picture arriving over a slow link cannot be shown while it arrives, even though most
of it is already here. A four-component JPEG decodes, and its chroma is upsampled by
repeating each sample, which is not what libjpeg does and not what the file says. And
for JPEG 2000 or a Photoshop composite there is no decoder at all.

Each repository here answers one of those, and nothing links against a C library to
do it: `CGO_ENABLED=0`, so every one of them cross-compiles to wherever Go goes.

## Repos

| Repo | |
|---|---|
| [`images`](https://github.com/go-images/images) | The processing library, shaped like scikit-image: convolution, Gaussian blur, Sobel and a full Canny detector, morphology, colour spaces, thresholding, resize (nearest, bilinear, area/box, bicubic, Lanczos). Every operation is a **pure function** — it returns a freshly allocated `*image.RGBA` and never touches its input. `Decode` auto-detects eight container formats. |
| [`depth`](https://github.com/go-images/depth) | One picture in, two eyes out. Depth from cues with no model file to ship, and stereo view synthesis that **gathers** rather than scatters — so every output pixel is written exactly once and no hole has to be filled afterwards. |
| [`jpeg`](https://github.com/go-images/jpeg) | A fork of `image/jpeg` that upsamples a four-component picture's chroma **the way libjpeg does**, rather than by repeating each sample. The difference is visible on CMYK and YCCK scans, which is where four components come from. |
| [`png`](https://github.com/go-images/png) | A fork of `image/png` with `DecodePartial(r) (image.Image, int, error)`: as much of the picture as `r` held, **and how much of it is real** — the number of complete pixel rows, counting from the top. For showing an image while it is still arriving. |
| [`gif`](https://github.com/go-images/gif) | The same `DecodePartial` for `image/gif`, on the first frame: the frame at its full declared size, the rows that are complete, and the error that stopped the decode — `nil` when it arrived whole. |
| [`jpeg2000`](https://github.com/go-images/jpeg2000) | A JPEG 2000 **encoder and decoder**, where the standard library has neither: `Encode`, `Decode`, the 5/3 reversible and 9/7 irreversible wavelets, and rate–distortion layer allocation. A fork of [`ajroetker/go-jpeg2000`](https://github.com/ajroetker/go-jpeg2000), and the only fork here. |
| [`psd`](https://github.com/go-images/psd) | Adobe Photoshop `.psd` and `.psb` composites, registered for `image.Decode` like any other format. |

## Why `DecodePartial` is a different function and not a flag

`image.Decode` has one shape and every caller depends on it: a picture, or an error.
A decoder that returned a half-made picture *and* an error would break the callers
that treat an error as "nothing came back", which is almost all of them — and it
would break them silently, by handing them a picture they do not know is incomplete.

So the partial answer has its own entry point, and the incompleteness is **in the
return** rather than implied: the middle value is how many pixel rows are complete,
counting from the top, and the rows past it hold whatever the image was allocated
with. A caller draws those rows and stops. Every existing caller of `Decode` keeps
the contract it was written against.

## The rules these are built on

- **`CGO_ENABLED=0`, no exceptions.** Not a preference: it is what lets one source
  tree cross-compile to wherever Go goes, and what keeps a picture decoder out of the
  class of bug a C image library is famous for.
- **A fork states its divergence.** Each of `jpeg`, `png` and `gif` differs from the
  standard library in exactly one named way, written in its README; `jpeg2000` names
  the upstream it continues. A fork that cannot say what it changed is a fork nobody
  can review.
- **Nothing is re-rolled that a sibling already holds.** The resize modes delegate
  to `go-gfx/gfx/resample`, and `Decode`'s wider formats to the shared codec registry
  in `go-gfx/gfx/codec`. A box average reimplemented in a second repository is two
  box averages to keep identical, and they will not stay identical.
- **A C library is a dependency even when it is invisible.** `ruby-vips` is libvips
  and `RMagick` is ImageMagick; a cgo-backed Go wrapper is the same bargain with the
  seams hidden. The cost of refusing it is, for now, the hand-tuned SIMD those have
  and this does not — stated in the roadmap rather than left to be discovered.

## Links

- Documentation: <https://go-images.github.io/docs/>
- Landing page: <https://go-images.github.io/>
- The brand, and how it is derived: [`brand`](https://github.com/go-images/brand)
