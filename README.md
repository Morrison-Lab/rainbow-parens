# rainbow-parens

A [Quarto](https://quarto.org/) extension that colors each pair of matching brackets by its nesting depth in HTML output,
so a reader can see at a glance which `(` closes which `)`.
It colors brackets in math and in code blocks of every language Quarto highlights (R, Python, Racket and the rest).
PDF and other non-HTML output is unchanged.
In revealjs slides only code is colored, because Quarto typesets slide math with MathJax 2 (see [Limits](#limits)).

## Installation

```bash
quarto add Morrison-Lab/rainbow-parens
```

This installs the extension in `_extensions/Morrison-Lab/rainbow-parens/`.

## Usage

Add the filter to the document or project metadata:

```yaml
filters:
  - Morrison-Lab/rainbow-parens
```

That colors brackets in all math and all highlighted code.
The optional settings narrow it:

```yaml
rainbow-parens:
  math: true          # color brackets in math (default true)
  code: [r, racket]   # only these code languages; true (default) for all, false for none
```

`rainbow-parens: false` turns the extension off for one page.
A value that is none of these stops the render with an error.

## What gets colored

- **Math.**
  The round, square and curly brackets in MathJax math, including stretchy ones from `\left( ... \right)`.
  An opener and its closer share a color even when their shapes differ, so the interval $[0, 1)$ pairs its `[` with its `)`.
  Brackets inside an explicit `\color{...}` keep the author's color.
- **Code.**
  The same brackets in highlighted code blocks,
  except inside strings, comments and character literals.
  Plain output blocks, whose brackets need not balance, are left alone.

A closer with no opener before it keeps the text color.
After the fourth level the colors start again.

## Colors

Each theme has its own palette of four colors, one per nesting level.
Both palettes were chosen with the palette validator from Anthropic's `dataviz` skill,
checking every pair of colors, not only neighbors:

| Level | Light theme | Dark theme |
|-------|-------------|------------|
| 1 | `#2757a8` blue | `#5e99fd` blue |
| 2 | `#84550f` brown | `#ebb678` sand |
| 3 | `#aa0970` magenta | `#ff609c` pink |
| 4 | `#8e2dee` violet | `#4cd9d2` aqua |

- **Color vision deficiency.**
  The worst pair under simulated protanopia or deuteranopia is OKLab ΔE 10.6 (light) and 9.7 (dark), above the validator's target of 8.
- **Normal vision.**
  The closest pair is ΔE 19.5 (light) and 19.1 (dark), above the floor of 15.
- **Contrast.**
  Every color reaches the WCAG text contrast of 4.5:1 on each background it is meant for:
  white, the off-white `#f5f2ea` and parchment `#f1e8d0` page tints and their code backgrounds for the light palette (4.57 to 7.04),
  and Bootstrap's darkly `#222222`, its code background `#303030` and the slides' `#1c1e20` for the dark palette (4.65 to 9.68).

The dark palette applies under Quarto's dark theme (`body.quarto-dark`) and the dark slide theme of the lab's qwt template (`body.qwt-dark`).
On any other page, the script checks the page background whenever the body's or root's class changes,
and adds `rp-dark`, which also selects the dark palette, while the background is dark (relative luminance below 0.179);
so a site with a single dark theme such as `theme: darkly` gets the dark palette too.
Override a color by setting `--rp-1` to `--rp-4` in your own stylesheet.
In print and in the forced-colors (high-contrast) mode, brackets keep the text color.

## How it works

The Lua filter adds a script and a stylesheet to HTML pages, and a `<meta>` tag carrying the settings.
The script registers a MathJax render action that runs after MathJax parses the TeX and before it draws the output.
It walks the parsed math in order, keeps a stack of open brackets, and adds a class `rp-d1` to `rp-d4` to each bracket.
MathJax copies that class onto the element it draws, so the stylesheet can switch colors with the theme.
For code, the script wraps each bracket in a `<span>` with the same class once the page has loaded.

The script must run before MathJax loads, which Quarto's HTML template arranges:
extension scripts come before its deferred MathJax script.
A page that later replaces `window.MathJax` with its own configuration object would drop the render action.

## Limits

- **revealjs math.**
  Quarto renders slide math with MathJax 2.7.9 through reveal.js's math plugin,
  and MathJax 2 has no render actions, so brackets in slide math keep the text color.
  Code on slides is colored.
- **MathJax 4 only.**
  On a page that loads MathJax 2 or 3 itself, or KaTeX, math is not colored.
- **Unmatched brackets.**
  A closer with no opener keeps the text color.
  An opener with no closer, such as the `[` of the French interval `]0, 1[`, leaves every later bracket in that formula or code block one level deeper.
  Brackets inside a token that Pandoc's highlighter does not class as a string, comment or character still count,
  such as the `(` in the Racket symbol `|a(b|`.

See [`example.qmd`](example.qmd) for a demonstration.
It names the filter as `rainbow-parens`, because it uses this repository's own `_extensions/rainbow-parens/` copy;
a document in another repository names it `Morrison-Lab/rainbow-parens`.

## License

MIT; see [`LICENSE`](LICENSE).
A copy sits in the extension folder, so it travels with `quarto add`.
