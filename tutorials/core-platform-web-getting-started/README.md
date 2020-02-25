# Title TBD

In this tutorial, we will go over the complete process of setting up a basic voice application for the web using the Jovo Framework.

> This tutorial expects basic knowledge about the Jovo Framework. If you're completely new to it, check out the [Getting Started](https://www.jovo.tech/docs/quickstart) page first

## Introduction
The Core platform is a generic platform for the Jovo Framework that is not bound to a specific device or environment.
The platform supports Automatic-Speech-Recognition (ASR), Natural-Language-Understanding (NLU) and Text-to-Speech (TTS) but it requires dedicated plugins to carry out these tasks. \
For a complete overview of available plugins, take a look [here]().


## Create A Jovo Project
The first step is to create a new Jovo project and add the `jovo-platform-core` package as a dependency:
```sh
$ jovo new core-platform-web-hello-world

$ npm install jovo-platform-core --save
```

Now, open the `src/app.js` file in your newly created project and register the platform:
```javascript
'use strict';

// ------------------------------------------------------------------
// APP INITIALIZATION
// ------------------------------------------------------------------

const { App } = require('jovo-framework');
const { Alexa } = require('jovo-platform-alexa');
const { GoogleAssistant } = require('jovo-platform-googleassistant');
const { JovoDebugger } = require('jovo-plugin-debugger');
const { FileDb } = require('jovo-db-filedb');

const { CorePlatform } = require('jovo-platform-core');

const app = new App();

const corePlatform = new CorePlatform();

app.use(
    new Alexa(),
    new GoogleAssistant(),
    new JovoDebugger(),
    new FileDb(),

    corePlatform
);
```

## Add a NLU plugin
Because the Core platform itself does not do any NLU-processing, a dedicated plugin to carry out this task is required. 
For this tutorial, the `jovo-nlu-dialogflow` package will be used. 

> If you are not familiar with Dialogflow, take a look [here]().

First of all, you will have to add the `jovo-nlu-dialogflow` package as a dependency:
```sh
$ npm install jovo-nlu-dialogflow --save
```

Now you can register the plugin to the Core platform:
```javascript
'use strict';

// ------------------------------------------------------------------
// APP INITIALIZATION
// ------------------------------------------------------------------

const { App } = require('jovo-framework');
const { Alexa } = require('jovo-platform-alexa');
const { GoogleAssistant } = require('jovo-platform-googleassistant');
const { JovoDebugger } = require('jovo-plugin-debugger');
const { FileDb } = require('jovo-db-filedb');

const { CorePlatform } = require('jovo-platform-core');
const { DialogflowNlu } = require('jovo-nlu-dialogflow');

const app = new App();

const corePlatform = new CorePlatform();

corePlatform.use(
    new DialogflowNlu()
)

app.use(
    new Alexa(),
    new GoogleAssistant(),
    new JovoDebugger(),
    new FileDb(),

    corePlatform
);

// ...
```

`DialogflowNlu` requires a credentials-file to work, so the next step is to retrieve this credentials-file. For more information take a look [here](). \
After you have downloaded the credentials-file, move it to `src` and rename it to `credentials.json`. 

The `DialogflowNlu` plugin should work now and the Jovo app should be fully setup.

## Setup the Web App
For the sake of simplicity of this tutorial, we will not create our own web app but rather download an example. \
You can download the example [here]().

After you have downloaded the example, extract it somewhere and open the directory in the terminal. \
Now you will have to install all dependencies:
```sh
$ npm install
```

## Starting both 
Now start the Jovo app first:
```sh
$ jovo run
```

This will launch the server for the Jovo app. \
Make sure to copy the url of the [Jovo Webhook]() that is printed after a successful startup.

You will now have to go into `src/main.ts` of the web app and replace the value of `WEBHOOK_URL` with the previously copied url.

Now you can start the web app:
```sh
$ npm run serve
```

After a successful startup, the url of the web app will be printed. Click on it to open the web app.

Overall the web app is now able to communicate with the Jovo app.

## Next up
While we got the basic project working, the Core platform also provides more features, that will be explained in a second. 

