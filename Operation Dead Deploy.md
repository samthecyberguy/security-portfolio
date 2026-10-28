# [Title, outcome-flavored: "Investigating an Identity Attack in Entra ID"]
 Investigating an Unauthorized Provisioning in an Azure Cloud Subscription

## Scenario - 2 to 3 sentences. What was the situation and what question did the investigation answer? Frame it like a work ticket, not homework.
A user was given temporary contributor access to a cloud subscription. They deployed a test enviornment and didn't follow the organization's governance standards thus creating a resource with unauthorized naming conventions. My role, as the Azure on call engineer, is to investigate how this happened and how to prevent this from happening in the future.


## Environment
One list: platform, services, tools, access level. Honest framing: "live multi-user Azure training tenant, Reader access."
This is a live multi-user Azure training tenant, with Reader access. 

## Investigation - The core. Numbered steps IN YOUR OWN WORDS: what you looked at, what you found, what you concluded at each step. 6 to 12 screenshots of meaningful moments (portal views, query results, before/after).

1. Looked inside of the Resource Manager's list of resource groups. All of the resource groups should follow Microsoft's standard naming convention ( "rg-", workload identifier, enviornment indicator, region, and instance number). Discovered that one resource group broke the naming convention. 

![Operation Dead Deploy - Step 1](<lab-screenshots/1- Operation Dead Deploy - Step 1 - Screenshot.jpg>)

2. Clicked into the Resource Group created by the user and discovered one resource created. Opened the resource and looked at the Tags blade. Noted that the user followed their instructions correctly here and created a tag containing a string used when documenting test resources. 

3. Went back to the Resource Group's overview and clicked on the Deploymnents blade. Located the deployment the user created and noted that it as well did not follow proper naming conventions. 

4. The subscription already has a policy assignment that enforces naming conventions, yet the user was able to bypass it with the resource group and resource inside of it. Navigated to the resource group and clicked on the Policies blade to confirm. 

5. Noted that the naming convention policy is labeled as "Non-compliant". The policy is still providing alerts for non-compliant actions but not blocking these actions from occuring. 

6. Clicked on "Assignments". There are four Initiative Assignments and one Policy Assignment for naming conventions. 

7. Clicked on the policy assignment. Under the parameters tab the Effect parameter is set to "Audit". The user mistakenly set the naming convention policy as "Audit" instead of "Deny", which would block the non-compliant action from occuring. This is how the resource group and resource created by the user were able to be created despite going against the policy. 

## What broke / what surprised me - The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.



## Findings and recommendations
What you determined, plus 2 or 3 recommendations as if you were reporting to the resource owner.

## What I learned
3 to 5 bullets. At least one technical, one "what I'd do differently."
