#  Investigating an Unauthorized Provisioning in an Azure Cloud Subscription

## Scenario
A user was given temporary contributor access to a cloud subscription. They deployed a test enviornment and didn't follow the organization's governance standards thus creating a resource with unauthorized naming conventions. My role, as the Azure on call engineer, is to investigate how this happened and how to prevent this from happening in the future.

## Environment
This is a live multi-user Azure training tenant, with Reader access. 

## Investigation

1. Looked inside of the Resource Manager's list of resource groups. All of the resource groups should follow Microsoft's standard naming convention ( "rg-", workload identifier, enviornment indicator, region, and instance number). Discovered that one resource group broke the naming convention. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 1 - Screenshot.jpg"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

2. Clicked into the Resource Group created by the user and discovered one resource created. Opened the resource and looked at the Tags blade. Noted that the user followed their instructions correctly here and created a tag containing a string used when documenting test resources. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 2 - Screenshot.png"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

3. Went back to the Resource Group's overview and clicked on the Deploymnents blade. Located the deployment the user created and noted that it as well did not follow proper naming conventions. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 3 - Screenshot.png"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

4. The subscription already has a policy assignment that enforces naming conventions, yet the user was able to bypass it with the resource group and resource inside of it. Navigated to the resource group and clicked on the Policies blade to confirm. 

5. Noted that the naming convention policy is labeled as "Non-compliant". The policy is still providing alerts for non-compliant actions but not blocking these actions from occuring. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 4,5 - Screenshot.jpg"
       alt="Operation Dead Deploy - Step 1"
       width="600">
</p>

6. Clicked on "Assignments". There are four Initiative Assignments and one Policy Assignment for naming conventions. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 6 - Screenshot.png"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

7. Clicked on the policy assignment. Under the parameters tab the Effect parameter is set to "Audit". The user mistakenly set the naming convention policy as "Audit" instead of "Deny", which would block the non-compliant action from occuring. This is how the resource group and resource created by the user were able to be created despite going against the policy. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 7 - Screenshot.jpeg"
       alt="Operation Dead Deploy - Step 1"
       width="600">
</p>

## What broke / what surprised me - The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.
Even though there was a policy already in place for naming conventions, the user was still able to bypass it and create improper naming conventions. After entering the Resource Manager the issue was clear to see, as all of the other Resource Groups had "rg-" at the beginning of the name, and the one the user created did not. The more laborious part of this investigation was finding out why the user was able to do this. Eventually, after doing some digging, I was able to see the Effect was set to "Audit", which is why the system alerted but did not block the improper naming convention from occurring. 

## Findings and recommendations
I found that the user was able to bypass the naming convention policy because the Effect parameter for the policy was set to "Audit", instead of "Deny". To prevent this from happening again in the future, set the Effect parameter for this policy to "Deny", this will block improper naming conventions from occurring. Additionally,  


## What I learned
3 to 5 bullets. At least one technical, one "what I'd do differently."
