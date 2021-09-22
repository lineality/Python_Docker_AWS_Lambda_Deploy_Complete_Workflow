# Python_Docker_AWS_Lambda_Deploy_Complete_Workflow
The goal here is for this guide to be a complete guide through every step for deploying a python docker image (such as a machine learning model) via AWS (Amazon Web Services). This process is entirely done using AWS through the web-console (so no local software installs a needed on your local computer, no special computers, special operating systems, special system specs, special software etc. Any browser on any computer should work.), including the required best-practice-security steps for setting up users, groups and permissions. The code development environment for this project is AWS-Cloud9. Being able to deploy a Machine Learning Model with an endpoint for access is a basic requirement for many applied projects and research projects, yet clear and complete instructions for such a basic and required process are too difficult to find. Hopefully this guide will be helpful for students, researchers, business persons, administrators, etc. 

TODO
	- basic model deploy
	- custom model deploy
	- slimmer model? 
	- include a TFlite example
		- include an alpine linux container?
	- instructions for adding MFA to root account...
	- include a sklearn example

- does the directory need to be named app? (yes?)
	- for python, what is the file structure? (WTF) 
	- experiment around with package library install requirements,
		- venv
		- raw pip install?
		- format of requirements file?

# Overall Description:
#### Using only AWS tools via web (no local software installs needed), deploy Machine Learning models to AWS Lambda for an external (or internal) restful-API-endpoint by uploading a Docker container to AWS-Lambda using Cloud9, S3, AWS Lambda, IAM, and API Gateway. 

## Note: 
#### As is typical with AWS, there are many ways to carry out a project, but even so the documentation is usually very poor and partial, requiring a combination of working bits from many sources with lots of 'filling in the gaps' about required steps and details that were never mentioned. This documentation attempts to be 100% complete as a step by step guide.

#  Sources
Sources used for this documentation include:

#### (Overall: an excellent resource)
#### 1. https://www.youtube.com/watch?v=HNm6jU_AUbE
#### AWS reInvent 2020 Run Lambda with Container Image | Tutorial & DEMO | Lambda and Kubernetes Dec 3, 2020 Agent of Change

#### 2. Enabling a virtual multi-factor authentication (MFA) device (console) 
#### https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html#enable-virt-mfa-for-iam-user 

#### (Overally: pretty bad AWS documentation, better than usual)
#### 2. New for AWS Lambda – Container Image Support, Danilo Poccia, DEC 2020
#### https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/

#### (Overall: horrible AWS documentation as usual)
#### 3. Basic AWS Lambda Project Creating Docker Image
#### https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/lambda-creating-project-docker-image.html 

#### (Overall, an OK resource using a different approach and more local resources)	
#### 4. deploy serverless machine-learning-models to aws-lambda
#### https://www.udemy.com/course/deploy-serverless-machine-learning-models-to-aws-lambda/ 

### (Focused on Python)
#### 5. Deploy Python Lambda functions with container images (which sites the next following article)
#### https://docs.aws.amazon.com/lambda/latest/dg/python-image.html 

#### 6. Create an image from an AWS base image for Lambda
#### https://docs.aws.amazon.com/lambda/latest/dg/images-create.html 
#### https://docs.aws.amazon.com/lambda/latest/dg/images-create.html#images-create-from-base



# AWS Services Used: 
This project uses the following AWS services. (Your projects may use more services. Note: You may need to use multiple permission-groups as each group can only have a limited number of permissions.)
- AWS cloud9
- EC2 for Cloud9: aws linux 2 ec2
- AWS Lambda: docker container upload method
- S3: for file storage, including models, containers, etc.
- IAM: for roles/users/groups that can access and use the AWS services

# Steps to create, upload to and deploy python code in a Docker image on AWS with endpoint access:

#### Note: You do not need to type in the "$" symbol. The "$" included in terminal command text here should be excluded when you type it in. The symbol is shown here to indicate that the text is a Terminal command.

## Step: create (a root) AWS account
https://console.aws.amazon.com/iamv2/home?#/users

## Step: create IAM user (not root)
#### IAM user will need 3 items
1. a set of login credentials:
- a 12 digit account number
- user name
- password

2. permissions:
- give permissions by creating a group that has these permissions and assign the user to that group: give IAM group/user these permissions:
- cloud9 user
- cloud9 admin
- ec2
- APIgateway invoke
- api gateway admin
- s3 
- AmazonEC2ContainerRegistryFullAccess (Elastic Container Registry)

## Step: Setup MFA for User
#### For long-term more-secure AWS login, set up MFA (Multi Factor Authentication) for the AWS user: The AWS Docs for this are actually accurate and useful.
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html#enable-virt-mfa-for-iam-user
#### Instructions paraphrased here: To enable a virtual MFA device for IAM user 
- Sign in to the AWS Management Console and open the IAM console 
- choose Users.
- double click on blue name of the intended user.
- Choose "Security credentials" tab. 
- Next to Assigned MFA device, choose "Manage"
- choose Virtual MFA device
- on your phone use an MFA application such as "Google Authenticator" https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en_US&gl=US, follow the instructions, scan the QR code, etc. Easy setup. 



## Step: Start up a Cloud9 coding environment:
- Go to aws
- Go to services
- Go to Cloud9
- Create new Environment
- Name environment, add description
- Environment TYpe: new EC2 
- Instance Type t2.micro (works)
- Platform: Amazon Linux 2 (works)
- Cost-saving settings: 30 min (works)
- IAM role: leave as default set by aws
- Network settings (advanced): ignore this
- Add new tag: ignore this
- Next step (hit this button)
- review specs and hit Create Environment (button)


## Step: log into aws console using the role you just created:
IAM user login will need 3 items:
- 12 digit account number
- user name
- password
#### Note: you will need to have those 12 digits on hand later when uploading the Docker image to AWS-ECR.

## Step: update container packages via terminal command: sudo yum update
(Default linux is not updated, so you may need to update all packages. Though of course there is a chance that for your particular project you need to update only some packages.)
- Note: yum vs. apt
AWS Linux2 uses a YUM package manager, as does redhat, fedora, etc. 
NOT like debian, ubuntu etc (which uses apt)	
To update EC2 container, run this code IN TERMINAL at bottom of interface 
```
$ sudo yum update 
```
- Type in Y or Yes to instal any updates.
#### Note:  The Cloud9 terminal uses control-v not control-shift-v to paste.
- update pip (the python package manager):
```
$ pip install --upgrade pip
```

## Step: Create New Folder (Directory) for your project
- create new folder called 
```
app
```


## Step: change directory (cd) your terminal into that new directory
```
$ cd app
```


## Step: create new file (for python code) for your project:
#### (note: if you are in the right directory, you can also make the new file by putting this command in the terminal: 
```
touch app.py
```
- Right click in your ''app' folder and select': new file
- file name: either way, there should be a new file in the directory(folder) that has this name:
```
app.py 
```


## Step: Add your code into the python file. Note: because this is for AWS lambda, it will need all the code for a python-aws-lambda. The hello-world sample is very minimal. 

#### Note: make sure the 'app' folder(directory) in the GUI is selected to show what is inside, or you will not see what is inside.

(type that in)
- double click on new file to open in editor (cloud9)
- paste in code;
- This code prints a timestamp and tests out whether imported libraries are working.
```
import json
import boto3
from datetime import datetime

def lambda_handler(event, context):
    # timestamp
    timestamp = datetime.now()

    output = f"Hello, Timestamp: { timestamp }"

    return {
        'statusCode': 200,
        'body': json.dumps( output )
    }
```
Alternate simpler Example code here: 
```
def handler(event, context): 
    return 'Hello, world'
```
#### and more here: https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/

#### Note: lambda_handler vs. handler: The normal code for a python lambda (AWS) is that the function needs to be called "lambda_handler" but it appears that when AWS-Lambda is using a docker-image, the name of the function must be "handler"

- note: python vs. node.js, it may be possible to hybridize node.js and python workflows, if a project calls for both: https://www.npmjs.com/package/python 

## Step: install your packages, libraries, dependencies:
While this super-simple hello-world demo does not have any installed requirements, likely the whole point of using Docker+AWS-Lambda is that you need your python-lambda to have a set of required packages (and perhaps files) that otherwise would be infeasible or impossible to connect to a normal (non-docker) AWS Lambda. 

## Step: Create a file called "requirements.txt" to list your python requirements.
- Run in a Terminal
```
$ pip3 freeze > requirements.txt
```
- Remove unneeded items: You may need to or want to Remove unneeded items from the list, perhaps consulting a venv  requirements.txt from a model or tool set in a local directory. 
Based on the AWS documentation it appears that you need a requirements.txt file even if that file is empty, and that it only 'needs' to contain packages/libraries/dependencies that are required by you for your project.
- 


#### Note: For the timestamp example used here, the only library needed, the only text needed in the requirements file is: "boto3" or the version used such as "boto3==1.18.44"

#### Note: The Mysterious ${LAMBDA_TASK_ROOT} Directory
On several pages (see: https://docs.aws.amazon.com/lambda/latest/dg/images-create.html & https://docs.aws.amazon.com/lambda/latest/dg/python-image.html) the AWS documentation says this: 
"Install any dependencies under the ${LAMBDA_TASK_ROOT} directory alongside the function handler to ensure that the Lambda runtime can locate them when the function is invoked."
It may be that this is AWS's inexplicably obscure way of saying: files you need to be included in the docker image must be in the same directory(folder) along with the docker file. But this is NOT an instruction regarding python dependencies, even though these statements are made in a context of python, which is highly irresponsible documentation writing practice. But perhaps a directory given the very odd name "${LAMBDA_TASK_ROOT}" could be created and files put inside? ....the situation is very unclear.


## Step: Create Dockerfile
Create a new file for your project.
- right click in your ''app' folder and select': new file
- file name: 
```
Dockerfile 
```
(with the first letter capitalized)
- The icon next to the file in Cloud9 should become the docker-whale icon after the file is named 'Dockerfile.'
- Note: you can also use 
```
$ touch Dockerfile
```
in the terminal to make the file.


## Step: Add text to Dockerfile
- double click on new file to open in editor (cloud9)
- paste in code: (Example working Dockerfile code from AWS docs, this code works):
```
FROM public.ecr.aws/lambda/python:3.8

# Copy function code
COPY app.py ${LAMBDA_TASK_ROOT}

# Install the function's dependencies using file requirements.txt
# from your project folder.

COPY requirements.txt  .
RUN  pip3 install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
CMD [ "app.handler" ]
```

#### Note: Below, the name for this docker image project used here is hello-world. If you change that, changes in various steps of the process (names, commands, etc.) must also be changed. 

## Step: Create Docker Image
- Run this terminal command in the same directory(folder) that contains the Dockerfile:
- Type this into the terminal (with the trailing space and period included)
```
$ docker build -t hello-world .
```
Note: if previous steps were not done in the correct terminal, then this won't work. 
You should still be in the 


## Note: Next we will test out the docker image to make sure it works, which will require a few steps using the original terminal AND a new cloud9 terminal_2, which you will open.


## Step: Run your Docker container:
- (still in terminal_1) Run this code in terminal:
```
$ docker run -p 9000:8080 hello-world:latest
```

## Step: Open a NEW second terminal (here called "terminal_2")
- select plus symbol (green) in the terminal section at the bottom of GUI
- select 'New Terminal'; a "terminal_2"should now exist

## Step: check docker with 'curl' (look for status:200)
- In the NEW terminal, terminal_2, run this code:
```
$ curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
```
- you should see the default function output (if there is any) e.g. "Hello, world."
- if you see "hello, world" printed in the terminal, then your test is passed! Your docker container ran, ran the code, and produced the correct output.


## Step: Stop the Docker image from running (with ctrl+C)
- Go back to your original Terminal 1 (which is running the Docker image)
- Press: ctrl + C (the control key and the c-letter key at the same time) to terminate the docker test.


#### Note: You are now done making and testing your Docker image! This is progress.


## Deploy to AWS and make an endpoint: 
Next will be the steps to deploy your container to AWS-lambda and use AWS-api-gateway to make an endpoint to 'hit' your model with a prediction request. (or in this sample, hitting the endpoint that prints 'hello, world')


#### Step: Store your Docker Image (in AWS) in the ECR: Elastic Container Registry, so that the docker image can be used later by AWS services. (Create the repository, push image to repository)
- (back in terminal 1)
- install docker image by running these lines:
- BUT: replace 123412341234 with your aws number (the 12 digit number you logged in with)
- & replace us-east-1 with your region (if it differs)
```
$ aws ecr create-repository --repository-name hello-world --image-scanning-configuration scanOnPush=true
$ docker tag hello-world:latest 123412341234.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest
$ aws ecr get-login-password | docker login --username AWS --password-stdin 123412341234.dkr.ecr.us-east-1.amazonaws.com
$ docker push 123412341234.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest
```

## Step: Make an AWS-Lambda-Function, follow these steps:
- In a new browser tab go to a new AWS service: AWS Lambda https://console.aws.amazon.com/lambda/ 
- Select: container image (as type along top of GUI) vs. (not) "Author From scratch"
- Fill in the two fields under basic information (see pic)
- Function Name -> Make up a function-name
- Container image uri -> browse images -> pick the named you saved it to (e.g. hello-world)
- Press the orange button that says "Create Function"






#### Note: You can change or update the docker-image assigned to a given lambda-function by using: image -> Deploy new image(button) (see pic)



## Step: Make an endpoint with AWS-API-gateway: Follow these steps:
- go to another aws service, api-gateway: https://console.aws.amazon.com/apigateway
- create api (push orange button)
- select type of endpoint: rest api -> build
- fill it name at: API name*
- blue button push -> Create API
- actions -> create method -> pull down  menu -> post
- click on grey check mark
- type in name of your lambda function in slot for name of lambda function
- push blue 'Save" button 
- (lgihting bolt symbol) test -> test -> inspect output
- actions -> deploy_API -> Deployment stage -> new_stage -> Stage name* -> select or make one
- push the blue deploy button
- Save "invoke URL" This will appear along the top of the browser window. Save it now as it can be tricky to find out later. (This uses a default AWS-made-up endpoint name, if you want a proper custom-named endpoint URL, that is a more elaborate (and expensive) process. This will work in a pinch, but a custom URL may be needed in the long term.)
- Fake example: " Invoke URL: https://vrtsdhdhdhdrhdr.execute-api.us-east-1.amazonaws.com/dev "







# Machine Learning Models
Two of the main python tools for machine learning (related to each-other) are sklearn (also called SciKitLearn) and Tensorflow (TF) (and TFlite or Tensorflow Lite). There are many articles online that explain how they relate to each-other. For our purposes here: TFlite is very small, Tensorflow is small, and Sklearn is bigger. 

