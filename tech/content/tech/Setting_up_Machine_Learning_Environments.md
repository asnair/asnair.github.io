+++
date = "2019-01-02T19:05:41-07:00"
title = "Setting up Machine Learning Environments"
draft = true
tags = ["Docker", "DevOps", "Tutorial"]
categories = ["DevOps"]

+++

#Practical Machine Learning

## Applied Neural Networks Part 2 

## ([see part 1 here](https://gooddebate.org/2017/12/applied-neural-networks/) although not required)

Recently I began working on a research product at Purdue, where I was tasked with taking video data from a driving simulator, and identifying objects in the video feed automatically. This type of problem is considered a categorization problem, meaning a Convolutional Neural Network (CNN) is probably the way to go. Many people think of machine learning as very abstract, mathematically intense, and nearly impossible to do if not trained in machine learning specifically. This is far from the case, and most of the time using Neural Networks that are already out there is faster, more efficient, and practically useful. This post is meant to get the average person using neural networks in situations that could use them. Generating the data to train the network should be the most intense part. There will be no math, and no coding, only configuration. 

Before getting started, you have to identify what problem you want to solve. For me, I wanted to identify simulated cars, signs, and markers in a video feed. This post is broken into the following parts:

- Practical Machine Learning
  - Identify the type of network
  - Create/Find data
  - Set up the network
  - train & validate the network
  - Adjust the network
  - Use the network

## Identify the network

Once you know what problem you're trying to solve, you can figure out what method best addresses that problem. 

##Create or Find data

If you can find data sources to train your network, this can be far easier than generating it yourself. For the problem I was trying to address, I tried to use a CNN trained on images of real cars. The simulator was different enough that this didn't work, so I had to train it myself. Useful tools for this process was Labelimg and augimg. Labelimg allows you to select objects in pictures, so I took the video i had, took screenshots every few frames, then tagged the objects I wanted in those screenshots. I was aiming for 50 pictures, if I the accuracy was too low I could add more later.

## Set up the network

When looking for CNNs, I searched for easy to implement CNNs with pretrained weights for cars. I ended up coming across easy_yoloV2, which uses the Yolo v2 architecture implemented in keras, with a tensorflow backend. Keras is a top level library that lets you implement NN algorithms accross different libraries like tensorflow or pytorch so you don't have to worry about specific implementation detail and can focus more on the big ideas.

## Train & Validate the network

This should be the easy part, since any NN you download that is already built should have automated ways to train and validate the data, and will give you estimated accuracy of the model based on the validation.

## Adjust the network

Based on the accuracy, you can add more training data, adjust the weights, and retrain until the accuracy fits your needs. If you're using an approach that doesn't make sense with the problem you are trying to solve, you will never have good accuracy. For example, for classifying objects using a Recursive Neural Network instead of a CNN would yield very poor results and you could never adjust the weights or give it enough data to be more accurate and and more efficient than a CNN.

## Use the network

Once you are satisfied with the model and accuracy, make sure to make the model accessible. Document everything you did, and make sure you can easily access the model to actively use it. If the model is implemented in a way where it can only be consumed through a series of hacks and tribal knowledge, it won't be useful for very long. Eventually, you'll forget how you set it up, and something will break, and if you don't document the system and how it's running it will be very difficult to fix.