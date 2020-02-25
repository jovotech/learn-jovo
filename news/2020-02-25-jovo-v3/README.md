# Jovo v3: Samsung Bixby, Web Apps, Conversational Components, and more

This is one of our biggest releases ever. Today, we published version `3.0` of the Jovo Framework, the most popular developer tool for building voice experiences that work on Amazon Alexa, Google Assistant, and way more other platforms.

This new release not only comes with many new integrations like Samsung Bixby and web apps, it also includes lots of improvements and something we call Conversational Components (which we're super excited about). Learn more about all the features below.

* [More Platforms and Clients](#more-platforms-and-clients)
   * [Samsung Bixby](#samsung-bixby)
   * [Web Apps and Chat](#web-apps-and-chat)
   * [Twilio Autopilot](#twilio-autopilot)
   * [Facebook Messenger](#facebook-messenger)
   * [And even more](#and-even-more)
* [RIDR Pipeline](#ridr-pipeline)
   * [Clients](#clients)
   * [ASR, NLU, and SLU Integrations](#asr--nlu--and-slu-integrations)
   * [TTS Integrations](#tts-integrations)
* [More Features](#more-feautre)
   * [Conversational Components](#conversational-components)
   * [Input Validation](#input-validation)
* [How to Update](#how-to-update)
   * [Breaking Changes](#breaking-changes)


*Like what we're doing? [Support us with a star on GitHub](https://github.com/jovotech/jovo-framework/)* 


## More Platforms and Clients

We've worked hard to integrate with more providers to enable companies and teams to add voice to any platform or device. One big step is also our RIDR Pipeline, but [more on that below](#ridr-pipeline).

Here are some of the new platforms that are supported with `v3` of the Jovo Framework:

* [Samsung Bixby](#samsung-bixby)
* [Web Apps and Chat](#web-apps-and-chat)
* [Twilio Autopilot](#twilio-autopilot)
* [Facebook Messenger](#facebook-messenger)
* [And even more](#and-even-more)

### Samsung Bixby

> Learn more in the Jovo Docs: [Samsung Bixby Platform](https://www.jovo.tech/docs/samsung-bixby).

You've waited for this! And it has even been [leaked](https://twitter.com/franciscorivash/status/1208061410422468609): The Jovo Framework now supports [Bixby](https://bixbydevelopers.com/), the assistant platform by Samssung that has made a lot of waves in the voice-first ecosystem recently.

Although Bixby Capsule development is different compared to Alexa Skills and Google Actions, we did our best to make it easier to also deploy your Jovo voice apps to the platform (and make use of all our helpful features like CMS integrations and local development).

```js
// @language=javascript

// src/app.js

const { Bixby } = require('jovo-platform-bixby');

app.use(
   // Other platforms and integrations
   new Bixby()
);

// @language=typescript

// src/app.ts

import { Bixby } from 'jovo-platform-bixby';

app.use(
   // Other platforms and integrations
   new Bixby()
);
```

You can find a sample project here: [jovo-bixby-capsule-example](https://github.com/jovotech/jovo-bixby-capsule-example).


### Web Apps and Chat

We even took it a step further: It's now possible to voice-enable (and build text-based conversational experiences for) web apps and websites.

This is possible due to the new Jovo Core Platform which comes with a standardized request and response format.

```js
// @language=javascript

// src/app.js

const { CorePlatform } = require('jovo-platform-core');

app.use(
   // Other platforms and integrations
   new CorePlatform()
);

// @language=typescript

// src/app.ts

import { CorePlatform } from 'jovo-platform-core';

app.use(
   // Other platforms and integrations
   new CorePlatform()
);
```

This core platform can use different [ASR, NLU, and TTS Integrations](#asr--nlu--and-tts-integrations) and also [Clients](#clients). You can find the first web starter, built in Vue.js, [in our GitHub repository](https://github.com/jovotech/jovo-framework/tree/v3/examples/typescript/core-platform-clients/vue/one-pager). More to come.


### Twilio Autopilot

> Learn more in the Jovo Docs: [Twilio Autopilot Platform](https://www.jovo.tech/docs/twilio-autopilot).

Jovo now supports Twilio Autopilot, a powerful service by Google that lets you deploy assistants to various platforms, including the phone.

```js
// @language=javascript

// src/app.js

const { Autopilot } = require('jovo-platform-twilioautopilot');

app.use(
   // Other platforms and integrations
   new Autopilot()
);

// @language=typescript

// src/app.ts

import { Autopilot } from 'jovo-platform-twilioautopilot';

app.use(
   // Other platforms and integrations
   new Autopilot()
);
```

You can also find some examples in our GitHub repository, either in [JavaScript](https://github.com/jovotech/jovo-framework/tree/master/examples/javascript/04_autopilot/hello-world) or [TypeScript](https://github.com/jovotech/jovo-framework/tree/master/examples/typescript/04_autopilot/hello-world).


### Facebook Messenger

> Learn more in the Jovo Docs: [Facebook Messenger Platform](https://www.jovo.tech/docs/facebook-messenger).

Jovo added Facebook Messenger support in `v2.1` ([see the announcement here](https://www.jovo.tech/news/2019-03-05-jovo-v2-1)) through [Dialogflow integrations](https://www.jovo.tech/docs/dialogflow-integrations). This was great for teams who were already using Dialogflow for Google Assistant, however, it also came with a few limitations.

Today's release entails a standalone version of Facebook Messenger, which can be used with any natural language integration supported by Jovo. You can add Facebook Messenger like this:

```js
// @language=javascript

// src/app.js

const { FacebookMessenger } = require('jovo-platform-facebookmessenger');

app.use(
   // Other platforms and integrations
   new FacebookMessenger()
);

// @language=typescript

// src/app.ts

import { FacebookMessenger } from 'jovo-platform-facebookmessenger';

app.use(
   // Other platforms and integrations
   new FacebookMessenger()
);
```

You can also find some examples in our GitHub repository, either in [JavaScript](https://github.com/jovotech/jovo-framework/tree/master/examples/javascript/05_facebookmessenger) or [TypeScript](https://github.com/jovotech/jovo-framework/tree/master/examples/typescript/05_facebookmessenger).


### And even more

We also built support for many more platforms, like mobile apps and hardware devices. We will be rolling out the support for these over the upcoming weeks. See our [Clients](#clients) section for more.


## RIDR Pipeline

The biggest addition of Jovo `v3` is the RIDR Pipeline that allows you to build your own custom voice and conversational experiences:

* **Request**: Be able to react to input from any platform or device
* **Interpretation**: Hook into ASR, NLU, and SLU services to make sense of the incoming request
* **Dialog Management**: Build the logic with help from the Jovo tooling for fast development
* **Response**: Send back the response to the platform or connect to a TTS provider

Our abstraction layer makes it possible to replace different building blocks, which adds more flexiblity and reduces vendor lock-in.

Learn more below:
* [Clients](#clients)
* [ASR, NLU, and SLU Integrations](#asr--nlu--and-slu-integrations)
* [TTS Integrations](#tts-integrations)

### Clients

The following clients will be rolled out over the next weeks:

* [Voice in Web Apps](https://www.jovo.tech/voice-web-apps)
* [Web Chat Widgets](https://www.jovo.tech/web-chat)
* [Voice in Mobile Apps](https://www.jovo.tech/voice-mobile)
* [Raspberry Pi Support](https://www.jovo.tech/raspberry-pi-voice)
* [Custom Hardware Support](https://www.jovo.tech/hardware-voice)


### ASR, NLU, and SLU Integrations

When you're building your own custom voice platform, automatic speech recognition (ASR) and natural language understanding (NLU), or the mix of both, spoken language understanding (SLU), become essential. Jovo now supports integrations into many of those services:

* [Jovo ASR Integrations](https://www.jovo.tech/docs/asr)
* [Jovo NLU Integrations](https://www.jovo.tech/docs/nlu)
* [Jovo SLU Integrations](https://www.jovo.tech/docs/slu)


### TTS Integrations

Building your own custom voice experience or want to have a consistent voice branding across different platforms and devices? Integrate with popular text-to-speech services with the Jovo TTS integrations, including:

* [Microsoft Azure Text to Speech](https://www.jovo.tech/docs/tts/microsoft-azure)
* [Google Cloud Text to Speech](https://www.jovo.tech/docs/tts/google-cloud)
* [Amazon Polly Text to Speech](https://www.jovo.tech/docs/tts/amazon-polly)


## More Features

That's not all yet. Over the last months, we made many changes to the Jovo code base before we decided to package everything into a `v3`. [You can find the CHANGELOG here](https://github.com/jovotech/jovo-framework/blob/master/CHANGELOG.md).

Some of the features we're most excited about are:
* [Conversational Components](#conversational-components)
* [Input Validation](#input-validation)

### Conversational Components

> Learn more in the Jovo Docs: [Conversational Components](https://www.jovo.tech/docs/components).

Conversational Components are pre-packaged modules that contain a language model, logic (handlers), and responses (i18n) to reach a certain goal. For example, there is a [component to ask a user for their phone number](https://github.com/jovotech/jovo-component-get-phone-number):

```js
// @language=javascript

// src/app.js

const { GetPhoneNumber } = require("./components/jovo-component-get-phone-number");

app.useComponents(new GetPhoneNumber());

// @language=typescript

// src/app.ts

import { GetPhoneNumber } from "./components/jovo-component-get-phone-number";

app.useComponents(new GetPhoneNumber());
```

We will roll out more components over the next few weeks, you can find a few at our [Jovo Components Overview](https://www.jovo.tech/components).

You can find an example project that uses a conversational component in our GitHub repository, either in [JavaScript](https://github.com/jovotech/jovo-framework/tree/master/examples/javascript/components) or [TypeScript](https://github.com/jovotech/jovo-framework/tree/master/examples/typescript/components).


### Input Validation

> Learn more in the Jovo Docs: [Input Validation](https://www.jovo.tech/docs/routing/input#validation).

When dealing with a lot of different inputs (e.g. slots, entities in some platforms), it can be challenging to validate all the incoming data. This is why we now support input validation.

```js
// @language=javascript

// src/app.js

MyNameIsIntent() {
    const schema = {
        name: new IsRequiredValidator()
    };

    const validation = this.validate(schema);

    if(validation.failed('name')) {
        return this.ask('Please tell me your name again.');
    }

    this.tell(`Hey ${this.$inputs.name.value}!`);
}

// @language=typescript

// src/app.ts

MyNameIsIntent() {
    const schema = {
        name: new IsRequiredValidator()
    };

    const validation = this.validate(schema);
    
    if(validation.failed('name')) {
        return this.ask('Please tell me your name again.');
    }

    this.tell(`Hey ${this.$inputs.name.value}!`);
}
```

You can also find some examples in our GitHub repository, either in [JavaScript](https://github.com/jovotech/jovo-framework/tree/master/examples/javascript/validation/src) or [TypeScript](https://github.com/jovotech/jovo-framework/tree/master/examples/typescript/validation/src).




## How to Update

> [Learn more in the Jovo Upgrading Guide](https://www.jovo.tech/docs/installation/upgrading).

To update to the latest version of Jovo, either create a new Jovo project, or use the following commands in an existing project:

```sh
# Update to the latest version of the Jovo CLI
$ npm install -g jovo-cli@latest

# Update Jovo packages in your Jovo project
$ jovo update
```

This will then prompt you again if you want to upgrade.


### Breaking Changes

Here's the good news: There are almost no breaking changes that make migration to Jovo v3 hard. The only change is that we dropped support for Node version 8.



**Any thoughts? Let us know in the comments below.**


<!--[metadata]: { "description": "Learn more about Jovo Framework version 3.0, which was released in February 2020.", "author": "jan-koenig", "tags": "Releases", "og-image": "" }-->
