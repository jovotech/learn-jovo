# Creating a Custom Logger as a Jovo Plugin

Learn how to add a custom logger using the [Jovo Plugin feature](https://www.jovo.tech/docs/plugins).

> You can find the full code example of this tutorial here: [jovo-templates/tutorials/plugin](https://github.com/jovotech/jovo-templates/tree/master/tutorials/plugin).

Or download it using the Jovo CLI:

```sh
$ jovo new --template tutorials/plugin
```

* [Introduction to Jovo Plugins](#introduction-to-jovo-plugins)
* [How Jovo Plugins Work](#how-jovo-plugins-work)
* [Creating a Custom Logger](#creating-a-custom-logger)
   * [Save Response](#save-response)
   * [Log the Default Fallback Intent](#log-the-default-fallback-intent)
   * [Enable Plugin](#enable-plugin)


## Introduction to Jovo Plugins

We added the Jovo Plugin feature in order to allow our users to customize the framework to their needs as well as to build features, which other members of our community can benefit from.

Plugins allow you to easily extend the Jovo Framework without having to mess with its core code and architecture. You can, for example, integrate your own analytics API or create a custom logger.

> [See example plugins here.](https://www.jovo.tech/plugins)

## How Jovo Plugins Work

The plugins work with middlewares, which all combined handle the complete request execution. As soon as the execution reaches the middleware your plugin is connected to, it will run your specified function. [Learn more about the Jovo framework architecture here](https://www.jovo.tech/docs/architecture).

Middleware | Description
--- | --- 
`setup` | First initialization of `app` object with first incoming request. Is executed once as long as `app` is alive
`request` | Raw JSON request from platform gets processed. Can be used for authentication middlewares.
`platform.init` | Determines which platform (e.g. `Alexa`, `GoogleAssistant`) sent the request. Initialization of abstracted `jovo` (`this`) object.
`platform.nlu'` | Natural language understanding (NLU) information gets extracted for built-in NLUs (e.g. `Alexa`). Intents and inputs are set.
`nlu` | Request gets routed through external NLU (e.g. `Dialogflow` standalone). Intents and inputs are set.
`user.load` | Initialization of user object. User data is retrieved from database.
`router` | Request and NLU data (intent, input, state) is passed to router. intentMap and inputMap are executed. Handler path is generated. 
`handler` | Handler logic is executed. Output object is created and finalized.
`user.save` | User gets finalized, DB operations.
`platform.output` | Platform response JSON gets created from output object.
`response` | Response gets sent back to platform.
`fail` | Errors get handled if applicable.

## Creating a Custom Logger

Since plugins feature is tailored for more advanced users, I will skip the creation of a Jovo project and jump right into the code example. The plugin we're going to create will be a simple logger to track Google's `Default Fallback Intent`.

The plugin I will show you here will be very small and written in JavaScript. To see a more fleshed out example check out the [`jovo-integrations`](https://github.com/jovotech/jovo-framework/tree/master/jovo-integrations) folder, which are all plugins as well:


To build our own plugin we have to create a class:

```javascript
class CustomLogging {

}
```

Inside that class we will add a constructor, which will be empty for now:

```javascript
class CustomLogging {
    constructor() {

    }
}
```

Next up is the core part of the plugin. The `install()` method is the place, where you connect to the middlewares:

```javascript
class CustomLogging {
    constructor() {

    }
    install(app) {
	
    }
}
```

### Save Response

The first middleware we want to connect to, is the `platform.output` middleware, to log our app's response as well as the state prior to the utterance of our user, which triggered the `Default Fallback Intent`. Every time the request execution gets to the `platform.output` middleware, our `saveOutput()` function will be executed as well.

```javascript
install(app) {
    app.middleware('platform.output').use(this.saveOutput.bind(this));
}

saveOutput(handleRequest) {

}
```

Our function will have a single input, the `handleRequest` object, which every function connected to a middleware will get. It has the following interface:

```typescript
export interface HandleRequest {

    /**
     * Current app instance
     */
    app: BaseApp;

    /**
     * Current host instance
     */
    host: Host;

    /**
     * Current jovo instance
     *
     * First initialization happens in 'platform.init' middleware
     */
    jovo?: Jovo;

    /**
     * Error
     */
    error?: Error;
}
```

We will need the `jovo` object, which is the same object we access using `this` inside we handler.

The first thing we will check for is, if the request came from the `GoogleAction` platform.

```javascript
saveOutput(handleRequest) {
    if (handleRequest.jovo.constructor.name === 'GoogleAction') {

    }
}
```

After that we get both the speech and, if it exists, the reprompt string and delete their `speak` tags:

```javascript
saveOutput(handleRequest) {
    if (handleRequest.jovo.constructor.name === 'GoogleAction') {
        let speech = handleRequest.jovo.$response.getSpeech().replace(/<\/?speak\/?>/g, '');
        let reprompt;
        if (handleRequest.jovo.$response.getReprompt()) {
            reprompt = handleRequest.jovo.$response.getReprompt().replace(/<\/?speak\/?>/g, '');
        }
    }
}
```

After that we also save the state and add everything to the `output` string, which we will add to the `constructor`:

```javascript
constructor() {
    this.output = '';
}
saveOutput(handleRequest) {
    if (handleRequest.jovo.constructor.name === 'GoogleAction') {
        let speech = handleRequest.jovo.$response.getSpeech().replace(/<\/?speak\/?>/g, '');
        let reprompt;
        if (handleRequest.jovo.$response.getReprompt()) {
            reprompt = handleRequest.jovo.$response.getReprompt().replace(/<\/?speak\/?>/g, '');
        }
        let state = handleRequest.jovo.getState() || '-';
        this.output = `\nspeech: ${speech} | reprompt: ${reprompt}\n`
        this.output += `\nState: ${state} | `;
    }
}
```

That's all we need to do here. Next up, is to check the incoming requests for the `Default Fallback Intent` and log our saved output with the user's utterance.

### Log the Default Fallback Intent

To do that we connect our plugin to the `platform.nlu` middleware, which will execute the `log()` function.

```javascript
install(app) {
    app.middleware('platform.output').use(this.saveOutput.bind(this));
    app.middleware('platform.nlu').use(this.log.bind(this));
}

log(handleRequest) {
    // do stuff
}
```

Inside the `log()` function we check for the platform, the `Default Fallback Intent`,add the user's utterance to the output, log the whole string and reset it back to and empty string.

```javascript
log(handleRequest) {
    if (handleRequest.jovo.constructor.name === 'GoogleAction') {
        let intentName = handleRequest.jovo.$request.getIntentName();
        if (intentName === 'Default Fallback Intent') {
            this.output += ` Raw Text: ${handleRequest.jovo.$request.toJSON().queryResult.queryText}\n`
            console.log(this.output);
            this.output = '';
        }
    }     
}
```

At the end our plugin looks like this:

```javascript
class CustomLogging {
    constructor() {
        this.output = '';
    }
    install(app) {
        app.middleware('platform.output').use(this.saveOutput.bind(this));
        app.middleware('platform.nlu').use(this.log.bind(this));
    }
    saveOutput(handleRequest) {
        if (handleRequest.jovo.constructor.name === 'GoogleAction') {
            let speech = handleRequest.jovo.$response.getSpeech().replace(/<\/?speak\/?>/g, '');
            let reprompt;
            if (handleRequest.jovo.$response.getReprompt()) {
                reprompt = handleRequest.jovo.$response.getReprompt().replace(/<\/?speak\/?>/g, '');
            }
            let state = handleRequest.jovo.getState() || '-';
            this.output = `\nspeech: ${speech} | reprompt: ${reprompt}\n`
            this.output += `\nState: ${state} | `;
        }
    }
    log(handleRequest) {
        if (handleRequest.jovo.constructor.name === 'GoogleAction') {
            let intentName = handleRequest.jovo.$request.getIntentName();
            if (intentName === 'Default Fallback Intent') {
                this.output += ` Raw Text: ${handleRequest.jovo.$request.toJSON().queryResult.queryText}\n`
                console.log(this.output);
                this.output = '';
            }
        }     
    }
}
```

### Enable Plugin

Last but not least, we add the plugin to our app using the `app.use()` function, which we can find at the top of our `./src/app.js` file:

```javascript
app.use(
    // other plugins, dbs, platforms, etc.
    new CustomLogging()
);
```

You can test the plugin either locally, where it will we logged to your console, or on [Lambda](https://aws.amazon.com/lambda/), which stores the logs on [Cloudwatch](https://aws.amazon.com/cloudwatch/). Here's an example how the logs might look like:

```text
speech: Hello World! What's your name? | reprompt: Please tell me your name.
State: TEST_STATE | Raw Text: I won't tell you my name
```

We at Jovo are really excited to see, which kind of plugins our community comes up with. By the way, the first ever Jovo plugin was created by [Cellular](https://github.com/cellular/jovo-plugin-raven).

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to create a custom logger as a Jovo plugin", "author": "kaan-kilic", "og-image": "https://www.jovo.tech/blog/wp-content/uploads/2018/06/tutorial-jovo-plugin.jpg" }-->
