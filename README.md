# Autonomous-Retail-Module
Autonomous Retail Module (ARM) for Loss Prevention.

## The Three Core Stages of the ARM System

### Stage 1: Ingestion & Sensing (The Eyes)

- Function: To see everything happening in the store and securely transport that visual data to the cloud.

### Stage 2: Real-time Analysis & Decision (The Brain)

- Function: To process the visual data, understand what's happening, and decide if an event is suspicious.

### Stage 3: Alerting & Response (The Voice & Hands)

- Function: To take immediate action based on the AI's decision, notifying the right people and logging the event for review.

### Solution Architecture Stages

1. IP Cameras → Amazon Kinesis Video Streams: IP cameras in the store capture live video. Kinesis Video Streams securely ingests these encrypted feeds into the AWS cloud.
2. Amazon SageMaker Endpoint → Amazon EventBridge: The video streams are fed into a custom AI model hosted on SageMaker. This model is "always watching" for specific, pre-trained behaviors like shelf sweeps or concealment. When a suspicious behavior is detected, the SageMaker model generates a structured JSON message (e.g., {"event_type": "shelf_sweep"}). The JSON message is sent to EventBridge, which acts as a central "event router," directing the message based on rules you define (e.g., "All 'shelf_sweep' events are high priority").
3. AWS Lambda → Amazon S3: The Lambda function retrieves the video clip of the incident from Kinesis Video Streams and saves this clip as a file in an Amazon S3 bucket for permanent evidence storage.
4. Lambda → Amazon DynamoDB: The Lambda function logs all event metadata (timestamp, camera ID, event type, link to the S3 video clip) into a DynamoDB table. This is your central, auditable case log.
5. Lambda → Amazon SNS (Simple Notification Service): After logging the event, the Lambda function publishes a concise alert message (e.g., "High-Priority Alert: Shelf Sweep at Aisle 5") to an SNS topic.
6. Amazon SNS → End Users: SNS instantly "fans out" the alert to all subscribed staff members via multiple channels, such as push notifications to a mobile app, SMS messages, or emails.
7. Amazon QuickSight → DynamoDB: A manager uses a QuickSight dashboard to connect to the DynamoDB data. This allows them to review cases, view trends, and see a "heatmap" of high-risk areas in the store.
