# select-videos-on-aws

Using AWS, you can delete only unnecessary files, leaving only the necessary files out of many movie files.  
In this project, pick up files that are not reflected in "human" and delete them.  
If you want to judge things other than "human", you can easily realize it by changing condition parameters

## Getting Started

Rekognition analyzes the movie file uploaded to S3 of AWS, and deletes the file unless the desired label is detected.

## Service Setup

### Architecture

![image](https://user-images.githubusercontent.com/27474266/53300818-2bede300-388f-11e9-9df6-9442eb9ed40b.png)

- When a movie file is uploaded to S3, Lambda function (newFileUploaded) requests a label detection to Amazon Rekognition.
- Afert the label detection, Amazon Rekognition publish the SNS topic and execute the Lambda function (labelDetectionCompleted).
It delete the file if there is no "Human" label.

### S3

If you have no S3 bucket, need to create any bucket to upload files.
["How Do I Create an S3 Bucket?"](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-bucket.html) is helpful.

### IAM

Create two roles.

1. rekognition  
After completing the analysis of Amazon Rekognition, it is necessary to issue SNS topic to Lambda function. When requesting analysis Passing ARN of this role to Amazon Rekognition will issue SNS topic prefixed with "AmazonRekognition".

1. lambda_rekognition_execution  
Here are the roles for executing the two Lambda functions to use.

#### rekognition

*IAM > create role > Rekognition*  
You can accept the default setting menu from here on.

#### lambda_rekognition_execution

*IAM > create role > Lambda*

and attache policies
- AmazonS3FullAccess
- AmazonRekognitionFullAccess
- CloudWatchLogsFullAccess

and JSON inline policies
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::aws:policy/service-role/AmazonRekognitionServiceRole"
    },
    {
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "ARN of rekognition role"
    }
  ]
}
```

### SNS

Create topic 
”AmazonRekognitionLabelDetectionCompleted”.  
Set the ARN of the topic created here to Lambda's environment variable "SNS_TOPIC_ARN".

### Lambda

Create two lambda functions.
1. newFileUploaded  
This function requests a label detection to Amazon Rekognition. A file upload to S3 is an execution trigger.

1. labelDetectionCompleted  
Confirm the labeled result and delete the file if there is no "Human" label.

#### newFileUploaded

Set as the following.  
trigger : S3  
trigger eventtype : ObjectCreated  
runtime : Node.js 8.10  
code : upload "newFileUploaded.zip"  
role : ”lambda_rekognition_execution”  
enviroment values :  
- ROLE_ARN=ARN of role ”rekognition”
- SNS_TOPIC_ARN=ARN of SNS topic ”AmazonRekognitionLabelDetectionCompleted”

#### labelDetectionCompleted

Set as the following.  
trigger : SNS  
trigger topic : AmazonRekognitionLabelDetectionCompleted  
runtime : Node.js 8.10  
handler : labelDetectionCompleted.handler  
code : upload "labelDetectionCompleted.zip"  
role : ”lambda_rekognition_execution”  

## Useage

Let's upload the mp4 file showing the "Human" and the mp4 file not to the bucket of S3. After waiting for a while, there will be only files showing "Human".

## References

[Official Document of Rekognition](https://docs.aws.amazon.com/rekognition/latest/dg/video.html)

[AWS Rekognition Node.js SDK](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Rekognition.html)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
