---
title: 'Jovo Staging and Deployment: New Example and Video Deep Dive'
excerpt: 'Learn more about Alexa Skills and Google Actions that can be deployed to AWS.'
date: '2022-05-02'
---

# Jovo Staging and Deployment: New Example and Video Deep Dive

![Jovo Staging and Deployment](./img/jovo-staging-deep-dive.png 'Alexa Skills and Google Actions local development and production')

Learn how you can use Jovo staging and deployment features to build an Alexa Skill and Google Action with a `dev` stage for local development and a `prod` stage that can be deployed to AWS.

Watch the video here:

https://www.youtube.com/watch?v=D4kYbr7Mm2s

## Get Started

Jovo offers many features that make it easier to build robust workflows, for example:

- [Staging](https://www.jovo.tech/docs/staging): Maintain different stages like `dev` and `prod`. For example, this allows you to use different Alexa Skill projects for different stages.
- Deployment, for example using the [Serverless integration](https://www.jovo.tech/marketplace/target-serverless): This allows you to deploy certain stages to cloud providers using the [Jovo CLI](https://www.jovo.tech/docs/cli).

To make this process easier, we created a new example on GitHub: [jovo-sample-alexa-googleassistant-lambda](https://github.com/jovotech/jovo-sample-alexa-googleassistant-lambda). The `README` and the video above include a step by step guide on how to set up the following:

- Maintain Alexa Skills and Google Actions using the [Jovo CLI](https://www.jovo.tech/docs/cli)
- Use a `dev` stage for local development and debugging
- Use a `prod` stage for the live app, hosted on [AWS](https://www.jovo.tech/marketplace/server-lambda)
- Deploy to AWS using the [Jovo Serverless integration](https://www.jovo.tech/marketplace/target-serverless)
