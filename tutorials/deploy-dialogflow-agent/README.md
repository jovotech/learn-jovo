# Deploying your Dialogflow Agent using the Jovo CLI

Still updating your Dialogflow agent manually? In this post you're going to learn how to use the Jovo CLI and Dialogflow v2 API to deploy a Dialogflow agent from your command line.

### Contents

* [Introduction](#introduction)
* [Step 1: Setting up Authentication](#step-1:-setting-up-authentication)
  * [Step 1.1: Creating a new Service Account](#step-11:-creating-a-new-service-account)
  * [Step 1.2: Activating the Account with the Cloud SDK](#step-2:-activating-the-account-with-the-cloud-sdk)
* [Step 2: Deploying your Agent with the Jovo CLI](#step-2:-deploying-your-agent-with-the-jovo-cli)

## Introduction

![](./img/dialogflow-window2-1.png)
 
Making changes to Dialogflow (the natural language understanding platform that is used for Google Assistant projects in Jovo) can be tedious if you want to keep it up to date with the [Jovo Language Model](https://www.jovo.tech/framework/docs/model). Usually, you have to do one of the following in the [Dialogflow Console](https://console.dialogflow.com/):

* Update the language model manually
* Use the _dialogflow_agent.zip_ file created with [jovo deploy](https://www.jovo.tech/framework/docs/cli#jovo-deploy) to RESTORE your existing Dialogflow agent in the preferences:


![](./img/dialogflow-restore-1024x343.jpg)

Can become quite annoying, right?

Finally, you can use the command line to update your Dialogflow agent! Since Jovo v1.0, we support the Dialogflow v2 API which allows you to set up an authentication for programmatic deployments. There are a few manual steps we need to do first, so let's get started.

## Step 1: Setting up Authentication

First of all, go to the [Dialogflow console](https://console.dialogflow.com/api-client/) and click on the gear icon right next to your agent's name to get to the settings tab. You will find a service account right beneath the project ID: 

![Dialogflow Service Account](./img/dialogflow_agent_service_account.png)

### Step 1.1: Creating a new Service Account

It will lead us to the Google Cloud Platform, where we will create a new service account using the button at the top of the page:

![Google Cloud IAM](./img/google_cloud_platform_IAM_landing.png)

![Google Cloud IAM New Account](img/google_cloud_platform_IAM_newAccount.png)

We select `Dialogflow API Admin` as the role:

![Google Cloud IAM Role](img/google_cloud_platform_IAM_role.png)

After that we create a new `JSON` key:

![Google Cloud IAM Key](img/google_cloud_platform_IAM_key.png)

Last but not least move the key file to our project's root folder.

### Step 1.2: Activating the Account with the Cloud SDK

![](./img/google-cloud-sdk.jpg)

The Jovo CLI uses the Google Cloud SDK to access Dialogflow through projects on the Google Cloud Platform. You can download it [here](https://cloud.google.com/sdk/docs/).

Install the SDK and initialize it using the [quickstart guide](https://cloud.google.com/sdk/docs/quickstarts) from Google.

## Step 2: Deploying your Agent with the Jovo CLI


After that add both the `projectId` of your Dialogflow agent and the path to the `keyFile` to your project's `project.js` file:

```js
// project.js

module.exports = {
    googleAction: {
      nlu: 'dialogflow',
      dialogflow: {
        projectId: '<your-project-id>',
        keyFile: './path/to/key-file'
      }
    },
    // Other configurations
};
```

Now run the build and deploy command and your agent will be updated:

```sh
jovo build -p googleAction --deploy
```

The result should look like this:

```text
   √ Deploying Google Action
     √ Creating file /googleAction/dialogflow_agent.zip
       Language model: en-US
       Fulfillment Endpoint: https://webhook.jovo.cloud/yourEndpoint
     √ Uploading and restoring agent for project test-73d3a
     √ Training started

  Deployment complete
```

**Any questions? You can reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to deploy a Dialogflow agent from the command line with the Jovo CLI", "author": "kaan-kilic", "tags": "Google Assistant, Dialogflow, Deployment", "og-image": "https://www.jovo.tech/blog/wp-content/uploads/2018/03/deploy-dialogflow-agent-1.jpg" }-->
