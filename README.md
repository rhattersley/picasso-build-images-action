# picasso-build-images-action

This action provides the ability to trigger image builds via Picasso that are pushed to Artifactory.

To use this action the following arguments are required:
* `images` A list of image build requests encoded as a JSON array of objects. Each image build request object must contain:
  * `name` The name to use for the image in Artifactory.
  * `path` The relative path within the source repository for the Picasso image definition file. It must not contain two consecutive full stop characters.
  * `tag` The tag to apply to the image in Artifactory.

The `name` and `tag` fields can only use lower and upper case letters, digits, full stops, underscores, and dashes, and may not start or end with a full stop, underscore, or dash.

## Examples

### Basic usage

```yaml
- name: Build my image
  uses: rhattersley/picasso-build-images-action@v1
  with:
    images: '[{"name": "MyImage", "path": "path/to/myimage.yml", "tag": "latest"}]'
```

### Dynamic list of images

```yaml
steps:
  - id: define-images
    uses: actions/github-script@v6
    with:
      script: |
        // Build a list of images, e.g. by globbing for definition files.
        const globber = await glob.create('picasso/*.yml');
        const defn_paths = await globber.glob()
        const path = require('path')
        function translate(defn_path) {
          return {
            name: path.basename(defn_path, path.extname(defn_path)),
            path: path.relative('', defn_path),
            tag: 'latest'
          }
        }
        const images = defn_paths.map(translate)
        return images
  - uses: rhattersley/picasso-build-images-action@v1
    with:
      images: '${{needs.define-images.outputs.result}}'
```

### Build a tagged image for a published release
```yaml
on:
  release:
    types: [published]

jobs:
  build-image:
    runs-on: ubuntu-latest
    steps:
      - uses: rhattersley/picasso-build-images-action@v1
        with:
          images: '[{"name": "MyImage", "path": "path/to/myimage.yml", "tag": "${{ github.event.release.tag_name }}"}]'
```
