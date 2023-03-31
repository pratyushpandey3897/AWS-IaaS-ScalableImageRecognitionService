# app-tier
This is a spring boot microservice application. But unlike the web tier, the controller in this application is only invoked during startup and most of the functionality is within the image recognition service. This service will use the SQS service within the app tier to poll the SQS Request queue. Once the messages are received it will invoke the classifier python function within the template image. This will produce the classifier result. When the Image Recognition service receives the response it calls the S3 Service to store the result into the s3 storage bucket and then invokes the sqs service to send the message to the sqs response queue.

# Systemctl File Config
To ensure that the java app-tier jar is executed when the instance spins up, we create a sytem service.
```
sudo nano /etc/systemd/system/my-app.service
```

```[Unit]
Description=My App Service
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/usr/bin/java -jar /home/ubuntu/apptier.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
```

```
sudo systemctl enable my-app.service
```
