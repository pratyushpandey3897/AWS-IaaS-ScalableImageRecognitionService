# AWS-IaaS-ScalableImageRecognitionService
The objective of this project is to create a cloud-based application that provides image recognition as a service to its users. The application should accept png images from the user, which will serve as a workload generator, and use cloud-based SAAS to perform image recognition and return the most relevant label for the image. The application should be capable of scaling and handling concurrent requests as per demand. To achieve this, the application must have two decoupled tiers: the app-tier and the web-tier. The web-tier should be the entry point for the application to receive external requests, forward them to a Simple Queue Service, and store the request image png in an s3 input bucket. The app-tier should scale up using the alarm set up on CloudWatch and poll the messages in the request queue, and perform image classification. An AMI with the classifier installed will be provided. Once the classifier predicts the best possible image, it should save it to a result s3 bucket and send the response to a response SQS queue, which will be polled by the web-tier to forward the response to the client.

# Architecture

<img width="766" alt="aws-IaaS-architecture" src="https://user-images.githubusercontent.com/68020102/229063118-f50f80e4-45d8-4a2f-a3f7-835cfb92c475.png">

# Infrastructure
1. SQS Request Queue. This queue is meant to store the file names of those images that need to be processed by the app tier for classification. Web tier pushes messages into this queue and App tier polls messages from this queue.
2. SQS Response Queue. This queue is meant to store the classification result of images that are classified by the app tier. App tier pushes messages into this queue and web tier polls messages from this queue.
3. Cloud Watch. This is a service provided by AWS to set up alarms. We leverage this service to set up alarms on the request queue. When the alarm is triggered it sets up the policy to scale the app tier. We explain this step in more detail in section 2.2
4. S3 Input Bucket. This is a simple storage service provided by AWS to store objects. We use this service to store the images from the web tier which is later used by the app tier to receive images for classification.
5. S3 Result Bucket. This is also the same S3 Service provided by AWS to store objects. We use this service to store the results from the app tier.

# Autoscaling

The app tier is autoscaled by leveraging AWS CloudWatch. For autoscaling, we followed the steps:
Create an autoscale group with the desired Apptier launch template.This launch template contains the required Apptier jar and systemctl script to execute the jar on instance boot up.To autoscale through alarms, we create cloudwatch alarms based on request queue and response queue size. We tried different alarm metrics to test all scenarios and find the best one that suits our case ( explained in the same section)
