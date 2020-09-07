---
layout: post
title: Community Registration OAuth Autologin
---

Community is a great tool to use as IDP for mobile or web apps alike, to provision users and enable your external apps connected to Salesforce authentication and security.

We were trying to do exactly this but run into this issue with the external Heroku app using [OAuth 2.0 Web Server Flow](https://help.salesforce.com/articleView?id=remoteaccess_oauth_web_server_flow.htm&type=0) to Register and auto authenticate users. It is the first step in the user provision process that we want to remove as many barriers as possible but remain reasonably secure for our users.

### Problem

Self Registration Ignores startURL and does not redirect the user correctly to our web app after the registration step.

We have a customer Community setup on our sandbox to enable user self-register and Authenticate with Salesforce Communities as IDP to our Connected App - a web application that is hosted on Heroku. The login process works correctly in this solution, but Self-registration does not. When user self-registers using our custom Visual Force page or Lightning Component (even using Salesforce existing examples of controller and pages) we create Person-Account and new user correctly. After a user is created we try to autologin using `Site.login` API in APEX controller it has `startURL` parameter to return the access token and redirect the user to that start URL. Instead of registration it always redirects the user to the Community home page as the default action. 

Important to note this only happens if our connected app is set to Admin pre-authorize all users and Relax IP restrictions on connected app policy, We want to avoid default popup for Self Authorization screen. If a connected app is set to All users can self-authorize then `startURL` redirect works correctly and returns the token and user to our Heroku app page after user registers and automatically login. It seems strange that the connected app setting has a different outcome for Registration redirect. I started to think that is a Site API bug.

### Solution

After some investigations and digging in documents, posting to the community, etc. I was able to find a solution to this problem that turned out to be constructing a startURL with client ID - Connected App consumer Key value.

During standard login this key is present but in Self-registration it needs to be added by the controller.
The Start URL should be the OAuth URL for our Flow (Web Server flow) with its parameters.

The site.login() method has 3 parameters:
1. Email/user
2. Password
3. startURL

It is the 3rd parameter `startURL` where we need to use the OAuth URL along with its parameters so that it will automatically SSO and redirect to the Heroku app.

Can you please try to change the startURL from Heroku URL to the one explained in the below article:
https://developer.salesforce.com/page/Digging_Deeper_into_OAuth_2.0_on_Force.com#Obtaining_a_Token_in_an_Autonomous_Client_.28Username_and_Password_Flow.29

We will need to POST the request to "https://login.salesforce.com/services/oauth2/token" URL along with the below-mentioned parameters:

+ response_type Set this to `code`.
+ client_id Your application's client id connected app consumer key.
+ redirect_uri  set to Heroku app authorization

Example URL	    

```
https://<community instance URL>/services/oauth2/authorize?response_type=code&client_id=<connected app Consumer key>&redirect_uri=https%3A%2F%2Fmyapp-web-dev.herokuapp.com%2Fauthorized
```

That is all we need to do to get our redirect to work. There is however another interesting detail I needed to resolve is how to get a consumer key from the connected app definition? That I was not yet able to get from metadata, there is no API to read this from APEX. I created an Idea for this.

IN the interest of time I solved this by storing consumer key in Custom Settings for each environment DEV, QA, PROD, etc, that way it is more flexible.

 
