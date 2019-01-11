# AWS Serverless Image Handler Lambda wrapper for Thumbor
A solution to dynamically handle images on the fly, utilizing Thumbor (thumbor.org).
Published version, additional details and documentation are available here: https://aws.amazon.com/answers/web-applications/serverless-image-handler/

## Docker Environment Setup
In order to build the package locally, you'll need to build the docker image. In order to do so, run the following command:

```bash
docker build -t serverless-image-handler .
```

This will build a docker image that has the following properties:

* pinned to the `amazonlinux` version that [Lambda runs on](https://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html)
* pinned yum repository to `releasever=2017.03`
  * this is important when building `pycurl` so that the compiled version of libcurl does not differ from the runtime version on the Lambda AMI
* has all of the base requirements installed
  * libpng
  * libjpeg
  * pngcrush
  * gifsicle
  * optipng
  * pngquant

## Building Lambda Package
To build the Lambda package, use the `serverless-image-handler` image that was built earlier. The following command will build the deployment packages:

```bash
docker run -it --rm --volume $PWD:/lambda serverless-image-handler source-bucket-base-name version
```

`source-bucket-base-name` should be the base name for the S3 bucket location where the template will source the Lambda code from.
The template will append '-[region_name]' to this value.
For example: ./build-s3-dist.sh solutions
The template will then expect the source code to be located in the solutions-[region_name] bucket

* Deploy the distributable to an Amazon S3 bucket in your account. Note: you must have the AWS Command Line Interface installed.
```bash
aws s3 cp ./dist/ s3://$DIST_OUTPUT_BUCKET-[region_name]/serverless-image-handler/$VERSION/ --recursive --exclude "*" --include "*.zip"
aws s3 cp ./dist/serverless-image-handler.template s3://$TEMPLATE_OUTPUT_BUCKET/serverless-image-handler/$VERSION/
```
_Note:_ In the above example, the solution template will expect the source code to be located in the my-bucket-name-[region_name] with prefix serverless-image-handler/my-version/serverless-image-handler.zip

* Get the link of the serverless-image-handler.template uploaded to your Amazon S3 bucket.
* Deploy the Serverless Image Handler solution to your account by launching a new AWS CloudFormation stack using the link of the serverless-image-handler.template
```bash
https://s3.amazonaws.com/my-bucket-name/serverless-image-handler/my-version/serverless-image-handler.template
```

## SafeURL hash calculation
* For hash calculation with safe URL, use following snippet to find signed_path value
```bash
http_key='mysecuritykey' # security key provided to lambda env variable
http_path='200x200/smart/sub-folder/myimage.jpg' # sample options for myimage
hashed = hmac.new(str(http_key),str(http_path),sha1)
encoded = base64.b64encode(hashed.digest())
signed_path = encoded.replace('/','_').replace('+','-')
```

***

Copyright 2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License. A copy of the License is located at

    http://aws.amazon.com/asl/

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the License for the specific language governing permissions and limitations under the License.
