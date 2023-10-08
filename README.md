# Hugo ideal image

## Introduction

I created image components a couple of times before:

- [Responsive images for Hugo](https://dev.to/stereobooster/responsive-images-for-hugo-dn9)
- [react-ideal-image](https://github.com/stereobooster/react-ideal-image)

But browsers keep improving - now almost all modern browsers support:

- [loading=lazy](https://caniuse.com/loading-lazy-attr)
- [srcset](https://caniuse.com/srcset)
- [picture](https://caniuse.com/picture)
- [webp](https://caniuse.com/webp)

I checked existing solutions and they either don't do what I want or are complicated (for my taste):

- [Hugo Images Module](https://hugomods.com/en/docs/images/)
- [lazyimg](https://github.com/hugo-mods/lazyimg)
- [DFD Hugo image handling module](https://github.com/danielfdickinson/image-handling-mod-hugo-dfd)

## Features

- Basic
  - sets HTML attribute `src` (required)
  - sets HTML attributes `alt`, and `class` if they passed
- Lazy-load
  - sets HTML attributes `loading="lazy"`, `decoding="async"`
  - sets HTML attributes `width`, `height` to prevent reflow ([LCP](https://web.dev/lcp/))
    - including for SVG
- Responsive
  - sets HTML attribute `srcset` and resizes the image
    - but doesn't upscale image (**not implemented**)
  - adds `source` element for `webp` format
- [Modern LQIP](https://github.com/transitive-bullshit/lqip-modern)
  - Can be disabled for transparent images (png, webp)
  - Another option would be dominant color (**not implemented**)
- [Portable markdown links](https://stereobooster.com/posts/portable-markdown-links/)
- Dark mode (**not implemented**)
- Maybe URL params for resize params (**not implemented**)
  - [Hint](https://gohugo.io/content-management/image-processing/#hint)
  - [Quality](https://gohugo.io/content-management/image-processing/#quality)
  - [Resampling filter](https://gohugo.io/content-management/image-processing/#resampling-filter)

## Implementation

There are different ways to implement image components in Hugo:

- [partial](https://gohugo.io/templates/partials/)
- [render hook](https://gohugo.io/templates/render-hooks/)
- [shortcodes](https://gohugo.io/templates/shortcode-templates/)

I think the most pragmatic way would be to implement partial which accepts an image as a resource. This way it can be reused in render hooks and shortcodes, and the resolution of the image would be the responsibility of the caller.

It can look something like this:

```
{{ partial "picture.html" (dict "img" $img "alt" $.Text) }}
```

Partial `picture.html` would be responsible for detecting width and height, resizing the image, and rendering HTML. There are no special requirements for HTML or CSS so it would be a pretty portable solution (just copy one file).

## Source code

- [picture partial](layouts/partials/picture.html)
- [render hook for image](layouts/_default/_markup/render-image.html)

## Notes

### Image CSS

Partial sets width and height for the image, but this is done so that the browser would know the ratio and can calculate the area that the image would take. Probably you want to use CSS to set the desired size for the image, for example:

```css
img {
  max-width: 100%;
  height: auto;
}
```

### LQIP CSS

For LQIP to work `picture` needs to be the same size as the image. There can be different solutions, but one of the simplest is to make a picture block element - `picture { display: block; }`

Also possible to add blur to the background which would improve appearance:

```css
picture > img {
  -webkit-backdrop-filter: blur(10px);
  backdrop-filter: blur(10px);
}
```

**Note**: LQIP doesn't work in Firefox because of [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1556156)

### Dark mode

I'm not sure yet how to implement "dark mode" for images. From an HTML point of view it is trivial to implement:

```html
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="..." />
  <source media="(prefers-color-scheme: light)" srcset="..." />
  <img src="..." />
</picture>
```

But how to do it from Markdown's point of view? Some options are:

- **convention**. If there is an image `img.png` and `img_dark.png`, the system can automatically find the second image and use it for dark mode
- **shortcode**. Will work, but this is not a markdown solution
- Use some kind of **marker in the url** to distinguis dark-/light-mode images. For example, `#gh-dark-mode-only` or `#gh-light-mode-only`. See [Specifying the theme an image is shown to](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#specifying-the-theme-an-image-is-shown-to)
- SVGs don't need a special solution, they can handle it internally. See [Creating SVG that appears black in light mode and light in dark mode](https://stackoverflow.com/questions/67187091/creating-svg-that-appears-black-in-light-mode-and-light-in-dark-mode)
- for black and white images [`filter: invert(1)`](https://developer.mozilla.org/en-US/docs/Web/CSS/filter-function/invert) would do the trick
