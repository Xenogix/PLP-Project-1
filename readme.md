# PLP - Project 1 - Fractal Explorer

## Description

Fractal Explorer is a command-line application written in Haskell that renders escape-time fractals (Mandelbrot, Julia, Burning Ship, ...) to an image file.

Exploring a fractal usually means opening a GUI or writing a one-off script. This tool lets the user describe a view (fractal, center, zoom, size) directly from the command line and get a reproducible image: the same command always produces the same picture, so interesting locations can be shared, scripted or turned into zoom sequences.

![Fractal](https://upload.wikimedia.org/wikipedia/commons/4/42/Burning_Ship_Fractal.png)

## Features

### Core Ideas

- Parse and validate command-line arguments, with clear error messages for invalid values
- Compute the selected fractal over the complex plane for the requested view (pure computation)
- Map iteration counts to colors with a simple palette
- Write the result as an image file (PPM/BMP, encoded by hand with libraries shipped with GHC)

### Parameters

| Name       | Shorthand | Default      | Description                                         |
|------------|-----------|--------------|-----------------------------------------------------|
| Size       | `-s`      | `1000,1000`  | Size (width,height) of the output image in pixels   |
| Position   | `-p`      | `0,0`        | Center point (x,y) of the view in the complex plane |
| Zoom       | `-z`      | `0`          | Exponential zoom level (scale = 2^zoom)             |
| Iterations | `-i`      | `200`        | Maximum number of iterations per pixel              |
| Function   | `-f`      | `mandelbrot` | Fractal to render (see below)                       |
| Output     | `-o`      | `output.ppm` | Output file name and location                       |

### Predefined functions

For each pixel, $c$ is the corresponding point of the complex plane and $z_0 = 0$ (for `julia`, $z_0 = c$). The pixel color depends on the number of iterations before $\lvert z_n \rvert > 2$.

| Name          | Formula                                                                                          | Parameters |
|---------------|--------------------------------------------------------------------------------------------------|------------|
| `mandelbrot`  | $z_{n+1} = z_n^2 + c$                                                                            | -          |
| `julia`       | $z_{n+1} = z_n^2 + k$                                                                            | $k$        |
| `burningship` | $z_{n+1} = \left(\lvert \mathrm{Re}(z_n) \rvert + i \lvert \mathrm{Im}(z_n) \rvert\right)^2 + c$ | -          |
| `tricorn`     | $z_{n+1} = \overline{z_n}^2 + c$                                                                 | -          |

### Function parameters

Some functions take extra parameters. They are optional and when omitted, the default value is used.

| Name     | Shorthand | Default       | Used by | Description                          |
|----------|-----------|---------------|---------|--------------------------------------|
| Constant | `-k`      | `-0.8,0.156`  | `julia` | Complex constant $k$ given as `re,im` |

### Additional features

If time allows, we would like to add the following features.

#### Interactive mode

An `--interactive` flag will render the fractal directly in the terminal (ANSI colors) and let the user navigate with the keyboard:

| Key           | Action                       |
|---------------|------------------------------|
| `w` `a` `s` `d` | Move the view              |
| `+` / `-`     | Zoom in / out                |
| `e`           | Export current view to file  |
| `q`           | Quit                         |

#### Custom functions

Let the user define their own iteration formula instead of choosing a predefined one:

```sh
fractal-explorer -f "z^3 + c" -o multibrot.ppm
```

The formula would be parsed into an expression tree and evaluated for each iteration. It could use the variables $z$ and $c$, complex constants, the operators `+ - * ^`, parentheses and a few functions (`conj`, `abs`, `re`, `im`). The predefined functions would then simply be predefined expressions rendered by the same engine. Invalid formulas would be reported as errors.

## Example

```sh
fractal-explorer -f burningship -p -1.76,-0.03 -z 5 -s 800,600 -o ship.ppm
```

Expected output:

```
Rendering burningship (800x600) at (-1.76, -0.03), zoom 5, 200 iterations...
Image written to ship.ppm
```

Invalid input is reported without crashing:

```sh
fractal-explorer -f notafunction
```

```
Error: unknown function "notafunction". Available: mandelbrot, julia, burningship, tricorn
```

## References

Everything listed below ships with GHC, so no extra package is required ([list of GHC boot libraries](https://gitlab.haskell.org/ghc/ghc/-/wikis/commentary/libraries/version-history)).

### Complex numbers

- [`Data.Complex`](https://hackage.haskell.org/package/base/docs/Data-Complex.html) (`base`): `Complex Double` type with arithmetic, `magnitude`, `realPart`, `imagPart` and `conjugate`, enough for every formula above.

### Rendering

- [Rosetta Code - Mandelbrot set (Haskell)](https://rosettacode.org/wiki/Mandelbrot_set#Haskell): escape-time computation with `Data.Complex`, rendered as ASCII in a few lines.
- [Rosetta Code - Julia set (Haskell)](https://rosettacode.org/wiki/Julia_set#Haskell): same idea, with the constant read from the command line.
- [Rosetta Code - Write a PPM file (Haskell)](https://rosettacode.org/wiki/Bitmap/Write_a_PPM_file#Haskell): writing an RGB image by hand using only `base`.
- [JuicyPixels](https://hackage.haskell.org/package/JuicyPixels): ready-made PNG encoding, **not shipped with GHC**

### Fractals

- [Mandelbrot set](https://en.wikipedia.org/wiki/Mandelbrot_set), [Julia set](https://en.wikipedia.org/wiki/Julia_set), [Burning Ship fractal](https://en.wikipedia.org/wiki/Burning_Ship_fractal), [Tricorn](https://en.wikipedia.org/wiki/Tricorn_(mathematics))
