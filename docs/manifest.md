# The `manifest` command

## Examples

* Print out a compact JSON RWPM.

    ```sh
    readium manifest publication.epub
    ```
* Pretty-print a JSON RWPM using two-space indent.

    ```she
    readium manifest --indent "  " publication.epub
    ```
* Extract the metadata of a publication with `jq`.

    ```sh
    readium manifest publication.epub | jq -r .metadata
    ```

## Inferring accessibility metadata

The `manifest` command can infer additional accessibility metadata when they are missing, with the `--infer-a11y` flag. 

```sh
readium manifest --infer-a11y=split publication.epub
```

It takes one of the following arguments:

| Option | Description |
| ------ |------------ |
| `no` (*default*) | No accessibility metadata will be inferred. |
| `merged` | Accessibility metadata will be inferred and merged with the authored ones in `accessibility`. |
| `split` | Accessibility metadata will be inferred but stored separately in `https://readium.org/webpub-manifest#inferredAccessibility`. |

### List of inferred metadata

| Key | Value | Rules |
| --- | ----- | ----- |
| `accessMode` | `auditory` | If the publication contains a reference to an audio or video resource (inspect `resources` and `readingOrder` in RWPM). |
| `accessMode` | `visual` | If the publications contains a reference to an image or a video resource (inspect `resources` and `readingOrder` in RWPM). |
| `accessModeSufficient` | `textual` | If the publication is partially or fully accessible (WCAG A or above).<br>Or if the publication does not contain any image, audio or video resource (inspect "resources" and "readingOrder" in RWPM)<br>Or if the only image available can be identified as a cover. |
| `feature` | `displayTransformability` | :warning: This rule is only used with reflowable EPUB files that conform to WCAG AA or above. |
| `feature` | `pageNavigation` | If the publications contains a page list (check for the presence of a `pageList` collection in RWPM). |
| `feature` | `tableOfContents` | If the publications contains a table of contents (check for the presence of a `toc` collection in RWPM). |
| `feature` | `MathML` | If the publication contains any resource with MathML (check for the presence of the `contains` property where the value is `mathml` in `readingOrder` or `resources` in RWPM). |
| `feature` | `synchronizedAudioText` | If the publication contains any reference to Media Overlays. |

## Inspecting images

The `manifest` command can inspect images and extract additional information from them, with the `--inspect-images` flag.

```sh
readium manifest --inspect-images publication.epub
```

When using this flag, each image returned in the manifest will contain the following keys:

* `height` (in pixels)
* `width` (in pixels)
* `size` (in bytes)
* `animated` (a boolean) under `properties`
* `hash` (an array of object) under `properties`


## Ignoring images

The `manifest` command provides two different flags for ignoring images, either by:

* using a directory with the `--infer-a11y-ignore-image-dir` flag
* or a list of algorithms/hashes with the `--infer-a11y-ignore-image-hashes` flag

The `--infer-a11y-ignore-image-dir` needs the path to a directory as an argument:

```sh
readium manifest --infer-a11y=split --infer-a11y-ignore-image-dir=directory publication.epub
```

The `--infer-a11y-ignore-image-hashes` flag takes one or more hashes (in the format &lt;algorithm&gt;:&lt;base64 value&gt;) separated by commas. It will automatically detect the list of algorithms and use them to inspect images.

```sh
readium manifest --infer-a11y=split --infer-a11y-ignore-image-hashes=phash-dct:YzZTDc7IMzk=,sha256:EvaoUnJkxsWkMM0NUf4CwOZMMvEpDRKk7omCBSN67Gc= publication.epub
```

## Using different hashing algorithms

By default, this utility will default to SHA-256 when inspecting images or ignoring images from a directory. This behaviour can be overriden using the `--hash` flag to use additional or different algorithms.

* Inspecting images.

  ```sh
  readium manifest --inspect-images --hash=sha256,phash-dct publication.epub
  ```
    
* Ignoring images from a directory.

  ```sh
  readium manifest --infer-a11y=split --infer-a11y-ignore-image-dir=directory --hash=sha256,phash-dct publication.epub
  ```

It supports one or more values from the following list:

| Option | Algorithm |
| ------ | --------- |
| `sha256` (*default*) | SHA-256 |
| `md5` | MD5 |
| `phash-dct` | [pHash DCT](https://phash.org/) |
| `https://blurha.sh` | [BlurHash](https://blurha.sh) |

In addition to cryptographic hashing algorithms (such as SHA-256 and MD5), perceptual ones such as pHash-DCT are also available. They're useful for detecting similar images or an identical image in a different format, but they have a higher risk of collision than cryptographic hashes.