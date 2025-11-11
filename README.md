## **Working with Amazon S3**

## **Overview**

In this lab, I created and configured an **Amazon Simple Storage Service (Amazon S3)** bucket to share images with an external user at a media company (**mediacouser**) who provides pictures of the products that the café sells. The lab also involved configuring the S3 bucket to automatically send an **email notification** to the administrator whenever the bucket contents were modified using **Amazon Simple Notification Service (SNS)**.

## **Objectives & Learning Outcomes**

By completing this lab, I learned how to:

* Use the **AWS CLI** (`s3api` and `s3` commands) to create and configure an S3 bucket.
* Verify write permissions for an IAM user on an S3 bucket.
* Configure **event notifications** on an S3 bucket to trigger SNS topics.

## Architecture Diagram
<img width="1536" height="400" alt="b952eb8e-753b-4c9b-b17d-7887de6e2a8d" src="https://github.com/user-attachments/assets/ed787998-b8d9-4707-a727-2a3a63b3b083" />

# Architecture Description:

1. The **CLI Host (EC2)** creates and manages the S3 bucket.
2. The **mediacouser IAM user** uploads, edits, or deletes images in the bucket.
3. The **S3 bucket** publishes change notifications (ObjectCreated or ObjectRemoved) to an **SNS topic**.
4. The **Administrator** receives email alerts for all bucket modifications.

## Commands & Steps

```bash
# Step 1: Configure AWS CLI credentials
aws configure
# Provide Access Key, Secret Key, region (us-west-2), and output format (json)

# Step 2: Create a unique S3 bucket
aws s3 mb s3://cafe------ --region us-west-2

# Step 3: Upload images to the /images prefix
aws s3 sync ~/initial-images/ s3://cafe-amara----5/images

# Step 4: Verify uploaded files
aws s3 ls s3://cafe-amara-----/images/ --human-readable --summarize

# Step 5: Review IAM group (mediaco) and IAM user (mediacouser)
# Check group policies for permissions like GetObject, PutObject, and DeleteObject.

# Step 6: Test mediacouser permissions
# Login as mediacouser (via console or CLI) and perform:
# View, Upload, and Delete object tests.

# Step 7: Create SNS topic for S3 event notifications
aws sns create-topic --name s3NotificationTopic

# Step 8: Subscribe administrator email to the topic
aws sns subscribe --topic-arn <SNS_TOPIC_ARN> --protocol email --notification-endpoint <YOUR_EMAIL>

# Step 9: Edit SNS topic policy to allow S3 to publish events
# Update Access Policy JSON to include S3 bucket ARN permission

# Step 10: Create S3 event notification configuration file
vi s3EventNotification.json

# JSON:
{
  "TopicConfigurations": [
    {
      "TopicArn": "<SNS_TOPIC_ARN>",
      "Events": ["s3:ObjectCreated:*", "s3:ObjectRemoved:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            { "Name": "prefix", "Value": "images/" }
          ]
        }
      }
    }
  ]
}

# Step 11: Attach the notification configuration to the S3 bucket
aws s3api put-bucket-notification-configuration \
  --bucket cafe-amaralab2025 \
  --notification-configuration file://s3EventNotification.json

# Step 12: Test bucket change notifications
aws s3api put-object --bucket cafe-amar---- --key images/Caramel-Delight.jpg --body ~/new-images/Caramel-Delight.jpg
aws s3api delete-object --bucket cafe-amara----- --key images/Strawberry-Tarts.jpg
```

## **Screenshots**

* **Bucket Creation and File Upload** – `aws s3 mb` and `aws s3 sync` commands executed successfully.
  <img width="1027" height="345" alt="bucket creation and file upload" src="https://github.com/user-attachments/assets/d22e6c31-de3e-40c2-8aa3-d6739b9bc213" />

* **SNS Suncription Notification Email** – administrator receives S3 event notification.
  <img width="944" height="305" alt="subscription email notification" src="https://github.com/user-attachments/assets/d97081aa-6ebb-40a2-8646-f0b661263ee8" />

## **Tools Used**

* **AWS CLI**
* **Amazon S3** (bucket creation, object storage, and event configuration)
* **Amazon SNS** (topic creation, subscription, and notification delivery)
* **IAM** (user and group permission management)
* **EC2 Instance Connect** (command execution terminal)'

## **Key Takeaways**

* S3 buckets can securely enable **external collaboration** using IAM users and specific prefix-based permissions.
* Event notifications combined with SNS allow **real-time alerts** for bucket object changes.
* IAM policies are essential for limiting **scope of actions** for different users.

## **What Actually Happened**

1. Created a new S3 bucket named `cafe-amaralab2025` using AWS CLI.
2. Uploaded initial images from the EC2 instance into the `/images` prefix.
3. Verified that mediacouser had restricted permissions to upload, delete, and view objects.
4. Created an SNS topic (`s3NotificationTopic`) and subscribed an admin email.
5. Configured S3 to publish notifications for create and delete events to the SNS topic.
6. Received confirmation and event notifications through email after testing uploads and deletions.


## **Author:** 

Amarachi Emeziem

Cloud Engineer/ Security
