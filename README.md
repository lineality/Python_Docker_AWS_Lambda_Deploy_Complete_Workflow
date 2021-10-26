Under Construction

```
TODO
	- may be link to a git for making an serializing a basic model in sklearn
	- try using environment variables in docker to use venv-env...
	https://bobcares.com/blog/activate-python-virtualenv-in-dockerfile/ 
	- test: uploading env to docker...
	- test: adding (copy?) directory to docker
- best way to move a folder
	- copy folder?
	- zip and move object then unzip?
	- 
- input test 
	- see if alpine container starts more quickly than default?

	https://aws.amazon.com/blogs/machine-learning/using-container-images-to-run-tensorflow-models-in-aws-lambda/ 
	- maybe section on TF-lite and find way to use TF-docker by adding aws interface to it?
	- or...

- Experiments to run:
		- try TFlite
		- try import pre-made env
			- Try mini-test with simple script and libraries etc.
		- try script 
	- find the alpine script...
https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support
- add line in docker file to copy-model
	- basic model deploy 
	- custom model deploy 
	- slimmer model? 
	- include a TFlite example
		- include an alpine linux container?
	- instructions for adding MFA to root account...
	- include a sklearn example
	env:
	- for python, what is the file structure?
	- experiment around with package library install requirements,
		- venv
		- raw pip install?
		- format of requirements file?
```

### This article/resource is one in a series of inter-related topics (for creating, deploying, maintaining, and collaborating around, Machine Learning and data science models and analysis):
1. Model Development Workflow
2. Model Documentation
3. Model Deployment 
#### The topic here is Model Deployment 

# Python_Docker_AWS_Lambda_Deploy_Complete_Workflow
The goal here is for this guide to be a complete guide through every step for deploying a python docker image (such as a machine learning model) via AWS (Amazon Web Services). This process is entirely done using AWS through the web-console (so no local software installations  needed on your local computer, no special computers, special operating systems, special system specs, special software etc. Any browser on any computer should work.), including the required best-practice-security steps for setting up users, groups and permissions. The code development environment for this project is AWS-Cloud9. Being able to deploy a Machine Learning Model with an endpoint for access is a basic requirement for many applied projects and research projects, yet clear and complete instructions for such a basic and required process are too difficult to find. Hopefully this guide will be helpful for students, researchers, business persons, administrators, etc. 

# Overall Description:
#### Using only AWS tools via web (no local software installs needed), deploy Machine Learning models to AWS Lambda for an external (or internal) restful-API-endpoint by uploading a Docker container to AWS-Lambda using Cloud9, S3, AWS Lambda, IAM, and API Gateway. 

# AWS Services Used: 
This project uses the following AWS services. (Your projects may use more services. Note: You may need to use multiple permission-groups as each group can only have a limited number of permissions.)
- AWS cloud9
- EC2 for Cloud9: aws linux 2 ec2
- AWS Lambda: docker container upload method
- S3: for file storage, including models, containers, etc.
- IAM: for roles/users/groups that can access and use the AWS services

# Overall Workflow Outline:
1. Log into appropriate AWS account (set up account and permissions if need be)
2. Set up files in (AWS)Cloud9 editor/IDE
3. Make a docker image
4. Test Docker image
5. Deploy Docker to storage in AWS (ECR)
6. Add Docker Image to (AWS)Lambda
7. Add endpoint with (AWS)API-Gateway

# Tips and Notes:

## DO make a new one. DON'T modify an old one.
#### As a rule of thumb: DO make a new one. DON'T modify an old one.

Like (fragile) python environments (which are best discarded and then recreated rather than tinkered with), the AWS process of docker deployment is buggy-glitchy, and prone to explode in random error messages. This makes interpretation of errors less clear. It is often best to try a process a few times completely from scratch when interpreting error messages (which often will be bugs and not problems with the code). Even the AWS Lambda, in theory you can add new images to an old Lambda, but in reality the changes of creating some strange and misleading error that a clean-build will not generate is too high. Even within cloud9: sometimes the terminals go buggy and you need to close and then open new terminal just to run a command. For a more enlightened future, we can all code well today so that future-people are not drowning in a sea of broken code garbage. Until that utopia comes about, the code we have explodes for any or no reason: so keep your process clean. 

#### Note:  As is typical with AWS, there are many ways to carry out a project, but even so the documentation is usually very poor and partial, requiring a combination of working bits from many sources with lots of 'filling in the gaps' about required steps and details that were never mentioned. This documentation attempts to be 100% complete as a step by step guide.

# Best Practice
Along with security, another best practice when using AWS is to delete any old items that you are finished with (in part to avoid being charged fees to keep them active in your account). Part of removing items can be clear naming of items: including a date-time in the title of your item can make it easy to see what you are deleting. Some resources for AWS are free or very cheap, but it is best practice to not leave old items around, especially if you may be paying for something you will never use again. 

## Naming
#### Naming files, directories(folders) and variables in common-sense ways that will make sense to future-you and other people is a create best-practice to follow. e.g.
- initials of person who made it: gga
- what aws service it is: cloud9
- date-time: 2021-09-24-1251
- use/purpose of item: py-docker-test
```
# Sample file name:
gga-py-docker-test-2021-09-24-1251-cloud9
GGA-2021-10-26-10-48-06-osha
```

#### Note: You do not need to type in the "$" symbol. The "$" included in terminal command text here should be excluded when you type it in. The symbol is shown here to indicate that the text is a Terminal command.

- Note: yum vs. apt
AWS Linux2 uses a YUM package manager, as does redhat, fedora, etc. 
NOT like debian, ubuntu etc (which uses apt)	
To update EC2 container, run this code IN TERMINAL at bottom of interface 

### Setup:
Windows that you may want to have open:
- https://console.aws.amazon.com/console/ 
- https://console.aws.amazon.com/cloud9/ 
- https://console.aws.amazon.com/ecr/
- https://console.aws.amazon.com/lambda/ 
- https://console.aws.amazon.com/apigateway/ 


# Follow these steps 
## To create, upload to and deploy python code in a Docker image on AWS with endpoint access:

## Step: create (a root) AWS account https://console.aws.amazon.com/iamv2/home?#/users

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
- Choose the "Security credentials" tab. 
- Next to Assigned MFA device, choose "Manage"
- Choose "Virtual MFA Device"
- On your phone: Use an MFA application such as "Google Authenticator" https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en_US&gl=US; follow the instructions, scan the QR code, etc. (Easy setup.)


## Step: add aws cloud9 to chrome browser 3rd party cookies
#### If using the chrome browser, you may need to add "https://console.aws.amazon.com/" to the list of all cookies allowed domains. settings -> cookies -> add whitelist 3rd party etc.


## Step: log into aws console using the role you just created:
IAM user login will need 3 items:
- 12 digit account number
- user name
- password
#### Note: you will need to have those 12 digits on hand later when uploading the Docker image to AWS-ECR.


## Step: Create a Cloud9 coding environment 
#### You can use the default options.
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


## Set up your Cloud9 editor:
#### Here is a cheatsheet-code for steps that are explained individually below. As you may find yourself making and testing many many docker images over and over again (just like you will do with pip-environments for some projects) here is a one-liner to set up the files structure do check for updates you will need:
- Update container packages via terminal command: $ sudo yum update
#### You may need to enter 'Y' to install updates if there are any.
- Update pip (the python package manager): $ pip install --upgrade pip
- Create New Folder (Directory) for your project: $ mkdir app
- Change directory (cd) your terminal into that new directory: $ cd app
- Create new file (for python code) for your project: $ touch app.py

### Here is a one-liner to carry out the above steps all together (or you can manually do each separately):
```
$ sudo yum update; mkdir app; cd app; touch app.py; touch Dockerfile; touch requirements.txt
```

#### See pic of what your file structure should look like:
#### Note: click on the new app folder (directory) to show what is inside.



## Step: Add your code into the python file. Note: because this is for AWS lambda, it will need all the code for a python-aws-lambda. The hello-world sample is very minimal. When you do this for the first time it is advisable to use a simple hello-world printing program, and gradually repeat the whole process using less-simple python programs: e.g.
- 1st: print hello world
- 2nd: use some kind of pip requirements (e.g. a date-time printout)
- 3rd: do an actual machine learning docker with sklearn or TFlite and a premade simple model (but still that just prints out so you can test the docker easily)
- etc.

#### Note: make sure the 'app' folder(directory) in the GUI is selected to show what is inside, or you will not see what is inside.

#### Add your code to app.py
- double click on new file to open in editor (cloud9)
- paste in code;
- This code prints a timestamp and tests out whether imported libraries are working.
```
import json
import boto3
from datetime import datetime

def handler(event, context):
    # timestamp
    timestamp = datetime.now()

    output = f"Hello, Timestamp: { timestamp }"

    return {
        'statusCode': 200,
        'body': json.dumps( output )
    }
```
#### Another super simple example:: 
```
def handler(event, context): 
    return 'Hello, world'
```

#### This may be useful in testing, it prints the python packages pre-installed in that docker (which might be causing conflicts with new installs):

```
def handler(event, context): 
    print("These are the pre-installed modules and libraries:")
    return print(help("modules"))
```

#### and more here: https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/

### Sample TEST Code for sklearn prediction and imported model with with terminal output for testing in cloud9 terminal (without needing to use ECR & aws-Lambda to test)
```
# for testing
import json
from joblib import dump, load
import boto3
 
"""
only joblib is needed to import
but sklearn (scikit-learn) must be installed
"""
 
def handler(event, context):
 
   ##############################
   # Load Machine Learning Model
   ##############################
   # load the picked/serialized ML model into the system
   regresssion_model_sklearn = load('your_model_name.joblib')
   # regresssion_model_sklearn_1 = load( response['Body'] )
  
   ##################
   # make prediction
   ##################
   prediction = regresssion_model_sklearn.predict([[ 5 ]])
  
   # format output json object
   output = {}
   output["model_prediction"] = str(prediction)
  
   return {
       'statusCode': 200,
       'body': json.dumps(output)
   }
```
### Sample Code for sklearn prediction and imported model AND user input for prediction
```
import json
from joblib import dump, load
import boto3
 
"""
only joblib is needed to import
but sklearn (scikit-learn) must be installed
"""
 
def handler(event, context):
 
   ##################
   # read the inputs
   ##################
   try:
       input_value = float(event["input_value"])
 
       # inspection
       print("input format ok")
 
   except (Exception) as error:  # Failure!!
       output = (f"""Error: There was an input format problem:
               input_value is a number. {error} \n""")
       # inspection
       print(error)
 
 
   ##############################
   # Load Machine Learning Model
   ##############################
   # load the picked/serialized ML model into the system
   regresssion_model_sklearn = load('your_model_name.joblib')
   # regresssion_model_sklearn_1 = load( response['Body'] )
  
   ##################
   # make prediction
   ##################
   prediction = regresssion_model_sklearn.predict([[ input_value ]])
  
   # format output json object
   output = {}
   output["model_prediction"] = str(prediction)
  
   return {
       'statusCode': 200,
       'body': json.dumps(output)
   }
```


#### Note: lambda_handler vs. handler: The normal code for a python lambda (AWS) is that the function needs to be called "lambda_handler" but it appears that when AWS-Lambda is using a docker-image, the name of the function must be "handler"

## Step: install your packages, libraries, dependencies:
TODO
While this super-simple hello-world demo does not have any installed requirements, likely the whole point of using Docker+AWS-Lambda is that you need your python-lambda to have a set of required packages (and perhaps files) that otherwise would be infeasible or impossible to connect to a normal (non-docker) AWS Lambda. 
```
python3 -m pip install boto3
pip install scikit-learn
```

## Step: Create a file called "requirements.txt" to list your python requirements.
Two ways to do this are to create one based on files already installed in your environment, or another way to do this is to simply manually write a requirements.txt doc yourself. 

#### You may want to separate your process of setting up your project and files, designing what packages you need, etc. vs. the process of creating and uploading your docker-image. For example, if you are just uploading a docker image then you will not actually be installing any pip packages in your cloud9 ide (most likely). 

#### To make a doc based on python packages you have installed:
- Run in a Terminal
```
$ pip3 freeze > requirements.txt
```
- Remove unneeded items: You may need to or want to Remove unneeded items from the list, perhaps consulting a venv  requirements.txt from a model or tool set in a local directory. 
Based on the AWS documentation it appears that you need a requirements.txt file even if that file is empty, and that it only 'needs' to contain packages/libraries/dependencies that are required by you for your project.

Here is an example of a requirements file for a project involving just sklearn (which is also called "scikit-learn" https://pypi.org/project/scikit-learn/ and AWS's boto3. The other libraries on this list are installed along with (mostly) sklearn. 
```
boto3
scikit-learn
```

## Note: 
#### The list of "installs you do" and the list of "installed" are not necessarily the same, and sometimes one will not work but the other will. E.g. All you need to install is "scikit-learn," which automatically includes installing other packages (which are then many 'installed' packages) but if you try to use the 'installed' package list as your requirements(to install), then everything will break. 

#### Odd message from AWS. It says to use ENV but the docker steps don't describe how to do that. and...the terminal says I am using 21.2.4 and cannot upgrade...
```
WARNING: Running pip as root will break packages and permissions. You should install packages reliably by using venv: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.1.1; however, version 21.2.4 is available.
You should consider upgrading via the '/var/lang/bin/python3.8 -m pip install --upgrade pip' command.
```


#### Note: For the timestamp example used here, the only library needed, the only text needed in the requirements file is: "boto3" or the version used such as "boto3==1.18.44"

#### Note: The Mysterious ${LAMBDA_TASK_ROOT} Directory
On several pages (see: https://docs.aws.amazon.com/lambda/latest/dg/images-create.html & https://docs.aws.amazon.com/lambda/latest/dg/python-image.html) the AWS documentation says this: 
"Install any dependencies under the ${LAMBDA_TASK_ROOT} directory alongside the function handler to ensure that the Lambda runtime can locate them when the function is invoked."
It may be that this is AWS's inexplicably obscure way of saying: files you need to be included in the docker image must be in the same directory(folder) along with the docker file. But this is NOT an instruction regarding python dependencies, even though these statements are made in a context of python, which is highly irresponsible documentation writing practice. But perhaps a directory given the very odd name "${LAMBDA_TASK_ROOT}" could be created and files put inside? ....the situation is very unclear.

#### Note: The docker image does not automatically include all the files in the directory. You need to specify in the Dockerfile what files you want put where in the file docker container.

## Step: Add text to Dockerfile
#### Note: This first example adds one part to the default-aws-dockerfile script: updating pip
If you follow the AWS docs Dockerfile, you will get a warning that pip should be upgraded. This addition 'fixes' that. 
```
## Update pip
# /var/lang/bin/python3.8 -m pip install --upgrade pip
```

- double click on new file to open in editor (cloud9)
- paste in code: (Example working Dockerfile code from AWS docs, this code works):
#### If you have no additional files to add:
```
FROM public.ecr.aws/lambda/python:3.8

## Update pip
RUN  /var/lang/bin/python3.8 -m pip install --upgrade pip

# Copy function code
COPY app.py ${LAMBDA_TASK_ROOT}

# Install the function's dependencies using file requirements.txt
COPY requirements.txt  .
RUN  pip3 install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
CMD [ "app.handler" ]
```
#### If you add files such as a serialized (pickled) model:
#### Note: if you renamed your model, you need to change that name here too.
```
FROM public.ecr.aws/lambda/python:3.8

# Copy function code
COPY app.py ${LAMBDA_TASK_ROOT}

# Copy serialized (pickled) model
COPY your_model_name.joblib ${LAMBDA_TASK_ROOT}

## Update pip
RUN  /var/lang/bin/python3.8 -m pip install --upgrade pip

# Install the function's dependencies using file requirements.txt
COPY requirements.txt  .
RUN  pip3 install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# Set the CMD to your "handler"
CMD [ "app.handler" ]
```


#### TODO: Using venv in docker?

#### Note: Below, the name for this docker image project used here is hello-world. If you change that, changes in various steps of the process (names, commands, etc.) must also be changed. 



## Step: upload your model, if you have one:
#### drag you local model file to the specific folder (not into the void of space in the side panel)
#### or use menu options to upload:


#### After selecting upload files from the files-menu, drag files into the pop-up box (see pic)



## Tip Note:
#### One recommendation is to make a simple test version of your application, one which does not take inputs but will simply output a sample output, so that you can test your docker container in Cloud9 without needing to take the time to install into lambda to test there.

## Step: Create Docker Image
#### Note, again, it is advisable to treat this whole cloud9 process as build-once-disposable part of your process. If you try to run this docker-build command more than once it will likely generate more and more errors. Recommended: only run build once. Make a new cloud9 to build again. 

- Run this terminal command in the same directory(folder) that contains the Dockerfile:
- Type this into the terminal (with the trailing space and period included)
```
$ docker build -t hello-world .
```
#### Check the output of this to make sure the build works completely. You will likely get some pip-warnings no matter what you do, I think you can ignore those. At the end it should say:
```
Successfully built ###########
Successfully tagged hello-world:latest
```

#### Note: If previous steps were not done in the correct terminal, directory, etc., then this build will not work. 

## Now Test Your Docker Image
#### Before deploying and using your docker image, you will want to test it to make sure it (or a test-version of it) works. Next we will test out the docker image to make sure it works, which will require a few steps using the original terminal AND a new cloud9 terminal_2, which you will open.

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
#### Note: If you need to make a change to your files and re-upload, your best bet may be to delete your stored image from ECR e.g. https://console.aws.amazon.com/ecr/repositories?region=us-east-1 
and repeat the above four terminal commands to send a fresh version to ECR

## Step: Make an AWS-Lambda-Function, follow these steps:
- In a new browser tab go to a new AWS service: AWS Lambda https://console.aws.amazon.com/lambda/ 
- Select: container image (as type along top of GUI) vs. (not) "Author From scratch"
- Fill in the two fields under basic information (see pic)
- Function Name -> Make up a function-name
- Container image uri -> browse images -> pick the named you saved it to (e.g. hello-world)
- Press the orange button that says "Create Function"






#### Note: You can change or update the docker-image assigned to a given lambda-function by using: image -> Deploy new image(button) (see pic)






## Edit your Lambda: Make Timeout longer





## Step: Best Practice: Delete old items from AWS
When you are done using your ECR image, delete it using these instructions:
#### Source: Deleting an ECR image
#### https://docs.aws.amazon.com/AmazonECR/latest/userguide/delete_image.html



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
Two of the main python tools for machine learning (related to each-other) are sklearn (also called SciKitLearn) and Tensorflow (TF) (and TFlite or Tensorflow Lite). There are many articles online that explain how they relate to each-other. For our purposes here: TFlite is very small, Tensorflow is small, and Sklearn is bigger. As yet, I cannot recommend a way to use Tensorflow with AWS - work in progress.


## Runtime Note:
#### The first time the container runs with sklearn it may take more seconds (e.g. 5.1 seconds in one case) than the default time limit, which you can reset to longer in: configuration-> general configuration -> edit -> timeout -> set to 1 minute. 
#### But then after it runs the first time it only takes a fraction of a second to run again.


#  Sources
Sources used for this documentation are listed at the end of the document.

#### Source: AWS reInvent 2020 Run Lambda with Container Image | Tutorial & DEMO | Lambda and Kubernetes Dec 3, 2020 Agent of Change (Overall: an excellent resource)
#### https://www.youtube.com/watch?v=HNm6jU_AUbE

#### Source: Enabling a virtual multi-factor authentication (MFA) device (console) 
#### https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html#enable-virt-mfa-for-iam-user 

#### Source: Deleting an ECR image
#### https://docs.aws.amazon.com/AmazonECR/latest/userguide/delete_image.html

#### Source: New for AWS Lambda – Container Image Support, Danilo Poccia, DEC 2020 (Overally: pretty bad AWS documentation, better than usual)
#### https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/

#### Source: Basic AWS Lambda Project Creating Docker Image (Overall: horrible AWS documentation as usual)
#### https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/lambda-creating-project-docker-image.html 

#### Source: deploy serverless machine-learning-models to aws-lambda (Overall, an OK resource using a different approach and more local resources)	
#### https://www.udemy.com/course/deploy-serverless-machine-learning-models-to-aws-lambda/ 

#### Source: Deploy Python Lambda functions with container images (which sites the next following article) (Focused on Python)
#### https://docs.aws.amazon.com/lambda/latest/dg/python-image.html 

#### Source: Create an image from an AWS base image for Lambda
#### https://docs.aws.amazon.com/lambda/latest/dg/images-create.html 
#### https://docs.aws.amazon.com/lambda/latest/dg/images-create.html#images-create-from-base

#### Source: Containerized Python Development – Part 1  https://www.docker.com/blog/containerized-python-development-part-1/

#### Source Copying Directories (e.g. where saved ML models are folders, TFlite)
https://stackoverflow.com/questions/28599571/add-or-copy-a-folder-in-docker 

end 

