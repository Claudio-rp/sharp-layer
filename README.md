# sharp for AWS Lambda Layers

[![GitHub release](https://img.shields.io/github/tag/Claudio-rp/sharp-layer.svg)](https://github.com/Claudio-rp/sharp-layer/tags)
[![Build action](https://github.com/Claudio-rp/sharp-layer/actions/workflows/build.yml/badge.svg)](https://github.com/Claudio-rp/sharp-layer/actions/workflows/build.yml)

## About

The prebuilt [sharp](https://www.npmjs.com/package/sharp) node module for AWS Lambda layer.

> **Note:** This is a fork of [pH200/sharp-layer](https://github.com/pH200/sharp-layer). Starting with sharp 0.35.3, the esbuild bundling step was removed because sharp's new native module loader uses dynamic `require()` calls that are incompatible with esbuild. The layer now copies sharp's pre-built dist files directly.

### Features

- Built and tested automatically using GitHub Actions
- Automatically releases [sharp](https://www.npmjs.com/package/sharp) updates with GitHub Actions
- Separated builds for `arm64` and `x64`

### Why use a Lambda layer for sharp?

Sharp includes native binaries (libvips) that need to match the Lambda runtime architecture. A pre-built layer avoids bundling issues and ensures the correct native binary is available at runtime. Check out [Optimizing Node.js dependencies in AWS Lambda](https://aws.amazon.com/blogs/compute/optimizing-node-js-dependencies-in-aws-lambda/) for more details.

## Download

[**Releases**](https://github.com/Claudio-rp/sharp-layer/releases)

Download latest [release-arm64.zip](https://github.com/Claudio-rp/sharp-layer/releases/latest/download/release-arm64.zip) or [release-x64.zip](https://github.com/Claudio-rp/sharp-layer/releases/latest/download/release-x64.zip)

## Usage

```js
import sharp from "sharp";
```

Check out [aws: Creating and sharing Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) for more details.

This package can be used with [sst](https://sst.dev). Check out [docs.sst.dev: Lambda Layers](https://docs.sst.dev/advanced/lambda-layers) and [sst.dev: Resize Images](https://sst.dev/examples/how-to-automatically-resize-images-with-serverless.html) for examples.

### Setting arm64 for sst functions

```js
function: {
  handler: '{handler}',
  runtime: 'nodejs20.x',
  architecture: 'arm_64',
  nodejs: {
    esbuild: {
      external: ['sharp'],
    },
  },
  layers: [
    new lambda.LayerVersion(stack, 'SharpLayer', {
      code: lambda.Code.fromAsset('layers/sharp'),
      compatibleArchitectures: [
        lambda.Architecture.ARM_64
      ]
    }),
  ]
}
```

### Setting up a lambda layer for AWS SAM

Providing **a zip file** locally actually works, even though it's **not** mentioned in the documentation.

```yml
## Lambda
ImageFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: image-lambda/
    Handler: app.handler
    Runtime: nodejs20.x
    Architectures:
      - arm64
    Timeout: 30
    MemorySize: 1024
    Layers:
      - !Ref SharpLayer
  Metadata:
    BuildMethod: esbuild
    BuildProperties:
      Format: esm
      OutExtension:
        - .js=.mjs
      EntryPoints:
        - app.ts
      External:
        - "@aws-sdk/*"
        - sharp # use layer
## Lambda layer
SharpLayer:
  Type: AWS::Serverless::LayerVersion
  Properties:
    LayerName: sharp
    ContentUri: layers/sharp/release-arm64.zip # zip
    CompatibleArchitectures:
      - arm64
    CompatibleRuntimes:
      - nodejs20.x
      - nodejs18.x
```

## Build

Fork this repo -> Actions -> Run build.yml

## References

[pH200/sharp-layer](https://github.com/pH200/sharp-layer) - original upstream repo

[Umkus/lambda-layer-sharp](https://github.com/Umkus/lambda-layer-sharp) - another maintained sharp lambda layer

[aws: Creating and sharing Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)

[aws: Working with layers](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-layers.html)

[aws: Building layers](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/building-layers.html)

[sharp: Installation - AWS Lambda](https://sharp.pixelplumbing.com/install#aws-lambda)
