# Reference Environment Variables in your project.js

Learn how to use environment variables in your [`project.js`](https://www.jovo.tech/docs/project-js) file.

* [Introduction](#introduction)
* [How to use env variables](#how-to-use-env-variables)
   * [Referencing env keys](#referencing-env-keys)
   * [Setting env values](#setting-env-values)
* [Use Cases](#use-cases)


Watch the video here:

[![Video: Reference environment variables in your app.json with Jovo](./img/video-env-variables.jpg "youtube-video")](https://www.youtube.com/watch?v=F_xaDXSuDGs)

(Note: this is video still uses `v1` of the Jovo Framework. [Learn more about changes in `v2` here](https://www.jovo.tech/docs/installation/v1-migration))


## Introduction

The [`project.js`](https://www.jovo.tech/docs/project-js) is Jovo project configuration file that stores information like which platforms are used (`alexaSkill`, `googleAction`), including additional information like project IDs, endpoints, etc.

One of the useful features of the `project.js` is the ability to add stages to deploy your voice app to different environments (like `local` on your computer and `dev` on Lambda). Environment variables are a great way to define values that are directly linked to the current environment. 

Referencing environment variables in your project can be helpful if you want to use the `project.js` as a generic template and then leave it to each environment how to set its values.

## How to use env variables

You need to do two steps to use env variables in your `project.js` file:

* [Referencing env keys](#referencing-env-keys)
* [Setting env values](#setting-env-values)

### Referencing env key

You can reference env variables in your `project.js` with `process.env.key`. For example, if you want to reference the Alexa Skill ID, you can do it like this:

```javascript
// project.js

module.exports = {
  alexaSkill: {
    nlu: 'alexa',
    skillId: process.env.SKILL_ID
  },
  // Other configurations
};
```

### Setting env values

For local development, you can use a `.env` file in your root project directory that could look like this:

```
STAGE=local
```

If you're hosting your voice app on a service like AWS Lambda, you can define the environment variables there:

![Staging environment variable in AWS Lambda](./img/staging-env-lambda.png "How to set the stage variable in Lambda")

> [Learn more about `config.js` and staging here](https://www.jovo.tech/docs/v2/config-js#staging).

## Use Cases

Environment variables can be especially helpful for local development (in a `local` stage) where every developer e.g. has their own Alexa Skill ID and uses their own ASK CLI profile to deploy it to the Amazon Developer Portal:

```javascript
stages: {
    local: {
        alexaSkill: {
                skillId: process.env.SKILL_ID,
                askProfile: process.env.ASK_PROFILE,
        }
    }
}
```
The initial `.env` file could then look like this:

```
STAGE=local
SKILL_ID=
ASK_PROFILE=default
```

A new `jovo build` and `jovo deploy` would then use the empty Skill ID to create a new Alexa Skill. If the developer then pastes the Skill ID into the `.env` file, it is always used to deploy the Skill in the `local` stage.


<!--[metadata]: { "description": "Learn how to use environment variables for Alexa Skills and Google Actions in your project.js file.", "author": "jan-koenig" }-->