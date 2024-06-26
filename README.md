 Build a Serverless Email Marketing Application on AWS


## 🤖 Introduction

In this hands-on tutorial, I’ll walk you through how to design and build a simple serverless email marketing application from scratch.  When we're done, our system will send emails on a schedule, grabbing HTML email templates and a list of contacts from an S3 bucket.


## 📐 Architecture Design


![Application Design](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/006fe0ae-d5b7-46b6-b712-e75b99540321)


## ⚙️ AWS Services Used

* Amazon Simple Email Service (SES)
* AWS Lambda
* Amazon Simple Storage Service (S3)
* Amazon EventBridge
* AWS Identity and Access Management (IAM)


## 🔋 Features


👉 A place to store email templates and list of contacts

👉 A way to send emails

👉 A way to “merge” email templates with contacts and send them to the email service

👉 A way to trigger sending of emails on a schedule



## ➡️ Step 1 - Creating an S3 bucket to store email templates and contacts

1. Sign in to the AWS Management Console and open the Amazon S3
2. In the left navigation pane, choose Buckets
3. Choose Create bucket
4. For Bucket name, enter a name for your bucket, i'm going to name mine `myemailstorage`
5. Block Public Access as best practice, and leave all the defaults, scroll down then click on "Create bucket"

![Create](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/34ac6c75-c5b5-4028-b126-215736e5e883)


## ➡️ Step 2 - Create The HTML email template & The CSV file with contact information

1. Inside of our S3 bucket, we want to store an email template, an HTML template that we'll actually send to people and then
we need a list of the contacts as well, the email addresses to send to.
2. For the email template, copy and paste the code below and save it as `email_template.html`

### <a name="snippets">🕸️ Code snippets</a>

<details>
<summary><code>email_template.html</code></summary>

```html
<!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Decode the Programmer's Riddle - Tiny Tales Mail</title>
            <style>
            body {
                font-family: Arial, sans-serif;
                margin: 0;
                padding: 20px;
                background-color: #f4f4f4;
                color: #333;
            }
            .container {
                max-width: 600px;
                margin: auto;
                background: #ffffff;
                padding: 20px;
                border-radius: 8px;
                box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            }
            h1, h2 {
                color: #007bff;
            }
            p {
                font-size: 16px;
                line-height: 1.5;
            }
            .challenge-details {
                background-color: #eee;
                border-left: 5px solid #007bff;
                padding: 10px;
                margin: 20px 0;
            }
            </style>
            </head>
            <body>
            <div class="container">
            <h1>This Week's Tech Challenge: Decode the Programmer's Riddle</h1>
            <p>Hello {{FirstName}}!</p>
            <p>It's that time again—Tiny Tales Mail is back with a new challenge to stretch your tech muscles and ignite your problem-solving spark. This week, we're dialing up the difficulty with a puzzle that blends coding logic, pattern recognition, and a bit of creative thinking. Ready to dive in?</p>
            
            <div class="challenge-details">
                <h2>Challenge: The Programmer's Riddle</h2>
                <p>Imagine you're exploring an ancient digital labyrinth. To escape, you need to input a code sequence into the central computer. You find a clue carved on the wall:</p>
                <p><strong>8, 5, 4, 9, 1, 7, 6, 3, 2, 0</strong></p>
                <p>But there's a catch: The sequence is not what it seems. To find the correct next number, you must uncover the hidden pattern. Here's a hint to guide you on your quest:</p>
                <p><strong>Hint:</strong> The order is determined not by mathematics, but by a property inherent to each number.</p>
            </div>
            
            <h2>How to Participate:</h2>
            <p>Convinced you've unraveled the labyrinth's secret? Email your solution and the reasoning behind it.</p>
            
            <h2>Prizes and Recognition:</h2>
            <p><strong>Grand Solver:</strong> The first to submit the correct answer will receive a $20 Amazon gift card and be featured in our next newsletter.</p>
            <p><strong>Creative Thinkers:</strong> The next four insightful submissions will earn a spot in our "Hall of Fame" section, celebrating your cleverness and creativity.</p>
            
            <p>This puzzle is designed to challenge and inspire. We encourage you to think outside the box and look forward to seeing the innovative ways you approach this problem.</p>
            
            <p>Best of luck, and let the best minds prevail!</p>
            <p>Eagerly awaiting your solutions,</p>
            <p><strong>The Tiny Tales Mail Team</strong></p>
            </div>
        </body>
    </html>
```

</details>

3. The other thing that that we're going to place in the S3 bucket is a CSV file of our contacts, make sure the emails that you put are ones that you can actually validate, meaning you can receive emails and click on a link that AWS sends you if you want them to actually go through, you can easily create one from scratch as well but make sure to call it `contacts.csv`



![Screenshot 2024-06-15 174658](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/a69277a1-c041-4b46-9bdf-51606a06795e)




## ➡️ Step 3 - Uploading the email template and contact list to the S3 bucket

1. Go back to the S3 bucket, naviagate to the object tab, upload the `email_template.html` and `contacts.csv` make sure that the files are called that because later on we'll use them in the Lambda function not always the best idea but for this demo you will need these names to get it to work.

2. Then click on "Upload"


![Screenshot 2024-06-15 174751](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/4cbc80c4-c192-4dbb-bd90-694beb04cf7c)


✔️ we've got the S3 bucket <br/>
✔️ we've got the list of contacts to send to <br/>
✔️ we've also got an email template that we can send <br/>


## ➡️ Step 4 - Setting up Amazon Simple Email Service (SES) with an email address and domain

Before you start using Amazon SES, you must complete the following tasks:
   * Verify an identity for sending email, Verify an email address or domain to use when sending email.
   * Send your first email, Send an email by using the Amazon SES console, the Amazon SES API, an AWS SDK, or the AWS Command-Line Interface.

1. Back over to the console, navigate to SES, then click "Get started"


![Screenshot 2024-06-15 174850](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/ac281e00-02b5-4200-ac7f-7d568f30d026)


2. You need to enter the email address that emails will come from, you do have to validate this, AWS will send you an email there's a link in that email that you need to click on, and make sure it's a valid email address

![Screenshot 2024-06-15 175102](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/405e6935-1932-4ab8-a486-d3ff79ae69be)


3. Add your sending domain, a domain identity that matches your website or business name. Amazon SES needs to be linked to your domain and verified in order to send emails to your recipients through SES.




4. Amazon is going to send an email to your email address, you'll need to open it up, click on the link to verify that you actually own the email address and once you do you'll be able to send emails from that address.


![Screenshot 2024-06-15 175236](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/ac3dfe7e-f2e0-451f-b5b1-a9be5736864e)


5. If you successfully did all of those steps then you also will have a Sandbox dashboard

![Screenshot 2024-06-15 175257](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/5c68176d-6547-4fb2-bc04-80c1623a35fc)

## ➡️ Step 4 - Sending a test email using Amazon SES

Let's send an email to make sure everything is working

1. On your Sandbox dashboard click on "Send test email"


![Screenshot 2024-06-15 175318](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/52c22b02-0be3-4c77-be0a-c665e6338484)

2. Select "Raw" which will let us test with HTML format
3. For Scenario, we want to test the scenario where everything is successful, select "Successful delivery"
(But you can also test different scenarios, like what happens if it bounces or what happens if there's a complaint and so on)
4. Then paste in the HTML that we have from the email template on the Message box
5. Click on "Send test email"


![Screenshot 2024-06-15 175727](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/067eb646-3955-4a81-89e7-fb02bd2ab241)

6. Then you will get a green checkmark, everything is working

![Screenshot 2024-06-15 175804](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/22c23744-14ac-488e-9928-36974e9db561)



## ➡️ Step 5 - Validating email addresses to send to (a requirement for the sandbox)

We need to validate the individual email addresses that we want to send to, that are stored in our contacts.csv file 

1. Back over the Sandbox dashboard, under verify email address click on "View all identities"
2. Let's create a new identity, click on "Create identity"
3. Identity type choose "Email address" and leave everything as default, then click "Create identity"

![Screenshot 2024-06-15 175858](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/9c4a3a0c-b15a-4ce0-837d-cf2d85ea2744)


4. You will recieve an email from your inbox, open it and click the link to verify, you will get a success message, we can now start sending emails to that address

5. After you verify all the emails, you will see a list of verified emails

![Screenshot 2024-06-15 175915](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/1792c4ec-4bfb-4f24-9722-9af702ae9c3f)


## ➡️ Step 6 -  Creating a new Python Lambda function for email logic

We need a way to merge the email template using Lambda to create and send personalized emails to SES with the contacts and then send them to SES to the email service.

1. Back to the console, navigate to Lambda, then click on "Create function"
2. We are going to "Author from scratch", for the function name i'm going to use `mySESConnection`, for Runtime we are going to use `python3.12` leave everything as default then click "Create function"



![Screenshot 2024-06-15 180037](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/c575b009-d68d-4305-a97f-1add4a7e2df1)

3. Scroll down to the code section, i've got some code already so you don't have to write this from scratch:


### <a name="snippets">🕸️ Code snippets</a>

<details>
<summary><code>Lambda Function Python Code</code></summary>

```python
import boto3
import csv

# Initialize the boto3 client
s3_client = boto3.client('s3')
ses_client = boto3.client('ses')

def lambda_handler(event, context):
    # Specify the S3 bucket name
    bucket_name = 'jm-email-marketing' # Replace with your bucket name

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
</details>


6. Grab this code then come back to the Lambda function and just replace everything
7. Make sure to replace your `bucket_name` `email_template` `contact_csv_file` and `you@yourdomainname.com` with your information if you named them differently with mine



![Screenshot 2024-06-15 180136](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/e0655d5a-23e4-475e-a3bf-276874a81d6c)

8. When copy and paste the Python and update it with your information, click on "Deploy"


![Screenshot 2024-06-15 180233](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/21311f7c-0331-4959-9a46-db9260f1a9cf)


## ➡️ Step 7 - Configuring and running a test event for the Lambda function

1. Set up a new test event, click on the Arrow to the right of test, then configure test event, create a new event


![Screenshot 2024-06-15 180259](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/6ca50f05-f98f-4dc6-ba68-4f4a60227efc)


2. We are going to name it `TestSendEmail`
3. we can use a really simple generic JSON test event, copy and replace it in the event, then click "Save"

```json
{
  "comment": "Generic test event for scheduled Lambda execution. The function does not use this event data.",
  "test": true
}
```

![Screenshot 2024-06-15 180259](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/6ca50f05-f98f-4dc6-ba68-4f4a60227efc)



N.B: If you click on Test back over the Lambda source code, you will get an "ACCESS DENIED" message, it's a pretty common thing to come across and this is where the permissions and the execution role come into play, let's do that on the next step.


## ➡️ Step 8 - Updating the Lambda execution role with appropriate permissions

1. Back to the Lambda, go to configutation tab, under permissions, click "Role name"

![Screenshot 2024-06-15 180333](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/9cfc92d1-3407-4835-8530-9374336757db)


2. Scrolling down to permissions policies if we expand the execution Url, we just have some default permissions in to basically write to class watch logs there's nothing specific to S3 or SES, we need to add that manually to do that I'm going to go create a new Creating a new policy with permissions for S3 and SES policy with those permissions in it and then we'll attach it to this role when we're done

## ➡️ Step 9 - Creating a new policy with permissions for S3 and SES

1. Click "Policies" on left menu, click "Create policy"
2. Toggle over to the JSON editor, copy IAM policy code below and replace it to the policy editor, make sure to update your bucket name, if you named yours differently, mine is `jm-email-marketing`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::jm-email-marketing/*"
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


![Screenshot 2024-06-15 180400](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/b03fe2bc-05fe-42f7-930c-30f538693c14)



N.B: mAKE sure you leave the `/*` at the end your bucket name but just update your bucket name. This basically says we're allowing get object so getting things out of the S3 bucket and then also we want to allow sending emails and sending raw emails which is the HTML code through SCS.


3. When you click on "Next", name your policy `LambdaS3SESPolicy`, then click "Create policy"


![Screenshot 2024-06-15 180530](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/3008aa56-e60e-4f2b-9220-8ae48eea8480)


4. Now the policy exists we just need to go back to the execution Role and attach it

![Screenshot 2024-06-15 180605](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/996bee1c-1a0f-40ec-81fb-7ede77aa6127)


5. Select the one we created `LambdaS3SESPolicy`
   

![Screenshot 2024-06-15 180633](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/0d8e839d-c756-4362-935d-4b9e68b7cd92)

✔️ The Lambda function should have the permissions it needs to talk to S3 as well as SCS

6. Now let's run the test event for the Lambda function again, back to the Lambda function, source code section, click on "test" button


![Screenshot 2024-06-15 181001](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/07178788-1fa7-4ed7-b47a-0d5434fb51ce)


✔️ this is looking better, I'm not getting an access denied, as you can see the email is sent to the two different email
addresses.

Let's check my inbox:


![Screenshot 2024-06-15 181031](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/ae270d57-8606-408b-ad84-e0e500fcc6b1)


✔️ It's filled in my first name that was the one placeholder that we had everything else is the same from the template, it worked perfectly 


## ➡️ Step 10 - Trigger the Lambda function to send emails on a schedule

We need a way to trigger sending those emails, in other words we need to trigger the Lambda function. There's lots of
ways that you can do that with Lambda you could trigger it when you upload a new email template to the S3 bucket, you could trigger from the console from the CLI, we're actually going to use EventBridge which used to be called cloudwatch events but with this you can set up a schedule to trigger it say weekly or daily or whenever you want to send your emails.

1. Create a new schedule using EventBridge:

A. To create an EventBridge, navigate to the console search for EventBridge, then select "EventBridge schedule"


![Screenshot 2024-06-15 181052](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/5662c368-821d-4aa5-8263-5b703f9a92ac)

B. For schedule name, i will name it `SendWeeklyEmail`
C. And then scrolling down you choose your schedule pattern so you can do this just one time or you can
set up recurring schedule like every week or every month:
* Based on the name of mine I intend to do this weekly but just for testing purposes let's do a one-time schedule, 
* I'll select today's date and then the time, 
* Select the time zone it's default to the local one which is great 
* And then the flexible time window this basically gives it some leeway basically a window in which it can run I'm going to turn that off and say I want it to run at exactly this time and then click "Next"


![Screenshot 2024-06-15 181202](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/1eb15317-5a2f-40cf-9c64-054eb7625a8a)


D. The target there's lots of options to chose from but basically we want to invoke our Lambda function

Tip: If you don’t see your function, make sure you’re in the same region where you created the function

E. Select the Lambda function, mine was `SendSESEmailToContacts`, then click "Next"


![Screenshot 2024-06-15 181245](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/34b29deb-8afd-41f2-9871-4895667b8329)



F. The schedule should be enabled by default, if you want to take any action after the schedule completes like deleting that will delete your schedule I'm going to select "none"
G. You can just leave everything the default, then click "Next", 


![Screenshot 2024-06-15 181449](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/266062ea-1a19-4212-abc3-524a0cbca1d6)


H. Review all, the click "Create schedule"
G. In few minutes it should trigger the Lambda function, the Lambda function should send us an email to your inbox right on schedule that you created.

Let's just go take a quick look at what happened and how you can debug if you didn't get the email like you thought you should:
* Back in the EventBridge, clikc the target tab then click the target link

![Screenshot 2024-06-15 181512](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/8a940f6b-c57b-48f0-a046-548bccd2ae73)

* That'll open up a new tab and then in Lambda if you come into the monitor tab this is where you can see what's going on with this particular function, then click "cloudwatch logs" 

✔️ If you check our log streams, if we look at the latest one, I've got success messages, I sent two different emails


![Screenshot 2024-06-15 181544](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/6786c212-0eff-439a-a9c6-58dfb5e74a35)

![Screenshot 2024-06-15 181616](https://github.com/8130092157/Email_Marketing_App_AWS/assets/75124490/1556e5c0-9cd4-42df-bc75-3b540aed722b)


## 🏆 CHALLENGE: Ideas to enhance the application!

* Incorporate DynamoDB to hold names and email addresses (instead of a CSV file)
* Build a UI you can use to trigger sending the emails (use API Gateway)


## 💰 Cost

All services used are eligible for the AWS Free Tier. However, charges will incur at some point so it's recommended that you shut down resources after completing this tutorial.
