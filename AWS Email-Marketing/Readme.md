# AWS SES Email Marketing System

A serverless email marketing solution that sends personalized emails to contacts using AWS Lambda, S3, and SES.

## Project Overview

This cloud-based email marketing system automates sending personalized emails to contacts from a CSV file. It demonstrates practical implementation of AWS serverless architecture for a real-world business need.

## Architecture

![AWS SES Email Marketing Architecture](Images/architecture-diagram.png)

The system workflow:
1. Contact information is stored in CSV format in an S3 bucket
2. HTML email templates are maintained in the same bucket
3. A Lambda function retrieves both the contacts and template
4. The function personalizes emails for each recipient using template variables
5. Emails are sent via Amazon SES with delivery tracking

## Key Components

### Lambda Function (Python)

```python
import boto3
import csv
 
# Initialize the boto3 client
s3_client = boto3.client('s3')
ses_client = boto3.client('ses')
 
def lambda_handler(event, context):
    # Specify the S3 bucket name
    bucket_name = 'awsemailmarketing'  # Replace with your bucket name
 
    try:
        # Retrieve the CSV file from S3
        csv_file = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines = csv_file['Body'].read().decode('utf-8').splitlines()
        
        # Retrieve the HTML email template from S3
        email_template = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html = email_template['Body'].read().decode('utf-8')
        
        # Parse the CSV file
        contacts = csv.DictReader(lines)
        
        for contact in contacts:
            # Replace placeholders in the email template with contact information
            personalized_email = email_html.replace('{{FirstName}}', contact['FirstName'])
            
            # Send the email using SES
            response = ses_client.send_email(
                Source='you@yourdomainname.com',  # Replace with your verified "From" address
                Destination={'ToAddresses': [contact['Email']]},
                Message={
                    'Subject': {'Data': 'Your Weekly Tiny Tales Mail!', 'Charset': 'UTF-8'},
                    'Body': {'Html': {'Data': personalized_email, 'Charset': 'UTF-8'}}
                }
            )
            print(f"Email sent to {contact['Email']}: Response {response}")
    except Exception as e:
        print(f"An error occurred: {e}")
```

### IAM Policy

The following IAM policy grants the Lambda function permissions to access S3 and send emails via SES:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::awsemailmarketing/*"  # Update with your bucket name
        },
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```

### Test Event for Lambda

```json
{
  "comment": "Generic test event for scheduled Lambda execution. The function does not use this event data.",
  "test": true
}
```

## Implementation Screenshots

### S3 Bucket Configuration

[S3 BUCKET SCREENSHOT PLACEHOLDER]

### Lambda Function Setup

[LAMBDA FUNCTION SCREENSHOT PLACEHOLDER]

### SES Email Identity Configuration

[SES CONFIGURATION SCREENSHOT PLACEHOLDER]

### Successful Test Run

[TEST EXECUTION SCREENSHOT PLACEHOLDER]

## Setup Instructions

1. **S3 Bucket Configuration**
   - Create a new S3 bucket (e.g., "awsemailmarketing")
   - Upload two files:
     - `contacts.csv`: Contains columns for at least FirstName and Email
     - `email_template.html`: HTML template with {{FirstName}} placeholders

2. **SES Configuration**
   - Verify the sender email address in Amazon SES
   - If in sandbox mode, verify recipient emails as well

3. **Lambda Function Deployment**
   - Create a new Python Lambda function
   - Copy the provided code
   - Update the bucket name and sender email variables
   - Set appropriate timeout (recommended: 3 minutes)

4. **IAM Role Setup**
   - Create a role with the IAM policy provided above
   - Attach the role to your Lambda function

5. **Testing**
   - Use the test event to manually trigger the function
   - Check CloudWatch logs for execution results
   - Verify emails are received by the contacts

## Additional Implementation Notes

- The system currently processes all contacts in the CSV file during each execution
- Error handling includes logging to CloudWatch
- The template supports basic personalization with the {{FirstName}} variable
- Consider implementing rate limiting for larger contact lists to stay within SES quotas

## Future Enhancements

- Add email open and click tracking
- Implement unsubscribe management
- Create a web interface for template and contact management
- Add support for additional personalization fields
- Implement segmentation for targeted campaigns

## Technologies Used

- **AWS Lambda** - Serverless compute service
- **Amazon S3** - Object storage for contacts and templates
- **Amazon SES** - Email sending service
- **AWS CloudWatch** - Monitoring and logging
- **AWS IAM** - Identity and access management
- **Python** - Programming language for Lambda function


