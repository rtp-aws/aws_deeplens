# Coffee Consumption Deepdive


## Role Modification

Not sure at this point which AWS things will use this Role.

We added this previously for the hard-hard safety example. 


Add this permission

| Name | Permissions |
| ------------- | -----:|
| AWSDeepLensGreengrassGroupRole  |  AmazonS3FullAccess |



## Creating new project for deeplens device

![face-detection-model.png](pics/face-detection-model.png)

Choose the face detection model and it will create a new lambda function named "deeplens-face-detection".  

The intial lambda function does not have an S3 in the code.  I'm not sure if its uploading to S3.

In the inference run code, add this code snippet to put overlay text on screen.

```
# JFD
cv2.putText(frame,
            'Yo',
            (xmax, ymax-text_offset),
            cv2.FONT_HERSHEY_SIMPLEX,
            2.5,
            (0, 165, 255),
            6)

# putText api parms
#
# image: It is the image on which text is to be drawn.
# text: Text string to be drawn.
# org: It is the coordinates of the bottom-left corner of the text string in the image. The coordinates are represented as tuples of two values i.e. (X coordinate value, Y coordinate value).
# font: It denotes the font type. Some of font types are FONT_HERSHEY_SIMPLEX, FONT_HERSHEY_PLAIN, , etc.
# fontScale: Font scale factor that is multiplied by the font-specific base size.
# color: It is the color of text string to be drawn. For BGR, we pass a tuple. eg: (255, 0, 0) for blue color.
# thickness: It is the thickness of the line in px.
# lineType: This is an optional parameter.It gives the type of the line to be used.
# bottomLeftOrigin: This is an optional parameter. When it is true, the image data origin is at the bottom-left corner. Otherwise, it is at the top-left corner.

```


## create an S3 bucket

This is the bucket name `deeplens-coffee-consumption`


## Modify the source for lambda function

* add bucket name to source
* modify timeout of function in deeplens project settings to be 600 seconds







