# Fork github.com/nfnt/resize

Resize
======

<a href="https://github.com/verystar/resize/actions"><img src="https://github.com/verystar/resize/workflows/build/badge.svg" alt="Build Status"></a>
<a href="https://codecov.io/gh/verystar/resize"><img src="https://codecov.io/gh/verystar/resize/branch/master/graph/badge.svg" alt="codecov"></a>
<a href="https://goreportcard.com/report/github.com/verystar/resize"><img src="https://goreportcard.com/badge/github.com/verystar/resize" alt="Go Report Card
"></a>
<a href="https://pkg.go.dev/github.com/verystar/resize"><img src="https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square" alt="GoDoc"></a>
<a href="https://opensource.org/licenses/mit-license.php" rel="nofollow"><img src="https://badges.frapsoft.com/os/mit/mit.svg?v=103"></a>

Image resizing for the [Go programming language](http://golang.org) with common interpolation methods.

Installation
------------

```bash
$ go get github.com/verystar/resize
```

It's that easy!

Usage
-----

This package needs at least Go 1.1. Import package with

```go
import "github.com/verystar/resize"
```

The resize package provides 2 functions:

* `resize.Resize` creates a scaled image with new dimensions (`width`, `height`) using the interpolation function `interp`.
  If either `width` or `height` is set to 0, it will be set to an aspect ratio preserving value.
* `resize.Thumbnail` downscales an image preserving its aspect ratio to the maximum dimensions (`maxWidth`, `maxHeight`).
  It will return the original image if original sizes are smaller than the provided dimensions.

```go
resize.Resize(width, height uint, img image.Image, interp resize.InterpolationFunction) image.Image
resize.Thumbnail(maxWidth, maxHeight uint, img image.Image, interp resize.InterpolationFunction) image.Image
```

The provided interpolation functions are (from fast to slow execution time)

- `NearestNeighbor`: [Nearest-neighbor interpolation](http://en.wikipedia.org/wiki/Nearest-neighbor_interpolation)
- `Bilinear`: [Bilinear interpolation](http://en.wikipedia.org/wiki/Bilinear_interpolation)
- `Bicubic`: [Bicubic interpolation](http://en.wikipedia.org/wiki/Bicubic_interpolation)
- `MitchellNetravali`: [Mitchell-Netravali interpolation](http://dl.acm.org/citation.cfm?id=378514)
- `Lanczos2`: [Lanczos resampling](http://en.wikipedia.org/wiki/Lanczos_resampling) with a=2
- `Lanczos3`: [Lanczos resampling](http://en.wikipedia.org/wiki/Lanczos_resampling) with a=3

Which of these methods gives the best results depends on your use case.

Sample usage:

```go
package main

import (
	"github.com/verystar/resize"
	"image/jpeg"
	"log"
	"os"
)

func main() {
	// open "test.jpg"
	file, err := os.Open("test.jpg")
	if err != nil {
		log.Fatal(err)
	}

	// decode jpeg into image.Image
	img, err := jpeg.Decode(file)
	if err != nil {
		log.Fatal(err)
	}
	file.Close()

	// resize to width 1000 using Lanczos resampling
	// and preserve aspect ratio
	m := resize.Resize(1000, 0, img, resize.Lanczos3)

	out, err := os.Create("test_resized.jpg")
	if err != nil {
		log.Fatal(err)
	}
	defer out.Close()

	// write new image to file
	jpeg.Encode(out, m, nil)
}
```

Caveats
-------

* Optimized access routines are used for `image.RGBA`, `image.NRGBA`, `image.RGBA64`, `image.NRGBA64`, `image.YCbCr`, `image.Gray`, and `image.Gray16` types. All other image types are accessed in a generic way that will result in slow processing speed.
* JPEG images are stored in `image.YCbCr`. This image format stores data in a way that will decrease processing speed. A resize may be up to 2 times slower than with `image.RGBA`. 


Downsizing Samples
-------

Downsizing is not as simple as it might look like. Images have to be filtered before they are scaled down, otherwise aliasing might occur.
Filtering is highly subjective: Applying too much will blur the whole image, too little will make aliasing become apparent.
Resize tries to provide sane defaults that should suffice in most cases.

### Artificial sample

Original image
![Rings](http://nfnt.github.com/img/rings_lg_orig.png)

<table>
<tr>
<th><img src="http://nfnt.github.com/img/rings_300_NearestNeighbor.png" /><br>Nearest-Neighbor</th>
<th><img src="http://nfnt.github.com/img/rings_300_Bilinear.png" /><br>Bilinear</th>
</tr>
<tr>
<th><img src="http://nfnt.github.com/img/rings_300_Bicubic.png" /><br>Bicubic</th>
<th><img src="http://nfnt.github.com/img/rings_300_MitchellNetravali.png" /><br>Mitchell-Netravali</th>
</tr>
<tr>
<th><img src="http://nfnt.github.com/img/rings_300_Lanczos2.png" /><br>Lanczos2</th>
<th><img src="http://nfnt.github.com/img/rings_300_Lanczos3.png" /><br>Lanczos3</th>
</tr>
</table>

### Real-Life sample

Original image  
![Original](http://nfnt.github.com/img/IMG_3694_720.jpg)

<table>
<tr>
<th><img src="http://nfnt.github.com/img/IMG_3694_300_NearestNeighbor.png" /><br>Nearest-Neighbor</th>
<th><img src="http://nfnt.github.com/img/IMG_3694_300_Bilinear.png" /><br>Bilinear</th>
</tr>
<tr>
<th><img src="http://nfnt.github.com/img/IMG_3694_300_Bicubic.png" /><br>Bicubic</th>
<th><img src="http://nfnt.github.com/img/IMG_3694_300_MitchellNetravali.png" /><br>Mitchell-Netravali</th>
</tr>
<tr>
<th><img src="http://nfnt.github.com/img/IMG_3694_300_Lanczos2.png" /><br>Lanczos2</th>
<th><img src="http://nfnt.github.com/img/IMG_3694_300_Lanczos3.png" /><br>Lanczos3</th>
</tr>
</table>


License
-------

Copyright (c) 2012 Jan Schlicht <janschlicht@gmail.com>
Resize is released under a MIT style license.
