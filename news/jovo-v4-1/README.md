---
title: 'Introducing Jovo v4.1: Alexa Conversations, ISP, Slack Plugin, and AWS Lex'
excerpt: 'Jovo v4 improves every step of the workflow, including a new browser-based Debugger, a new CLI, an output template engine, a component-based system, and much more. Learn more below.'
date: '2021-11-18'
---

# Jovo v4.1: Alexa Conversations, ISP, Slack Plugin, and AWS Lex

![Jovo v4](./img/jovo-v4-1.png 'Jovo launches version 4')

In November, we [launched Jovo `v4`](https://www.jovo.tech/news/jovo-v4), which was our most exciting update so far. The Jovo `v4.1` now builds on top of that and comes with many improvements and new integrations. Learn more below.

## Introduction

`v4.1` is all about building on top of the `v4` architecture and showing what can be built with it. The most notable features are:

- [Alexa Conversations](#alexa-conversations): Build Alexa Skills that make use of the Alexa Conversations dialogue management engine
- [Alexa ISP](#alexa-isp): Offer in-skill purchases in your Alexa Skills
- [AWS Lex](#aws-lex): Integrate your Jovo apps with Lex speech recognition and natural language understanding
- [Slack Plugin](#slack-plugin): Send errors and notifications to a Slack channel

`v4.1` also comes with many smaller improvements and bugfixes.

## Alexa Conversations

We're excited to offer an integration with [Alexa Conversations](https://developer.amazon.com/docs/alexa/conversations/about-alexa-conversations.html) that uses machine learning driven dialogue management.

This way, you can build hybrid Alexa Skills where rules driven interactions can be built with Jovo [components](https://www.jovo.tech/docs/components) and [handlers](https://www.jovo.tech/docs/handlers), and then delegate to probabilistic dialogue management for tasks like slot filling:

```typescript
import { DialogDelegateRequestOutput } from '@jovotech/platform-alexa';
// ...

someHandler() {
  // ...

  return this.$send(DialogDelegateRequestOutput, {
    target: 'AMAZON.Conversations',
    updatedRequest: {
      type: 'Dialog.InputRequest',
      input: {
        name: '<utteranceSetName>',  // Utterance set must use the Invoke APIs dialog act
    }
  });
}
```

[You can find a sample repo here](https://github.com/jovotech/skill-sample-nodejs-alexa-conversations-pet-match) and [learn more in the Jovo docs](https://www.jovo.tech/marketplace/platform-alexa/alexa-conversations).

## Alexa ISP

The [integration for Alexa ISP](https://www.jovo.tech/marketplace/platform-alexa/isp) allows you to interact with the Alexa In-Skill Purchasing (ISP) engine.

Here is an example how you can accept a request after a successful purchase using a helper for the [`@Handle` decorator](https://www.jovo.tech/docs/handlers#handler-routing-and-the-handle-decorator):

```typescript
import { Handle } from '@jovotech/framework';
import { AlexaHandles, IspType, PurchaseResult } from '@jovotech/platform-alexa';
// ...

@Handle(AlexaHandles.onIsp(IspType.Buy, PurchaseResult.Accepted)) // or ('Buy', 'ACCEPTED')
successfulPurchase() {
  return this.$send({ message: 'Thanks for buying.' });
}
```

[Learn more in the Jovo docs](https://www.jovo.tech/marketplace/platform-alexa/isp).

## AWS Lex

We now offer an integration with [AWS Lex](https://aws.amazon.com/lex/), which is a service that can do both [automatic speech recognition (ASR)](https://www.jovo.tech/docs/asr) and [natural language understanding (NLU)](https://www.jovo.tech/docs/nlu).

You can install it like this:

```sh
$ npm install @jovotech/slu-lex
```

[Learn more in the Jovo docs](https://www.jovo.tech/marketplace/slu-lex).

## Slack Plugin

The [Slack Plugin](https://www.jovo.tech/marketplace/plugin-slack) allows you to monitor your Jovo apps as it automatically sends errors to a Slack channel.

You can install it like this:

```sh
$ npm install @jovotech/plugin-slack
```

As an upgrade to the `v3` version of this plugin, you can even send notifications manually right from your handlers:

```typescript
this.$slack.sendMessage('Some notification');
```

[Learn more in the Jovo docs](https://www.jovo.tech/marketplace/plugin-slack).

## Getting Started with Jovo

There are several ways how you can get started with Jovo:

- Follow our [getting started guide](https://www.jovo.tech/docs/getting-started) to install the Jovo CLI and create a new project.
- Coming from Jovo `v3`? Follow our [migration guide](https://www.jovo.tech/docs/migration-from-v3).
- Take a look at our `v4` template for [TypeScript](https://github.com/jovotech/jovo-v4-template) and [JavaScript](https://github.com/jovotech/jovo-v4-template-js).

## Breaking Changes

When you're building for Alexa and run into problems while bundling the code, make sure that the bundle script in your `package.json` file includes the following:

```json
{
  "scripts": {
    "bundle": "[...] --external:@alexa/*"
  }
}
```

The reason for this is that `esbuild` can't resolve `vscode`, a dependency used in `@alexa/acdl`, which is used for the Alexa Conversations integration.

[Find a sample `package.json` here](https://github.com/jovotech/jovo-v4-template/blob/master/package.json).
