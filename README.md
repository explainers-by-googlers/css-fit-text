# CSS fit-width text Explainer

This proposal is an early design sketch by Blink Layout Team in Google to describe the problem below and solicit
feedback on the proposed solution. It has not been approved to ship in Chrome.

## Participate
- [csswg-drafts/issues/2528](https://github.com/w3c/csswg-drafts/issues/2528)

## Table of Contents

<!-- Update this table of contents by running `npx doctoc README.md` -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

In text layout, web authors want to align the lines with both ends of the container, but web authors want to achieve this by adjusting the font size instead of justification. Currently, there is no such method in CSS, and the only option is to manually adjust the font size through trial and error or using JavaScript.

Web authors want to fit the text into a container of a specific size without it overflowing. For example, if the container width is narrow and a long word inevitably overflows the container, web authors want to reduce the font size to make it fit within the container. Web authors want to avoid text overflowing the container due to unexpectedly long words used in text translations or when end-users provide arbitrary text.

### Why CSS?
A CSS solution elevates "fit-width text" from a complex, potentially janky scripting problem to a fundamental, performant, and easy-to-use aspect of the CSS layout system.

#### Performance
JavaScript solutions for fitting text typically involve reading element dimensions (layout reads), calculating new styles (computation), and writing new styles (layout writes). When triggered frequently (e.g., during window resizing, container resizing via ResizeObserver, or dynamic content loading), this read-compute-write cycle can lead to layout thrashing, causing jank, slow rendering, and high CPU usage. A native CSS implementation can integrate the text fitting calculation directly into the browser's existing layout and rendering pipeline. This means the fitting happens efficiently as part of the initial layout or subsequent reflows, without the overhead and potential timing issues of stepping in and out of JavaScript.

#### Improved User Experience 
JavaScript-based fitting often suffers from a "flash of unstyled content" or a visible "jump" or reflow as the text is initially rendered at one size and then resized by the script. By making text fitting a native layout property, the browser can calculate the final size/scale before or during the initial paint, resulting in text that appears correctly sized and positioned from the start, providing a much smoother and more professional user experience. This will also improve the quality of the FCP. 

#### Simplified Authoring & Maintainability
Achieving robust text fitting in JavaScript is complex. It requires handling initial load, window resizes, container resizes, potential font loading issues, minimum/maximum size constraints, different fitting methods (font size vs. scaling), and managing the state across various elements. A declarative CSS solution allows developers to achieve the desired behavior with a simple property declaration, significantly reducing the amount of code they need to write, test, and maintain. It integrates seamlessly into the CSS cascade, can be easily controlled by media queries or container queries, and reduces the reliance on complex imperative logic.



## Goals

- Provides a way to align both ends of the text with the width of the container. Consider both cases: fitting by enlarging the text and fitting by shrinking the text.

## Non-goals

- Provide a way to adjust the container width to match to the widest line.
- Provide a way to adjust font size to fit text to the container's width and height.
- Introduce a new line-wrapping algorithm to fit lines to the container width.

## Use cases

Publishing (news sites, blogs, magazines, portfolios) heavily relies on flexible and visually appealing typography that adapts to various layouts and screen sizes. This feature is highly relevant here, particularly for fluid & responsive headlines.  This includes: 

### Expanding

A short headline for a prominent article might look lost in a wide column on a desktop, text-grow could automatically increase its size or character/word spacing to fill the available width, creating a stronger visual impact without manual tweaking or JS calculations based on breakpoints.


### Shrinking

A long headline containing several words or a very long word might easily overflow its container on smaller screens or in constrained sidebar layouts. Text-shrink ensures the headline reduces its size or adjusts spacing to fit within the allocated width, preventing truncation or awkward line breaks without needing JS to measure and resize.


### Combining behaviors 
Using both text-grow and text-shrink creates fluid headlines that always attempt to occupy 100% of their container width, adapting automatically whether the container gets wider or narrower. Creates responsive layouts where text scales naturally with the design.


### Fitting Captions and Pull Quotes
Text accompanying images or used as pull quotes often needs to precisely fit the width of the related content block. text-shrink can prevent overflow, while text-grow can be used to ensure short captions don't look awkward in wide containers.

### Aligning Text Blocks
For certain stylistic layouts, ensuring paragraphs or short text blocks align perfectly to the container's left and right edges by subtly adjusting spacing or size can be achieved declaratively with these properties, rather than relying on justified text which can sometimes lead to excessive gaps.

## [Potential Solution]

We'd like to introduce two CSS properties.

- Name:
  'text-grow'
- Value:
  `<fit-target> <fit-method>? <length>?`
- Initial:
  none
- Applies to:
  text containers

* Name:
  'text-shrink'
* Value:
  `<fit-target> <fit-method>? <length>?`
* Initial:
  none
* Applies to:
  text containers

```
<fit-target> = none | consistent | per-line
```
- `per-line`: Makes each line in the target container larger/smaller independently
- `consistent`: Makes all lines in the target container larger/smaller by a scaling factor for the widest line.

```
<fit-method> = scale | scale-inline | font-size | letter-spacing | ...
```
- `scale`: Scale glyphs in the original font-size.
- `scale-inline`: Scale glyphs in the original font-size only horizontally. It's similar to SVG [`lengthAdjust=spacingAndGlyphs`](https://svgwg.org/svg2-draft/text.html#TextElementLengthAdjustAttribute).  This method doesn't change line height.  See Use case 3C.
- `font-size`: Update the font-size and re-compute glyphs.
- `letter-spacing`: Adjust line width by letter-spacing.  It's similar to SVG [`lengthAdjust=spacing`](https://svgwg.org/svg2-draft/text.html#TextElementLengthAdjustAttribute).  This method doesn't change line height. See Use case 3B.


```<length>```: maximum font-size for `text-grow`, minimum font-size for `text-shrink`.


### Exmaples

#### Expanding

```css
text-grow: per-line;
```
<!--See "Use cases" "Expanding" A--> If a line width is narrower than the container width, line's font-size is increased so that the line width matches the container width. Even if a single font-size is used in the container, each line might have different font-sizes.
If a line width is wider than the container width, the line is unchanged.

```css
text-grow: per-line 30px;
```
Ditto.  However the increased font-size is capped to 30px.  So, lines might be narrower than the container width.

```css
text-grow: consistent;
```
<!--See "Use cases" "Expanding" B--> Compute a scaling factor so that the widest line in the container fits to the container width, and scale all lines in the container by the scaling factor.
If the widest line is wider than the container width, nothing happens.

#### Shrinking

```css
text-shrink: per-line;
```
<!--See "Use cases" "Shrinking" A--> If a line width is wider than the container width, line's font-size is decreased so that the line width matches the container width. Even if a single font-size is used in the container, each line might have different font-sizes.
If a line width is narrower than the container width, the line is unchanged.

```css
text-shrink: per-line 8px;
```
Ditto. However the decreased font-size must not be less than 8px.  So lines might be wider than the container width.

```css
text-shrink: consistent;
```
<!--See "Use cases" "Shrinking" B--> Compute a scaling factor so that the widest line in the container fits to the container width, and scale all lines in the container by the scaling factor.
If the widest line is narrower than the container width, nothing happens.

#### Combining behaviors

```css
text-grow: per-line;
text-shrink: per-line;
```
<!--See "Use cases" "Combining behaviors" A--> If a line width is narrower than the container width, line's font-size is increased so that the line width matches the container width.  If a line width is wider than the container width, line's font-size is decreased so that the line width matches the container width.

```css
text-grow: per-line letter-spacing;
text-shrink: per-line letter-spacing;
```
<!--See "Use cases" "Combining behaviors" B--> If a line width is narrower than the container width, line's letter-spacing is increased so that the line width matches the container width.  If a line width is wider than the container width, line's letter-spacing is decreased so that the line width matches the container width.

```css
text-grow: per-line scale-inline;
text-shrink: per-line scale-inline;
```
<!--See "Use cases" "Combining behaviors" C--> If a line width is narrower or wider than the container width, line's text is scaled horizontally so that the line width matches the container width.

```css
text-grow: consistent;
text-shrink: consistent;
```
Compute a scaling factor so that the widest line in the container fits to the container width, and scale all lines in the container by the scaling factor.  It works even if the widest line is wider or narrower than the container width.

```css
text-shrink: per-line scale-inline;
text-align: justify;
```
<!--See "Use cases" "Combining behaviors" D--> Lines narrower than the container width are justified, and lines wider than the container width are scaled horizontally.


## Detailed design discussion


## Considered alternatives


## References & acknowledgements

