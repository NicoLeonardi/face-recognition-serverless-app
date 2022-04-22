# Sample serverless application

This is a sample application to demonstrate Datadog serverless observability features. It is based out of the [Image Processing AWS Serverless Workshop](https://github.com/aws-samples/aws-serverless-workshops/tree/master/ImageProcessing) but with the following differences:

* Python, instead of Node.js
* It uses the [Serverless Framework](https://serverless.com/) for deployment
* It uses HTTP calls and SNS messages for coordination, instead of Step Functions

## Deployment

The application requires an [AWS Rekognition Collection](https://aws.amazon.com/rekognition/). You can create one with the `aws-cli` tool:

```aws rekognition create-collection --collection-id <your-rekognition-collection-id>```

The rest of the application is deployed using the Serverless Framework. Follow [their instructions](https://serverless.com/framework/docs/getting-started/) to install it.

Once you have the Serverless framework installed, you can deploy the application following these commands:

```
git clone https://github.com/arapulido/face-recognition-serverless-app
cd face-recognition-serverless-app
npm install
serverless deploy --param="ddenv=facerecognition"  --param="ddApiKey=<datadog_api_key>" --param="owner=<your_name>" --param="rekognition-collection-id=<your_rekognition_collection_id>"
```

Once it has been deployed, get the HTTP endpoint for the `face-search` function and redeploy it with the correct endpoint option:

```
serverless deploy --param="ddenv=facerecognition"  --param="ddApiKey=<datadog_api_key>" --param="owner=<your_name>" --param="rekognition-collection-id=<your_rekognition_collection_id>" --param="face-search-endpoint=<face_search_function_endpoint>"
```

## Usage / Testing

There are two ways to use this application, directly using the lambda functions or through a web application.

### Through a web application

There is a [sample web application](https://github.com/arapulido/upload-picture-s3-face-recognition) that will upload a picture to S3 and will called the first function of the pipeline directly, for a more end-to-end user experience. Follow the instructions to deploy and use that application in its [README.md file](https://github.com/arapulido/upload-picture-s3-face-recognition/blob/master/README.md).

### Calling the first Lambda function

1. Upload test images with or without faces to your newly created S3 bucket
1. Call the `detect-faces` endpoint with one of the images as parameter:

```
curl -X POST <detect-faces-endpoint> --data '{"srcBucket": "<s3-bucket-id>", "name":"<image-filename>", "userId":"<random-user-id>"}'
```

The workflow of functions will do the following:

* `face-detection` will check if the picture contains 1 face. If it has more than 1 face or no faces, will return an error.
* `face-search` will check if the same face is already part of the Rekognition collection, and will err if that's the case (duplicated face).
* `face-index` will index the face into the Rekognition collection (to find duplicates later).
* `persist-metadata` will store some metadata into a DynamoDB table.
