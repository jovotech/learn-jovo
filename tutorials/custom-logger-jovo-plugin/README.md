# Creating a Custom Logger as a Jovo Plugin

Learn how to add a custom logger using the [Jovo Plugin feature](https://www.jovo.tech/docs/advanced#plugins).

You can find the full code example of this tutorial here: [jovo-templates/tutorials/plugin](https://github.com/jovotech/jovo-templates/tree/master/tutorials/plugin).

Or download it using the Jovo CLI:

```sh
$ jovo new --template tutorials/plugin
```

### Contents

- [Introduction to Jovo Plugins](#introduction-to-jovo-plugins)
- [How Jovo Plugins Work](#how-jovo-plugins-work)
- [Creating a Custom Logger](#creating-a-custom-logger)

## Introduction to Jovo Plugins

We added the Jovo Plugin feature in order to allow our users to customize the framework to their needs as well as to build features, which other members of our community can benefit from.

Plugins allow you to easily extend the Jovo Framework without having to mess with its core code and architecture. You can, for example, integrate your own analytics API or create a custom logger.

[See example plugins here.](https://www.jovo.tech/plugins)

## How Jovo Plugins Work

The plugins work with listeners, which get triggered by certain events. These events range from requests your app gets to image cards you send out as a response. As soon as the listeners gets triggered its code block gets executed.

The events you can create a listener for always have the Jovo object as an argument, which gives you access to all the methods you were able to use inside your handlers. In addition to that you also get specific information for the respective event. The `tell` event for example comes with the speech String you used. Here’s a list of all the events:

| Category | Name | Method | Arguments |
| --- | --- | --- | --- |
| Routing | request | `this.app.on('request')` | `jovo` |
|  | response | `this.app.on('response')` | `jovo` |
|  | followUpState | `this.app.on('followUpState')` | `jovo`, `state` |
|  | removeState | `this.app.on('removeState')` | `jovo` |
|  | toIntent | `this.app.on('toIntent')` | `jovo`, `intent` |
|  | toStateIntent | `this.app.on('toStateIntent')` | `jovo`, `state`, `intent` |
|  | toStatelessIntent | `this.app.on('toStatelessIntent')` | `jovo`, `intent` |
|  | endSession | `this.app.on('endSession')` | `jovo` |
| Output | tell | `this.app.on('tell')` | `jovo`, `speech` |
|  | ask | `this.app.on('ask')` | `jovo`, `speech`, `repromptSpeech` |
|  | showSimpleCard | `this.app.on('showSimpleCard')` | `jovo`, `title`, `content` |
|  | showImageCard | `this.app.on('showImageCard')` | `jovo`, `title`, `content`, `imageUrl` |
|  | showAccountLinkingCard | `this.app.on('showAccountLinkingCard')` | `jovo` |
| Error | handlerError | `this.app.on('handlerError')` | `jovo`, `error` |
|  | responseError | `this.app.on('responseError')` | `jovo`, `error` |

## Creating a Custom Logger

Since plugins feature is tailored for more advanced users, I will skip the creation of a Jovo project and jump right into the code example. The plugin we're going to create will be a simple logger to track Google's `Default Fallback Intent`.

To begin with we have to import the plugin class in our `app.js` file:

```javascript
const {App, Plugin} = require('jovo-framework');
```

To build our own plugin we have to create a class, which extends the base plugin class:

```javascript
class CustomLogging extends Plugin {

}
```

Inside that class we will add a constructor, which will call the parent class' constructor:

```javascript
class CustomLogging extends Plugin {
    constructor(options) {
        super(options);
    }
}
```

Next up is the core part of the plugin. The `init()` method is the place, where you define the listener functions:

```javascript
class CustomLogging extends Plugin {
    constructor(options) {
        super(options);
    }
    init() {
	
    }
}
```

To add a listener we use the following structure:

```javascript
this.app.on(event, (arguments) => {
	// code logic
});
```

The first thing we want to log is if the user triggered the `Default Fallback Intent` as well as how they did it, i.e. the current state and utterance. The `request` event comes in handy for that. To access the requested intent's name and the user's utterance we can use the `getIntentName()` and `getRawText()` functions:

```javascript
init() {
	this.app.on('request', (jovo) => {
    	if (jovo.getIntentName() === 'Default Fallback Intent') {
        	console.log(`State: ${jovo.getState()} | Raw Text: ${jovo.platform.getRawText()}`);
    	}
	});
}
```

To comprehend how the `Default Fallback Intent` was triggered we will also log our applications output. But we don't want to log every output, so instead of directly logging to the console everytime the `ask` event occurs, we will save the content of it to a variable instead, which we define in `init()`:

```javascript
init() {
    let output = '';
    this.app.on('ask', (jovo, speech, repromptSpeech) => {
        output += `speech: ${speech} | reprompt: ${repromptSpeech}`;
    });
}
```

We can now modify the `request` listener to log the content of `output` as well, by adding the stuff we wanted to log in `request` to `output`:

```javascript
init() {
    let output = '';
    this.app.on('request', (jovo) => {
        if (jovo.getIntentName() === 'Default Fallback Intent') {
            output += `\nState: ${jovo.getState()} | Raw Text: ${jovo.platform.getRawText()}`
            console.log(output);
            output = '';
        }
    });
    this.app.on('ask', (jovo, speech, repromptSpeech) => {
        output += `speech: ${speech} | reprompt: ${repromptSpeech}`;
    });
}
```

That's our logger. The last thing left to do is the registration of the plugin:

```javascript
app.register(new CustomLogging());
```

You can test the plugin either locally, where it will we logged to your console, or on [Lambda](https://aws.amazon.com/lambda/), which stores the logs on [Cloudwatch](https://aws.amazon.com/cloudwatch/). Here's an example how the logs might look like:

```text
speech: Hello World! What's your name? | reprompt: Please tell me your name.
State: TEST_STATE | Raw Text: I won't tell you my name
```

We at Jovo are really excited to see, which kind of plugins our community comes up with. By the way, the first ever Jovo plugin was created by [Cellular](https://github.com/cellular/jovo-plugin-raven).

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to create a custom logger as a Jovo plugin", "author": "kaan-kilic", "og-image": "https://www.jovo.tech/blog/wp-content/uploads/2018/06/tutorial-jovo-plugin.jpg" }-->