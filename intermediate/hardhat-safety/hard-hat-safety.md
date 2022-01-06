# Hard Hat Safety

#### Official guide to Hard Hat Safety

## Overview
https://www.awsdeeplens.recipes/300_intermediate/310_worker_safety/

There are three types of image ML 

* image classification - what does the image show?
* object detection - where in the image are bounding boxes for the different objects
* image segmentation - highlight the pixels the same color for each type of object

This project performs object detection.


In this project, the deeplens device has a deeplens project named **davisjf-worker-safety** with  two components:

* deeplens device lambda function  - davisjf-worker-safety-deeplens/versions/1
* deeplens device ML model - deeplens-object-detection

This guide shows how to configure the  deeplens device lambda function **davisjf-worker-safety-deeplens** to have the following characteristics


|                           |                                  |
| :--------------           | -----------------------------:   |
| **Function Name**         |  davisjf-worker-safety-deeplens  |
| **Runtime**               |  python 2.7                      |
| **Entry Point**           |  lambda_function.lambda_handler  |
| **Trigger**               |  none                            |
| **Environment Variables** |  none                            |                                 
| **Role**                  |  DeepLensInferenceRole           |


The python source for the function includes the **greengrass** sdk folder.  The source code itself is uploaded as a zipfile, but the contents of the zip file are in the git repo in the  "worker-safety-AWS-DeepLens-Inference-Lambda-function" directory.  TODO should this be renamed to object detection?  

The lambda_function.py file is modified on line 34 to specify the S3 bucket-name to be "davisjf-worker-safety"  The code does inference/prediction for 20 classes.  The code posts messages in the deeplens specified  iot_topic pubsub topic as given in the aws deeplens device "project output".  This is given in the project output section [here.](https://us-east-1.console.aws.amazon.com/deeplens/home?region=us-east-1#device-details/aws-rtp-DL)  The setting for this device is "$aws/things/deeplens_ZF0JFQ1CQTi-gC5zuctrWQ/infer"


This guide also shows how to setup a second lambda function used by this project.  This second lamda function is triggered by the images put in the S3 bucket by the first lambda function - davisjf-worker-safety-deeplens.  This second function is configured as follows: 

|                           |                                                |
| :--------------           | --------------------------------------:        |
| **Function Name**         |  davisjf-worker-safety-cloud                   |
| **Runtime**               |  python 3.7                                    |
| **Entry Point**           |  lambda_function.lambda_handler                |
| **Trigger**               |  S3 davisjf-worker-safety ObjectCreated        |
| **Environment Variables** |  key iot_topic: value worker-safety-demo-cloud |                                 
| **Role**                  |  RecognizeObjectLambdaRole                     |

The source for this function is in this git repo in the *worker-safety-AWS-DeepLens-detect-Lambda-function* directory.  This code is a single file and is not modified.  The environment variable specifies the iot_topic name for iot pubsub. **NOTE** the official guide uses the same pubsub topic for both lambda functions.  Since the pubsub topics are used for outputs, I prefer to have distinct names so that the output from each function goes to different pubsub topics.  Hence the value for iot_topic name is worker-safety-demo-cloud rather than "$aws/things/deeplens_ZF0JFQ1CQTi-gC5zuctrWQ/infer".

### Project 

comments 

#### Model

Provide notes and context on the model settings page of deeplens-object-detection

#### Function

Placeholder for notes on davisjf-worker-safety-deeplens code

TODO: notation is filename.function for entrypoint specification. But where is the lambda_function in the file?  The script file has a class.  Is the handler a builtin for the base class?

This code takes a raw photo/frame once every second and uploads to S3.  It maintains a folder structure for persons/day/minute/file_every_second.jpg.  Raw means the bounding box is not included in the image.  Perhaps the "persons" folder is only for people.  Even if no one is in frame, ie. no person in photo, the photo is uploaded to S3.

Regarding the **persons** portion of the S3 path, it is hardcoded.  It does not vary based upon the object in the frame.

The ML operation for the frame is put in the IOT MQTT pub/sub.   The topic name is determined by the AWS Deeplens->Device->project-output->MQTT.  The topic name is of this form ``$aws/things/deeplens_ZF0JFQ1CQTi-gC5zuctrWQ/infer`

A couple of notes about the code output in MQTT.

* The client object used to push message is from greengrass sdk.  Search for client.publish code to see output to MQTT.
* Text "Frame pushed to S3" is from line 123.
* Text "Response: some JSON payload" is from line 122.  This is the response code from pushing the frame to S3 using boto library.

```
grep -n client.publish lambda_function.py 
122:        client.publish(topic=iot_topic, payload="Response: {}".format(response))
123:        client.publish(topic=iot_topic, payload="Frame pushed to S3")
126:        client.publish(topic=iot_topic, payload=msg)
152:        client.publish(topic=iot_topic, payload='Loading object detection model')
154:        client.publish(topic=iot_topic, payload='Object detection model loaded')
230:            client.publish(topic=iot_topic, payload=json.dumps(cloud_output))
232:        client.publish(topic=iot_topic, payload='Error in object detection lambda: {}'.format(ex))
```


#### Notes on the code for the deeplens

The prediction call :

```
parsed_inference_results = model.parseResult(model_type, model.doInference(frame_resize))

# iterates the objects in the result 
for obj in parsed_inference_results[model_type]:
                if obj['prob'] > detection_threshold:
                    if(output_map[obj['label']] == 'person'):
                        detectedPerson = True
                        print("vvv detection_threshold={}     curent prob={}".format(detection_threshold, obj['prob']))
                        print("obj={}".format(obj))
                        break

```

The iterative obj results are:

```
[2021-01-31T13:50:31.795-05:00][INFO]-lambda_function.py:201,

obj={'ymax': 311.8924560546875, 
     'label': 15, 
     'xmax': 244.43661499023438, 
     'xmin': 78.04900360107422, 
     'ymin': 50.69304275512695, 
     'prob': 0.97900390625}
```



What is difference between payload is msg and cloud_output?

* msg is when an error happens during push of frame to S3
* The model on the device performs object detection in line 173.  The threshold for probability of class is 0.25.  The code looks like it should only attempt to upload if it sees a person > 0.25 certainty.  It resizes the image before push.  The cloud_output is JSON text which contains the labels and  class probability.




#### The lambda function for davisjf-worker-safety-cloud Notes

This describes how the second lambda functiion (the one which reads from S3) works.


##### input
This code is configurable/parameterized via the environment variable for the lambda to use the specified MQTT name.  The s3 trigger is the other configurable parameter.  The specified bucket is used to determine input.

##### output

* publish via MQTT Image URL, 
    * "PersonsWithHat"
    * "PersonsWithoutHat"
    * "Message": "Person(s): ...Person(s) With Safety Hat: ...Person(s) Without Safety Hat: 0"
* cloudwatch metrics.

##### logic for DetectWorkerSafety()
Determines the number of people used in the MQTT publish.  It does this by calling the AWS Rekognition and then two other routines for parsing output.  Function order is:

1. persons, hardhats = getPersonsAndHardhats(bucket, image_filename, width, height)
    Given a photo, it returns dictionaries of people and hats with associated bounding boxes and class certainty.
     
2. xxx = matchPersonsAndHats(personsList, hardhatsList) It takes the dictionaries and converts to lists, then matches up counts based upon adjacncyt of detected labels for hats and people.  It seems to be looking for hard hats which are above and centered over a person bounding box.  Something like that. TODO: examine in detail logic.

DetectWorkerSafety() pushes to cloudwatch metrics, and calls another routine to perform the MQTT publish.  

## Setup IAM AWS Roles

This section shows how to configure the two IAM AWS Roles used by the two lambda functions.

### Recognize

This is the role used by the standalone lambda function.

| Name | Permissions |
| ------------- | -----:|
| RecognizeObjectLambdaRole  |  AmazonS3FullAccess |
|                            |  AmazonRekognitionReadOnlyAccess |
|                            |  CloudWatchLogsFullAccess |
|                            |  CloudWatchFullAccess       |                                                 
|                            |  AWSIotDataAccess |
|                            |  AWSLambdaFullAccess |


### Deeplens Inference

This is the role used by the lamda function deployed to the deeplens device.


| Name | Permissions |
| ------------- | -----:|
| DeepLensInferenceLambdaRole  |  AmazonS3FullAccess |
|                            |  AmazonRekognitionReadOnlyAccess |
|                            |  CloudWatchLogsFullAccess |
|                            |  CloudWatchFullAccess       |                                                 
|                            |  AWSIotDataAccess |
|                            |  AWSLambdaFullAccess |



### Deeplens Greengrass Predefined Attach Role

This appears to be where you modify a preexisting role to have additional permissions.

| Name | Permissions |
|  -------------   | -----:|
| AWSDeepLensGreengrassGroupRole  |  AmazonS3FullAccess |


## Setup a S3 bucket

| Name | Region |
|  -------------   | -----:|
| davisjf-worker-safety  | US East (N. Virginia) us-east-1  |


## Deploy an object detection project


### Create the lambda inference function for the device

Create the function, using:

o  the inference function directory is [here](deeplens-function).  Upload the zip.  
o  the role is DeepLensInferenceLambdaRole.
o  Modify code so that line 34 has bucket_name = 'davisjf-worker-safety'
o  Use python 2.7



### Create an AWS DeepLens Project for the device

https://console.aws.amazon.com/deeplens/

| Project Name          | Function Name                  | Model Name                | Type |
| -------------         |    ----------:                 | ----------:               | ---- |
| davisjf-worker-safety | davisjf-worker-safety-deeplens | deeplens-object-detection | Name | 


The inference function is from the zip file [here.](https://www.awsdeeplens.recipes/code/worker-safety/worker-safety-deeplens-lambda.zip)  It is the function for the deeplens. Note, that is the original source which has since been modified.


Create this function via:

* Author from scratch
* Python 2.7
* Use an existing role `service-role/AWSDeepLensLambdaRole`


The changes made to the lambda function are:

* line 34 "bucket_name"  should be the name of your bucket.  `bucket_name = "davisjf-worker-safety"`

### Deploy the project to AWS DeepLens

Deploy the project created above.

Examining the code shows that it uses the bucket and a Greengrass iot topic name of AWS_IOT_THING_NAME.  It seems to be 
identifying 20 classes and their probabilities and bounding boxes.

## Create a cloud Lambda function to detect hard hats with Amazon Rekognition

Using AWS Lambda create function


| Function Name               | Runtime      | Role |
|  ----------:                | ----------:  | ---- |
| davisjf-worker-safety-cloud | python 3.7   | RecognizeObjectLambdaRole |

Modifications

* Author from scratch
* Use existing role - RecognizeObjectLambdaRole
* Environment Variable 
   * key iot_topic
   * value $aws/things/deeplens_ZF0JFQ1CQTi-gC5zuctrWQ/infer  pulled from the aws deeplens device project output section.
   * WRONG value worker-safety-demo-cloud  corresponds to the name of the pub/sub topic
* Originally use cloud-lambda-py which is [here.](https://www.awsdeeplens.recipes/code/worker-safety/cloud-lambda.py)
*  The version above has been modified as used [here](cloud-function).  
* Trigger is S3 
    * bucket is davisjf-worker-safety
    * event type "default" object creation
    * make sure Enable trigger checkbox is confirmed
    

    https://console.aws.amazon.com/iot/home?region=us-east-1#/test


## View Output in Cloudwatch

https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#dashboards:name=worker-safety-dashboard-davisjf

custom namespace
metrics

