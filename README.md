# StreetView_ImageClassification
Personal project using SAS software. This is not an official SAS demo.

## Introduction

#### Objective
This project has the objective of building an image classifier capable of distinguishing between pictures of houses taken in place and screenshots of street view.

#### Why?
In some contexts this could be considered as fraud. For example in the context of contractors whose job is to visit houses, they might be requested to provide pictures of the facade of the house as proof of visit. However, it has been detected that some people use screenshots from Google Street View to provide false proof of visit.

#### How?
Using a custom CNN architecture and key specific preprocessing in SAS Viya (VDMML) to build a customized image classification model.

## Street view images
If you have ever used Street View, you know that the images are some sort of 360° image. This type of images are what in photography is called wide-angle. 

#### Wide-angle
If you have ever seen a GoPro picture, well those are wide-angle pictures. These type of cameras are great because they capture a lot of the surroundings into the picture, but they have a downside: they are strange to look at. Straight lines no longera appear tobe straight and the relative size of objects varies depending on distance from the center of the image. This is why Google applies a post process called "defishing" to make the images look normal again.

#### Defishing 
What this process does is essentially stretch the image in a way that the center of the image stays the same, but the edges are zoomed-in. This way, the relative sizes of objects come back to normal and straight lines are straight again. There is, however, a price to pay when doing this: because the edges are zoomed in, they have lower resolution than the center of the image. This is exactly what we are looking for to determine that the image is taken from Street View


![Alt text](/Doc_Resources/zoom.jpg?raw=true "Resolution degradation")

## Implementation
The fact thet Street View images have resolution degradation allows us to solve this problem using a relatively small CNN and no transfer learning. 
#### Preprocessing
Since we are interseted in detecting resolution degradation, we are not going to pass the whole image to the CNN, but only information about the resolution. This was achieved with a Sobel edge detection filter. A laplacian filter was also considered but yielded lower accuracy.

Given this, the pre-processing techniques applied are:
-Grayscale (Since resolution is color independent)
-Sobel edge detection 3x3 kernel (the sharper the edge, the better the resolution)
-Resize to 1120x1120  (Large image size to avoid loosing edge information)

Also, data augmentation techniques were implemented to triplicate the data set:
-Mirror images
-Crop & resize

#### Convolutional Neural Network

The defined architecture has four groups of layers, each group consisting of:\
-Convolutional layer
-Batch normalization layer
-Pooling layer

The architecture is as follows:
-Input 1120x1120x1
-Max pooling 5x5    Output:224x224
-First group:
   7x7x10 Convolution
   Batch Normalization
   3X3 Max pooling
   Output: 56x56
-Second group:
   3x3x20 Convolution
   Batch Normalization
   2x2 Max pooling
   Output: 14x14
-Third group:
   1x1x128 Convolution
   Batch Normalization
   2x2 Mean pooling
   Output: 7x7
-Fourth group:
   1x1x256 Convolution
   Batch Normalization
   7x7 Mean pooling
   Output: 1x1x256
-Output layer: 2 neuron softmax
