name: SLSA Provenance for BigHorsePolo26
on:
  push:
    branches: [main]
  release:
    types: [created]
permissions:
  contents: write
  id-token: write
  actions: read

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      digests: ${{ steps.hash.outputs.digests }}
    steps:
      - uses: actions/checkout@v4
      - run: |
          zip -r site.zip. -x ".git/*" ".github/*"
          echo "hashes=$(sha256sum site.zip | base64 -w0)" >> $GITHUB_OUTPUT
        id: hash
      - uses: actions/upload-artifact@v4
        with:
          name: site
          path: site.zip

  provenance:
    needs: [build]
    uses: slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@v1.4.0
    with:
      base64-subjects: "${{ needs.build.outputs.digests }}"
      upload-assets: true
