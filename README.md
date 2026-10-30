# denizbt.github.io
My personal webpage! Made with Jekyll, and HTML/CSS/Javascript.

## Publication figures

The `figures` section at the top of `research.html` controls the image beside each
paper. Add your image to `assets/images/`, then fill in its path and descriptive
alt text. For example:

```yaml
figures:
  idealens:
    src: "/assets/images/idealens-figure.webp"
    alt: "Describe what the figure shows, including its main takeaway."
```

The other entries are `decompilation` and `pan`. Leave `src` empty to keep the
placeholder. Figures fit inside a 4:3 space without being cropped; clicking one
opens the full image. A WebP or PNG about 600 pixels wide is a good starting point
for these thumbnails.
