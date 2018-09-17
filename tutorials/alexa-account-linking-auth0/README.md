# How to set up Account Linking for Alexa with Auth0 and Jovo

In this guide, we will show you how to set up **Account Linking for your Amazon Alexa Skill**, without having to deal with all the security issues and the burden of running your own OAuth 2.0 server. We will do this by using [Auth0](https://auth0.com/), a service for developers to authorize and authenticate users, which is a helpful tool for Alexa Account Linking.

### Contents

*   [Introduction](#introduction)
    *   [What you need for Alexa Account Linking](#alexa-account-linking-requirements)
*   [How OAuth2 Works](#how-oauth2-works)
*   [What is Auth0?](#what-is-auth0)
*   [Alexa Account Linking: Connecting your Skill with Auth0](#connecting-your-skill-with-auth0)
    *   [Auth0 Setup](#auth0-setup)
    *   [Alexa Skill Setup](#alexa-skill-setup)
    *   [Add Account Linking to your Code](#add-account-linking-to-your-code)

Jovo is an open-source development framework for building voice apps that work on both Amazon Alexa and Google Assistant with only one code base. Take a look at the [Jovo Framework Docs](https://www.jovo.tech/framework/docs) or our [Voice App Courses](https://www.jovo.tech/learn) to learn more.

👉 Just interested in a user's email address? Take a look at [Login with Amazon](https://www.jovo.tech/blog/alexa-login-with-amazon-email/). 👉 Interested in Google Assistant Account Linking? [How to set up Account Linking for Google Actions with Auth0 and Jovo](https://www.jovo.tech/blog/google-action-account-linking-auth0/).   

![](./img/line2.png)



## Introduction



![](./img/alexa-account-linking-1024x464.png)

 One way to make your voice application stand out from the other Skills is by using **Account Linking**. It allows you to get to know your users and lets you personalize the experience with user specific data. For example, you could do things like this:

*   Greet your users with their name every time they start your voice application
*   Get access to an email address to provide a user with additional information via email
*   Create multimodal experiences by connecting a voice experience with e.g. a web service, Messenger chatbot, or other accounts

### What you need for Alexa Account Linking

Amazon requires the **OAuth 2.0 protocol for Account Linking**. You can find the official documentation by Amazon here: [Link an Alexa User with a User in Your System](https://developer.amazon.com/docs/custom-skills/link-an-alexa-user-with-a-user-in-your-system.html). As a developer you have two options to implement Account Linking for your Skill:

*   You set up your own OAuth 2.0 server, or
*   You use one of the many identity providers (Google, Facebook, Twitter, etc.) out there

First of all, setting up an OAuth 2.0 server can be difficult (at least, I had a tough time), and using a single identity provider restricts your user. Not everybody has a Facebook account. A service like Auth0 comes in handy in such a situation. It allows us to use the many identity providers and the standard username-password login at the same time. In this post, we're first learning more about OAuth2 in general, and then walk through setting up a simple example of Alexa Account Linking with Auth0, step by step.   

![](./img/line2.png)



## How OAuth2 Works

[OAuth 2.0](https://oauth.net/2/) is an authorization protocol which allows third-party applications to gain limited access to an HTTP service. It enables the user to provide access to restricted resources, without giving away their credentials (username and password). The protocol defines the following four roles:

*   **Resource Owner**: In our case the user, who wants to link their account with your Alexa Skill.
*   **Resource Server**: The server, where our user's data is located at. Has to be able to respond to resource requests using access tokens.
*   **Client**: The application, which wants to access the user's account. In this case it's Alexa.
*   **Authorization Server**: The server that authenticates the identity of the resource owner and provides the access token.

Here's how the roles interact with each other: 

![](./img/how-oauth2-works.png)

   

![](./img/line2.png)



## What is Auth0?



[![](https://www.jovo.tech/blog/wp-content/uploads/2017/12/auth0-screenshot.jpg)](https://auth0.com/)

 [Auth0](https://auth0.com/) provides a service for developers to authorize and authenticate users without having to deal with all the security issues and the burden of running your own OAuth 2.0 server. Here are some of the tools Auth0 provides:

*   [User Management](https://auth0.com/user-management) that supports both standard username + password logins and social logins like Google, Facebook, etc.
*   [Auth0 Lock](https://auth0.com/lock), a login box
*   [Passwordless](https://auth0.com/passwordless) Logins

Usually, such a service is not free, but Auth0 offers a free tier for up to 7,000 regular active users/month. They count a regular active user as someone who has ‘authenticated with username/password, passwordless connections or any social provider in the last calendar month, counted per application’. In the following steps, we are going to use both _username + password_ and _social login_ features by Auth0 to create an Account Linking process for our Alexa Skill.   

![](./img/line2.png)



## Alexa Account Linking: Connecting your Skill with Auth0

There are three steps needed to connect your Alexa Skill with Auth0. 1. Set up an Auth0 application and the providers we want to use for _social login_ 2\. Fill out the Account Linking form on the Amazon Alexa Developer platform 3. Add the code, so we can prompt our users to link their account with our Skill.

### 1\. Auth0 Setup

In this section, we're going through all the necessary steps you need to create an application on Auth0 and to pick identity providers for your social login. First of all you have to create an account on [Auth0](https://auth0.com/).

#### Create an Application

To create an application click on the orange button on the top right corner of the dashboard: 

![](./img/auth0_landing.png)

 Name your application and select '_Machine to Machine Applications_': 

![](./img/auth_new_application.png)

 Choose the **Auth0 Management API** and select **all scopes**. Going over all the scopes would be to much for this blogpost. 

![](./img/auth_management_api.png)

 The result should be a dashboard like this: 

![](./img/auth_application_dashboard.png)

 Switch to the **Settings** tab and change the _Token Endpoint Authentication Method_ to _Basic_. 

![](./img/auth_authentication_method.png)

 At the bottom of the settings tab click on **Show Advances Settings** and switch to **OAuth**. Select _HS256_ on _JsonWebToken Signature Algorithm_ and save your changes. 

![](./img/auth0-advanced-settings.jpg)



#### Social Logins

The next step is to enable the social logins you want to use. You can find them in _Connections > Social_: 

![](./img/auth0Setup_6.png)

 Here is an overview of connections you can choose from: 

![](./img/auth0-social-connections.jpg)

 Choose the provider you want to enable and the data you want to access. There is guide on how to set up every provider. It's marked on the screenshot: 

![](./img/auth0Setup_7.png)



### 2\. Alexa Skill Setup

If you don't already have on, go to the [Amazon Developer Platform](https://developer.amazon.com/), create an Alexa Skill and open the **Account Linking** tab: Below, you can find a list of all the information needed for Account Linking, and where to get it from:

*   **Authorization URL**: Auth0 Client we created earlier → Settings → Advanced Settings → Endpoints → OAuth Authorization URL
*   **Client ID**: Auth0 Client we created earlier → Settings → Client ID
*   **Domain List**: cdn.auth0.com
*   **Scope**: openid, offline_access, profile, email
*   **Authorization Grant Type**: Auth Code Grant
*   **Access Token URI**: Auth0 Client we created earlier → Settings → Advanced Settings → Endpoints → OAuth Token URL
*   **Client Secret**: Auth0 Client we created earlier → Settings → Client Secret

The scopes we use indicate the level of access we want on the user's data:

*   _openid_ allows us to use the API request we need later on, to access the stored user data.
*   _offline_access_ is needed because the access token we get will expire at some point. To request a new one, without having to re-authenticate the user, we need that scope.
*   _profile_ (basic data like name and birthdate) and _email_ are used to tell the server what kind of data we want.

This is how it looks like when it's filled out: 

![](./img/alexa_accountlinking.png)

 Now before you close both tabs, we have to add the _Redirect URLs_ provided by Amazon (see in the above screenshot) to our Auth0 Client. On your Client's **Settings** tab, there is the _Allowed Callback URLs_ field. Add the _Redirect URLs_ to that field and separate them with a comma. 

![](./img/auth0-allowed-callback-urls.jpg)



### 3\. Add Account Linking to Your Code

👉 Just getting started with Alexa Skill development? Take a look here: [Build an Alexa Skill in Node.js with Jovo](https://www.jovo.tech/blog/alexa-skill-tutorial-nodejs/).

You're pretty much at the end of this tutorial. The last thing we want to show you is a code example, which should give you an idea how to implement Account Linking in your own code.
```javascript
this.alexaSkill().showAccountLinkingCard();
this.tell("Please link your account");
```
This will display an Account linking card in the Alexa companion app or the Alexa website. By clicking on **Link Account** the user will be redirected to your own Auth0 login page. 

![](./img/auth0Setup_9.png)

 That's how the Auth0 login page looks like: 

![](./img/auth0Setup_10.png)

 To access the stored user data, you simply make an API request, using the access token your skill gets with every request made after the user linked his account. I recommend the `request` package since it is the simplest solution. You can install it with npm:
```sh
$ npm install --save request
```
After that import it in your `app.js` file:
```javascript
const request = require('request');
```
Now let's get to the request:
```javascript
let token = this.getAccessToken();
let options = {
    method: 'GET',
    url: 'https://jovotest.auth0.com/userinfo/', // You can find your URL on Client --> Settings --> 
                                                 // Advanced Settings --> Endpoints --> OAuth User Info URL
    headers:{
        authorization: 'Bearer ' + token,
    }
};

// API request
request(options,(error, response, body) => {
    if (!error && response.statusCode === 200){

        let data = JSON.parse(body); //Store the data we got from the API request
        console.log(data);
        /*
        To see how the user data was stored,
        go to Auth -> Users -> Click on the user you authenticated earlier -> Raw JSON
        */
        this.tell('Hi, ' + data.given_name);
    } else {
        this.tell('Error');
    }
}
```
At the end your code should look like this:
```javascript
/*
Checks if there is an access token. 
No access token -> Ask the user to sign in
If there is one -> API call to access user data 
*/
if (!this.getAccessToken()){
    this.alexaSkill().showAccountLinkingCard();
    this.tell('Please link your account');
} else {
    let token = this.getAccessToken();
    let options = {
        method: 'GET',
        url: 'https://jovotest.auth0.com/userinfo/', // You can find your URL on Client --> Settings --> 
                                                     // Advanced Settings --> Endpoints --> OAuth User Info URL
        headers:{
            authorization: 'Bearer ' + token,
        }
    };
            
    // API request
    request(options, (error, response, body) => {
        if (!error && response.statusCode === 200){   
            var data = JSON.parse(body); // Store the data we got from the API request
            console.log(data);
            /*
            To see how the user data was stored,
            go to Auth -> Users -> Click on the user you authenticated earlier -> Raw JSON
            */
            this.tell('Hi, ' + data.given_name);
        } else {
            this.tell('Error');
        }
    });
}
```
That's it, you made it!   **Any questions? You can reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to set up Account Linking for your Alexa Skill with Auth0", "author": "kaan-kilic" }-->