# Add DynamoDB to Store User Data

Learn how to use DynamoDB for certain development environments to store user data for your Alexa Skills and Google Actions with Jovo.

* [Introduction](#introduction)
* [Add DynamoDB as Database](#add-dynamodb-as-database)
   * [Install the Integration](#install-the-integration)
   * [Add Stages](#add-stages)
   * [Use Environment Variables](#use-environment-variables)
* [Additional Steps](#additional-steps)
   * [Specify Stage on AWS Lambda](#specify-stage-on-aws-lambda)
   * [Add Permissions to Lambda Role](#add-permissions-to-lambda-role)
   * [Test on Lambda](#test-on-lambda)

Watch the video here:

[![Video: Use DynamoDB to Store User Data in your Voice Apps](./img/video-dynamodb-dev.jpg "youtube-video")](https://www.youtube.com/watch?v=AevYJhAVQzg)

(Note: this is video still uses `v1` of the Jovo Framework. [Learn more about changes in `v2` here](https://www.jovo.tech/docs/installation/v1-migration))

## Introduction

Jovo offers a [database interface](https://www.jovo.tech/docs/databases) that allows you to persist user specific data in a variety of different databases. For additional information on which user specific data is stored, take a look at  the [Jovo User object](https://www.jovo.tech/docs/data/user).

For local development, it is recommended to use the [Jovo File DB](https://www.jovo.tech/docs/databases/file-db) that stores data in a `db/db.json` file for easy debugging. For hosting on AWS Lambda, most people use [DynamoDB](https://www.jovo.tech/docs/databases/dynamodb).

Switching between these different database types for local development and deployment to Lambda can be tedious. In this guide, you will learn how to use [individual config files](https://www.jovo.tech/docs/config-js#staging) to use different databases or tables for different stages.

## Add DynamoDB as Database

> [Find the full docs about our DynamoDB integration here](https://www.jovo.tech/docs/databases/dynamodb).

### Install the Integration

To add DynamoDB as a database, you first need to download the package from npm:

```sh
$ npm install --save jovo-db-dynamodb
```

Then, enable it in your `app.js` file in the `src` folder:

```javascript
// app.js

const { DynamoDb } = require('jovo-db-dynamodb');

// Enable DB after app initialization
app.use(new DynamoDb());
```

You can then define the `tableName` in your app configuration in the `src` folder:

```javascript
// config.js

module.exports = {
    // Other configurations
    db: {
        DynamoDb: {
            tableName: 'yourTableName',
        }
    }
};
```

However, this will add DynamoDB to the app no matter which stage it is currently in, disabling the FileDB that is helpful for local debugging.

This is why we recommend setting up different staging environments. Learn more about this in the next step.

### Add Stages

With [staging](https://www.jovo.tech/docs/config-js#staging) in your `config.js`. you can add different app configurations for each environment your app is running in.

You can use individual config files for each stage (`config.dev.js`, `config.prod.js`, etc.), that get merged into the default configuration.

For example, a `config.prod.js` could just define the configurations for the production stage (when the app is live) and use DynamoDB:

```javascript
// config.prod.js

module.exports = {
    // Other configurations
    db: {
        DynamoDb: {
            tableName: 'yourTableName',
        }
    }
};
```

The `config.js` would still look like this:

```javascript
// config.js

module.exports = {
   logging: true,

   intentMap: {
      'AMAZON.StopIntent': 'END',
   },

   db: {
        FileDb: {
            pathToFile: '../db/db.json',
        }
    },
};
```


### Use Environment Variables

Additionally, you can also use environment variables to give you more flexibility about the naming of your DynamoDB once the source code is deployed to Lambda:

```javascript
// config.prod.js

module.exports = {
    // Other configurations
    db: {
        DynamoDb: {
            tableName: process.env.TABLE_NAME,
        }
    }
};
```

## Additional Steps

After you have added updated the `config` files, you can upload your code to AWS Lambda (see our guide on [how to deploy to Lambda](./deploy-lambda-cli.md './deploy-lambda-cli') for more information). 

There are a few more steps to do to make DynamoDB work:

* [Specify Stage on AWS Lambda](#specify-stage-on-aws-lambda)
* [Add Permissions to Lambda Role](#add-permissions-to-lambda-role)
* [Test on Lambda](#test-on-lambda)


### Specify Stage on AWS Lambda

To be able to use different configurations for different stages, you need to let your code know which stage it is currently in.

You can do so by adding either `NODE_ENV` or `STAGE` to the environment variables on AWS Lambda:

![Staging environment variable in AWS Lambda](./img/staging-env-lambda.png "How to set the stage variable in Lambda")

This will be used to determine, which config file should be used, in case you've set it.

### Add Permissions to Lambda Role

To make your Lambda function able to create database tables, and edit their content, you need to give the IAM role that's associated with it the right permissions. You can find out more in the official documentation by Amazon: [AWS Lambda Permissions Model](http://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html). 

The easiest way to go is to give your role full permission by attaching the `AmazonDynamoDBFullAcess` policy:

![AWS Lambda DynamoDB Full Access](./img/dynamodb-fullaccess-permissions.jpg "How to give your Lambda role the right permissions for DynamoDB")

However, if you prefer to only give it the permissions needed, you can also create a new policy. Jovo needs the following permissions to be able to use the DynamoDB database integration:

* Read
   * GetItem
* Write
   * CreateTable
   * PutItem

Here is a policy summary you can use for it:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:CreateTable",
                "dynamodb:PutItem",
                "dynamodb:GetItem"
            ],
            "Resource": "arn:aws:dynamodb:*:*:table/*"
        }
    ]
}
```

After attaching this policy to your role, you should be good to go.


### Test on Lambda

After you have uploaded your code to Lambda, given your role the right permissions, you can test the function.

The initial test should create a fresh new DynamoDB table.


<!--[metadata]: { "description": "Learn how to use DynamoDB for certain development environments to store user data for your Alexa Skills and Google Actions with Jovo.", "author": "jan-koenig", "tags": "Database, DynamoDB" }-->
