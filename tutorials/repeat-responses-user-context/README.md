# How to Repeat Responses using the Jovo User Context Object

Learn how to easily add a repeat functionality to your voice application using the [Jovo User Context](https://www.jovo.tech/docs/data/user#context) (available since Jovo v1.2).

- [Why you should add a Repeat Intent](#why-you-should-add-a-repeat-intent)
- [How the User Context works](#how-the-user-context-works)
    - [Configuration](#configuration)
- [Implementing the Repeat Functionality](#implementing-the-repeat-functionality)
    - [Creating a Jovo Project](#creating-a-jovo-project)
    - [Adding the RepeatIntent to your Language Model](#adding-the-repeatintent-to-your-language-model)
    - [Deploying your Language Models](#deploying-your-language-models)
    - [Implementing the RepeatIntent in your Code](#implementing-the-repeatintent-in-your-code)

### Code

You can find the full code example of this tutorial here: [jovo-templates/tutorials/repeat](https://github.com/jovotech/jovo-templates/tree/master/tutorials/repeat). Or download it using the Jovo CLI:

```sh
$ jovo new --template tutorials/repeat
```

## Why you should add a Repeat Intent

![](./img/repeat-intent-example.png)

One of the biggest difficulties of recreating human to human conversations with an assistant is the fact that humans have context. Conversations are different depending on the location, current news, and so much more. These things are truly hard to implement, but there's one thing, actually the simplest as well as most essential thing you can easily add to your voice application.

I am talking about repeating yourself if the person on the other end missed what you just said. One of the great things about voice apps is that they allow you to interact with them while you're busy with your hands (e.g. cleaning up). However, this advantage may also be one of the biggest disadvantages of voice: You're never sure if your user is actively listening or doing something else as well. This is why it is strongly encouraged to add the functionality to repeat yourself with a _RepeatIntent_. It's such an useful feature, that Amazon created a built-in intent (_AMAZON.RepeatIntent_) for that and Google listed it as a [best practice](https://developers.google.com/actions/assistant/best-practices#let_users_replay_information).

As easy as it sounds, until today it was still a manual task to store all the necessary information in session attributes or a database so that they are later available to use. The [newly introduced](https://www.jovo.tech/blog/jovo-framework-v1-2/) User Context feature makes the implementation of the repeat feature easier.

## How the User Context works

The Jovo [User Context](https://www.jovo.tech/docs/data/user#context) automatically saves the essential data of the past interaction pairs (request and response) in your database. This includes the **intent**, **state**, **inputs** (slots/entities), **output speech**, **reprompt**, and **timestamp**.

The pairs are stored inside an array, which has the most recent pair at index `0` and the least recent at index `size - 1`. For example, you can get the latest output speech using either `this.$user.context.prev[0].response.speech` or `this.$user.getPrevSpeech(0)`.

By default, the Jovo Context feature isn't implemented. We are going to do this in the next step.

### Configuration

You can enable user context in your `config.js` file:

```javascript
// config.js

module.exports = {
    // Other configurations
    
    user: {
        context: true,
    },
};
```

By default, only the last interaction pair is stored in the database (which is enough for [implementing the repeat functionality](#implementing-the-repeat-functionality) below).

You are also able to freely decide how many interaction pairs, as well as what exactly you want to save, by changing your application's configuration, `config.js`, which has the following default values:

```javascript
// config.js

module.exports = {
    // Other configurations
    
    user: {
        context: {
            enabled: false, // Set this to true
            prev: {
                size: 1,
                request: {
                    intent: true,
                    state: true,
                    inputs: true,
                    timestamp: true,
                },
                response: {
                    speech: true,
                    reprompt: true,
                    state: true,
                },
            },
        }
    },
};
```

The `size` value defines how many pairs are saved. To disable the storage of certain elements, simply change their value to `false`.

Here's an example:

```javascript
// config.js

module.exports = {
    // Other configurations
    
    user: {
        context: {
            enabled: true,
            prev: {
                size: 3,
                request: {
                    timestamp: false,
                },
                response: {
                    state: false,
                },
            },
        }
    },
};
```

## Implementing the Repeat Functionality

If you haven't installed the Jovo CLI yet, please do, since I will make use of its features to save us time. You can learn more about the installation [here](https://www.jovo.tech/framework/docs/installation#-jovo-cli-installation).

```sh
$ npm install -g jovo-cli
```

In addition to that, I will create a new project in this tutorial, so if you want to add the _RepeatIntent_ to an exisiting project simply skip ahead to ["Adding the RepeatIntent to your Language Model"](#adding-the-repeatintent-to-your-language-model).

### Creating a Jovo Project

Alright, let's create a new project and initialize both Amazon Alexa and the Google Assistant as our platforms. We will just call it RepeatTutorial:

```sh
# Option 1: Create blank project
$ jovo new RepeatTutorial

# Option 2: Complete tutorial source code
jovo new RepeatTutorial --template tutorials/repeat

# Go into project directory
$ cd RepeatTutorial
```

### Adding the RepeatIntent to your Language Model

To spare us the hassle of adding the `RepeatIntent` to both Dialogflow and the Alexa Developer Console, we will simply add the intent to the [Jovo Language Model](https://www.jovo.tech/docs/model).

We will use the built-in intent `AMAZON.RepeatIntent` by Amazon and create a `RepeatIntent` on Dialogflow. Open the language model in your project's `models` folder and add the following to the `intents` array:

```javascript
{
    "invocation": "my test app",
    "intents": [
        {
            "name": "RepeatIntent",
            "alexa": {
                "name": "AMAZON.RepeatIntent"
            },
            "phrases": [
                "repeat",
                "could you repeat that",
                "what",
                "say that again"
            ]
        },
        {
            "name": "HelloWorldIntent",
            "phrases": [
                "hello",
                "say hello",
                "say hello world"
            ]
        },
		// More intents
```

Now we can use the Jovo CLI to build the language model for both platforms:

```sh
$ jovo build
```

### Deploying your Language Models

Next up, we're going to deploy the language models to their respective developer portals.

To deploy your Alexa Skill you have to install and initialize the [ASK CLI](https://developer.amazon.com/docs/smapi/quick-start-alexa-skills-kit-command-line-interface.html) first:

```sh
# Install ASK CLI
$ npm install -g ask-cli

# Initialize ASK CLI
$ ask init
```

That's it. You can now deploy your Alexa Skill to the developer portal:

```sh
$ jovo deploy
```

Deploying to Dialogflow takes a little bit more work, since you have to go through a bigger [authentication process](https://dialogflow.com/docs/reference/v2-auth-setup). But the Jovo CLI offers you a workaround, by letting you export your Dialogflow agent as a zip file, which you can then import at the Dialogflow website. After running `$ jovo deploy`, you should be able to find the zip file under `/platforms/googleAction/` folder.

To import the zip you have to go an existing agents (create a new one real quick) settings page and select the **Export and Import** tab: 

![](./img/dialogflow_agent_settings_importMarked.png)

After that choose the **Restore from ZIP** option and upload your zip file: 

![](./img/dialogflow_agent_import.png)

### Implementing the RepeatIntent in your Code

With version 1.2 of the Jovo framework you can simply use the `repeat()` method to repeat your applications last response. Let's add that together with the `RepeatIntent` itself to our handler:

```javascript
RepeatIntent() {
    this.repeat();
}
```

If you want to access the speech and reprompt directly, you can also use the user class' getter methods:

```javascript
RepeatIntent() {
    let speech = this.$user.getPrevSpeech(0);
    let reprompt = this.$user.getPrevReprompt(0);
    
    this.ask(speech, reprompt);
}
```

Alright, that's pretty much it for the `RepeatIntent`.


**Any questions? You can reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to add the repeat functionality to your voice app", "author": "kaan-kilic", "tags": "Output", "og-image": "https://www.jovo.tech/blog/wp-content/uploads/2018/06/repeat-intent-jovo-1.jpg" }-->
