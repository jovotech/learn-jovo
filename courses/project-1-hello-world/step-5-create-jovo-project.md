# Step 5: Create a Jovo Project

In this lecture, you will learn how to create a new voice app project with Jovo, an open source development framework for Alexa Skills and Google Actions.

* [Introduction to Jovo](#introduction-to-jovo)
* [Installation](#installation)
* [Creating a New Project](#creating-a-new-project)
* [Understanding the Jovo App Logic](#understanding-the-jovo-app-logic)
* [Next Steps](#next-steps)

  

## Introduction to Jovo

Jovo is a simple and open source framework to build cross-platform voice applications. Instead of having to build distinct apps for your Alexa Skill and Google Action, you can now have one platform-agnostic source of code that works with both voice APIs. Throughout this course, you will learn how to use the Jovo Framework step by step. If you're interested in learning more about Jovo now, here are some resources:

* [Jovo Framework Repository on GitHub](https://github.com/jovotech/jovo-framework/) ⭐️
* [Jovo Framework Documentation](https://www.jovo.tech/docs/)
* [Join our Newsletter](https://www.jovo.tech/newsletter)

## Installation

The best way to get started with Jovo is to install the Jovo Command Line Tools ([see the GitHub repository](https://github.com/jovotech/jovo-cli)), as it makes it easy to create new projects from templates. [See our documentation for more other ways to get started with Jovo](https://www.jovo.tech/docs/installation).

You can install the Jovo CLI by using the following command:

```sh
$ npm install -g jovo-cli
```

After the installation, you can test if everything worked with the following command:

```sh
$ jovo
```

This should look like this:

```text

  Commands:

    help [command...]             Provides help for a given command.
    exit                         Exits application.
    new [options] [directory]      Create a new Jovo project
    init [options] [platform]      Initializes platform-specific projects in
                                 app.json.
    build [options]               Build platform-specific language models
                                 based on jovo models folder.
    deploy [options]              Deploys the project to the voice
                                 platform.
    get [options]      		 	  Downloads an existing platform project
                                 into the platforms folder.
    run [options]   		 	  Runs a local development server
                                 (webhook).
```

Great! Now you can use the command line to easily create new Jovo projects.

## Creating a New Project

Let's create a new project. You can see from the console output above that it's possible to create new projects with the "new" command, which will clone the [Jovo Sample App](https://github.com/jovotech/jovo-sample-voice-app-nodejs) (default template) into the specified directory:

```sh
$ jovo new HelloWorld
```

```text

  I'm setting everything up

   √ Creating new directory /HelloWorld
   √ Downloading and extracting template helloworld
   √ Installing npm dependencies

  Installation completed.
```

Success!

## Understanding the Jovo App Logic

Let's take a look at the code provided by the sample application. For now, you only have to touch the [app.js](https://www.jovo.tech/docs/app-js) file. This is where the logic will happen. The folder structure ([see the docs](https://www.jovo.tech/docs/project-structure)) looks like this:

![](./img/folder-structure-models.png)

Let's take a look at the lower part first: The handler is where you will spend most of your time when you're building the logic behind your voice app. It already has a "HelloWorldIntent," as you can see below:

```javascript
app.setHandler({
    LAUNCH() {
        this.toIntent('HelloWorldIntent');
    },

    HelloWorldIntent() {
        this.ask('Hello World! What\'s your name?', 'Please tell me your name.');
    },

    MyNameIsIntent() {
        this.tell('Hey ' + this.$inputs.name.value + ', nice to meet you!');
    },
});
```

What's happening here?

When your app is opened, it triggers the [LAUNCH](https://www.jovo.tech/docs/routing/intents#launch) intent, which contains a [toIntent](https://www.jovo.tech/docs/routing#tointent) call to switch to the HelloWorldIntent. Here, the ask method is called to ask for the user's name. After they answer, the MyNameIsIntent gets triggered, which greets them with their name.

## Next Steps

We have the voice app platform projects and the source code in place. In the last step of this course, we're connecting the code to the voice APIs from Amazon Alexa and Google Assistant (Dialogflow) to receive our first "Hello World":

> [Step 6: Hello World!](./step-6-hello-world.md)


<!--[metadata]: { "description": "In this lecture, you will learn how to create a new voice app project with Jovo, an open source development framework for Alexa Skills and Google Actions.", "author": "jan-koenig" }-->
