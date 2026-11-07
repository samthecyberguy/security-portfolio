# Investigating a Consent-Phishing Attack That Led to a Rouge App Registration in an Azure Tenant

## Scenario
Reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations. 

## Environment
This is a live multi-user Azure training tenant, with Reader access. 

## Investigation

1. Entry - An employee clicked on a phishing link, entered credentials and completed MFA, allowing the attacker to use session hijacking and bypass Conditional Access. The poor access control policies of the tenant gave the employee Owner permissions of a legacy connector app, thus giving the same access to the attacker who was able to hijack their session.  

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 1 - Screenshot.jpg"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

2. Escalate - The attacker was able to take advantage of the user's Owner permissions and create a client secret on the legacy connector app. This automatically elevated their access from that of a typical user to a highly privileged Service Principal, which include API permissions. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 2 - Screenshot.png"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

3. Pivot - When a client secret is changed, the old one no longer works anymore. So, the attacker registered their own app and added its' service principal to the legacy connector app's Owners list. This gives the attacker the ability to keep creating new client secrets. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 3 - Screenshot.png"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

4. Persistence - 

5. Noted that the naming convention policy is labeled as "Non-compliant". The policy is still providing alerts for non-compliant actions, but not blocking these actions from occuring. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 4,5 - Screenshot.jpg"
       alt="Operation Dead Deploy - Step 1"
       width="600">
</p>

6. Clicked on "Assignments". There are four Initiative Assignments and one Policy Assignment for the naming convention. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 6 - Screenshot.png"
       alt="Operation Dead Deploy - Step 1"
       width="400">
</p>

7. Clicked on the Policy Assignment. Under the Parameters tab the Effect parameter is set to "Audit". Due to the naming convention policy being set as "Audit" instead of "Deny", it does not block the improper naming convention from occuring. "Audit" only give an alert, while "Deny" blocks. This is how the resource group and resource created by the user were able to be created despite going against the policy. 

<p align="center">
  <img src="lab-screenshots/1- Operation Dead Deploy - Step 7 - Screenshot.jpeg"
       alt="Operation Dead Deploy - Step 1"
       width="600">
</p>

## What broke / what surprised me 
Even though there was a policy already in place for naming conventions, the user was still able to bypass it and create improper naming conventions. After entering the Resource Manager the issue was clear to see, as all of the other Resource Groups had "rg-" at the beginning of the name, and the one the user created did not. The more laborious part of this investigation was finding out why the user was able to do this. Eventually, after doing some digging, I was able to see the Effect was set to "Audit", which is why the system alerted but did not block the improper naming convention from occurring. 

## Findings and recommendations
I found that the user was able to bypass the naming convention policy because the Effect parameter for the policy was set to "Audit" instead of "Deny". To prevent this from happening again in the future, set the Effect parameter for this policy to "Deny". This will block improper naming conventions from occurring. Additionally, instead of giving the user temporary Contributor access for the whole subscription, give them access only to the resource groups they need. This follows the principle of least privilege.    

## What I learned
- Just because a policy is applied to a resource it doesn't mean that policy will block the improper action. It depends on the Effect parameter.
- Even something that seems insignificant, like using the wrong naming convention for a single resource, it can have a considerable impact down the line.
