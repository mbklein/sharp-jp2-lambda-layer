# Sharp for AWS Lambda (with JPEG 2000 support)
AWS Lambda Layer providing [sharp](https://github.com/lovell/sharp) with JPEG 2000 and WebP support.

Forked and modified from [sharp-heic-lambda-layer](https://github.com/zoellner/sharp-heic-lambda-layer) by Andreas Zoellner.

## Prerequisites

* Docker
* [SAM v1.33.0 or higher](https://github.com/awsdocs/aws-sam-developer-guide/blob/master/doc_source/serverless-sam-cli-install.md)
* Node v20 (for v4.x)

## Usage

```bash
npm run build
SAM_BUCKET=your-s3-bucket npm run deploy
```

The example can be deployed using the following commands
```bash
cd examples
sam build
sam deploy --guided
```

### Lambda Layer
- Add the lambda layer with ARN `arn:aws:lambda:us-east-1:${AWS:AccountId}:layer:sharp-jp2:${LAYER_VERSION}` to any lambda function (replace `${LAYER_VERSION}` with the appropriate version and `${AWS:AccountId}` if you're not using a layer from the same account as the function). You can also import the layer ARN using `!ImportValue SharpJP2LayerArn`.
- Remove sharp from the dependencies in the function code (it will otherwise conflict with the one provided through the layer)
- See [example template](examples/sam-template.yaml) for a complete sample template.

### Environment Variables for build
|            Name | Required |           Default Value |                                   Description |
|-----------------|----------|-------------------------|-----------------------------------------------|
|      SAM_BUCKET |      yes |                         | Name of S3 Bucket to store layer              |
|       S3_PREFIX |       no | sharp-jp2-lambda-layer | Prefix within S3 Bucket to store layer        |
|      STACK_NAME |       no | sharp-jp2-lambda-layer | Name of CloudFormation stack                  |
|      LAYER_NAME |       no |              sharp-jp2 | Name of layer                                 |
|      AWS_REGION |       no |               us-east-1 | AWS Region to deploy to                       |
| ORGANIZATION_ID |       no |                    none | ID of Organization to grant access to layer   |
|       PRINCIPAL |       no |                 account | Principal to grant access to layer            |

For details on `ORGANIZATION_ID` and `PRINCIPAL` please see the equivalent properties in the [CloudFormation Docs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-layerversionpermission.html).

The special value `none` for `ORGANIZATION_ID` is used to disable organization based access.
The special value `account` for `PRINCIPAL` is used to give access to the account the layer is deployed to.

The environment variables are used to create a `samconfig.toml` file that configures the `sam package` and `sam deploy` commands.

### Note regarding build process
Previously, some custom docker images were needed to build this layer. AWS has since published newer images which work out of the box. `saml-cli` version `v1.33.0` is using `public.ecr.aws/sam/build-nodejs14.x:latest-x86_64`

## Background
This repo exists as it is rather painful to compile all libraries required to get sharp to work with JP2 files in an AWS Lambda environment. The sharp repository has several [issues](https://github.com/lovell/sharp/issues) related to this.

### Layer contents
This lambda layer contains the node module [sharp](https://github.com/lovell/sharp). But unlike a normal installation via `npm i sharp` this layer does not use the prebuilt sharp and libvips binaries. This layer compiles libwebp, libde265, libheif, libvips, and sharp from source in order to provide JP2 (and webp) functionality in an AWS Lambda environment.

### CompatibleRuntimes
- `nodejs12.x` (v1.x)
- `nodejs14.x` (v2.x)
- `nodejs16.x` (v3.x)
- `nodejs20.x` (v4.x)

## Contributions
If you would like to contribute to this repository, please open an issue or submit a PR.

If you'd like to sponsor work like this, please visit the [original repo](https://github.com/zoellner/sharp-heic-lambda-layer).

## Licenses
- openjpeg is Copyright Universite catholique de Louvain (UCL), Belgium. See https://github.com/uclouvain/openjpeg/blob/master/LICENSE for details.
- libwebp is Copyright Google Inc. See https://github.com/webmproject/libwebp/blob/master/COPYING for details.
- sharp is licensed under the Apache License, Version 2.0. Copyright Lovell Fuller and contributors. See https://github.com/lovell/sharp/blob/master/LICENSE for details.
- libvips is licensed under the LGPL 2.1+. See https://github.com/libvips/libvips/blob/master/COPYING for details.
- The remainder of the code in this repository is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## Related Resources
Visit [sharp.pixelplumbing.com](https://sharp.pixelplumbing.com/) for complete instructions on sharp.

### Code Repositories
- [sharp](https://github.com/lovell/sharp)
- [openjpeg](https://github.com/uclouvain/openjpeg)
- [libwebp](https://github.com/webmproject/libwebp)
- [libvips](https://github.com/libvips/libvips)
