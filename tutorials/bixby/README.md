# Build your first Samsung Bixby Capsule with Jovo

With the launch of `v3.0.0` of the Jovo Framework, we introduced a new platform integration for Samsung Bixby. It enables you to program highly scalable cross-platform voice applications for Bixby and use the countless integrations for the Jovo Framework, such as persistent user storage and Content-Management-Systems.

In this tutorial, you will start with an example project, showcasing the integration between Bixby and Jovo, and learn about the basic concepts of Samsung Bixby. In the end, you will have a working Bixby capsule, that, upon retrieving a name, will greet you with a welcome message.

-   [Introduction](#introduction)
    -   [About Bixby](#about-bixby)
    -   [About Jovo](#about-jovo)
    -   [About the platform integration](#about-the-platform-integration)
-   [Getting Started](#getting-started)
    -   [Setting up your Jovo app](#setting-up-your-jovo-app)
    -   [Installing Bixby Studio](#installing-bixby-studio)
    -   [Configuring your capsule](#configuring-your-capsule)
    -   [Running your first query](#running-your-first-query)
-   [Understanding the Underlying Principles](#integration-basics)
    -   [Models](#models)
        -   [Primitives](#primitives)
        -   [Actions](#actions)
    -   [Resources](#resources)
        -   [Base Configuration](#base-configuration)
        -   [Locale-specific Configuration](#locale-specific-configuration)
        -   [Layouts](#layouts)
        -   [Deployment Configuration](#deployment-configuration)
-   [Next Steps](#next-steps)
    -   [Training a new utterance](#training-a-new-utterance)
    -   [Testing the new Training Model](#testing-the-new-training-model)

## Introduction

### About Bixby

Bixby is the voice-driven virtual assistant developed by Samsung. Voice Applications for Bixby are called Capsules. If you're familiar with developing voice apps for platforms such as Alexa or Google Assistant, you might come across some similarities between the platforms, such as training the assistant with a natural language model to understand and train on user utterances. However, Bixby follows some majorly different development paradigms.
While development for other platforms may be imperative driven, Bixby implements a declarative driven development process by allowing you to describe your intents with models, rather than pure code. You can think of these models as an interface for your app logic, so Bixby knows when to expect what data.

### About Jovo

Jovo is an open-source framework, allowing developers to build powerful conversational apps for multiple platforms with one codebase. It also allows you to use platform-specific features, as well as a number of Jovo integrations for your application, such as databases, analytic tools and CMS.

### About the platform integration

With the Bixby platform integration for the Jovo Framework, you can outsource your code logic to your Jovo application, while maintaining your Bixby models inside your capsule. This allows you to develop precise capsule functionality, while still being able to develop for other platforms with the same codebase. This is possible by providing Bixby with remote endpoints, sending a request on every user utterance, if it matches an intent provided by your capsule. In that case, the Jovo instance acts as a server, receiving those requests and mapping them inside the Jovo Handler.

## Getting Started

### Setting up your Jovo app

To get started, the first thing you'll need is our Bixby Hello World Template. Not only does it come with our [Hello World Example](https://www.jovo.tech/templates/helloworld), it also includes all Bixby models required for the integration to work properly. You can download it using the following command:

```sh
$ jovo new bixby-hello-world --template bixby
```

Change your terminal's working directory to your new project folder and start your Jovo instance. If you don't have the Jovo CLI yet, you need to download it before using `npm`.

```sh
# Install Jovo CLI globally
npm install jovo-cli -g

# Change working directory to example project
cd bixby-hello-world

# Run Jovo voice app, optionally with --watch parameter
jovo run [-w]
```

This will start the development server of your voice app and give you a webhook url as an endpoint. Copy this url, you will need it in a second.

![Terminal](./img/terminal-jovo-run.png 'Copy your webhook url.')

### Installing Bixby Studio

Next up, you need to download the Bixby Developer Studio. It functions as an IDE and allows you to develop, test and publish your capsules. Although you can edit your capsule files in your own environment, we strongly recommend using Bixby Studio for all your work inside the capsule, as external changes can easily break it or lead to undesired behavior.

> You can download the Bixby Developer Studio [here](https://bixbydevelopers.com/).

After the installation has finished, start Bixby Studio and open the capsule by going to `File` -> `Open Capsule...` -> `./platforms/bixby/`.

![Bixby Studio](./img/open-capsule.png 'Open your capsule.')

![Bixby Studio](./img/open-capsule-folder.png 'Open your capsule.')

### Configuring your capsule

In the left panel, you should now see your capsule `playground.jovo_test` with a bunch of folders and files. Don't worry, we'll go through each of them in a minute. For now, open the training tool by selecting the file `./resources/en/training` and compile your training model by clicking on `Compile NL Model`. This will generate a plan for every utterance, defining the exact route Bixby is going to take and what inputs to gather, when the corresponding utterance is triggered.

![Bixby Studio](./img/training-temp.png 'Compile your NL Model')

Now you need to provide Bixby with a remote url, so it knows where to send requests to upon a user prompt. To do this, open the file `./resources/base/capsule.properties` and paste your copied webhook url into the respective property field.

```
# capsule.properties

config.default.remote.url=https://webhook.jovo.cloud/xxxxxxxx-xxxx-xxx-xxxx-xxxxxxxxxxxx
```

### Running your first query

To finally test your capsule, open the Bixby Simulator, which allows you to test your capsule in a variety of simulated scenarios and analyze the execution graph of your last tested utterance. You can launch it by going to `View` -> `Open Simulator` or by clicking on the Bixby Simulator button:

![Terminal](./img/simulator-icon.png 'Copy your webhook url.')

Enter "open jovo test capsule" as your first utterance and select `Run NL` to run it.

![Terminal](./img/simulator-first-utterance.png 'Copy your webhook url.')

Congrats, you just ran your first query in your capsule. Now, let's go into detail of what the individual files of your capsule mean.

## Understanding the Underlying Principles

In the left panel, you should see your capsule `playground.jovo_test` with the following file structure:

```
assets
│   jovo-logo-16x16.png
|
models
|
└───Jovo
│   │
│   └───actions
│   |   │   JovoPlayAudioAction.model.bxb
│   │
│   └───primitives
|   |   |   JovoSessionId.model.bxb
|   |   |   JovoSpeech.model.bxb
|   |   |   JovoState.model.bxb
|   |
│   └───structures
|   |   |   JovoLayout.model.bxb
|   |   |   JovoResponse.model.bxb
|   |   |   JovoSessionData.model.bxb
│
└───actions
|   │   LaunchAction.model.bxb
|   │   MyNameIsAction.model.bxb
│
└───primitives
|   │   NameInput.model.bxb
|
resources
|
└───base
│   │   capsule.properties
│   │   endpoints.bxb
|
└───en
│   │
│   └───layout
│   │   |    Result.view.bxb
|   |
│   │   training
│   │   capsule-info.bxb
│   │   jovo-test.hints.bxb
|
capsule.bxb
```

Starting with `assets/`, this folder only contains one image, which serves as an icon to represent the capsule, similar to icons for an Alexa skill. Its path is used in `en/capsule-info.bxb`, so if you want to change the icon, you have to change it there as well.

### Models

Next up, the `models/` folder contains all the models necessary for the capsule to work properly. Models are the building blocks of a Bixby Capsule. They determine what and how Bixby can accomplish tasks, with desired inputs and outputs. However, models do not contain any actual code, they rather act as an interface for your code logic, so Bixby knows how to handle your data upon a user prompt.

Models can be split up into concepts and actions. A concept can be explained as an entity representing a "thing", such as a name, an animal, or something more complex like a restaurant order. There are two kinds of concepts: primitives and structures. Actions on the other hand act as an interface for an operation that Bixby can execute. With the help of concepts, they define a ruleset of inputs and outputs to achieve a desired result upon a user prompt.

> Learn more about models [here](https://jovo.tech/docs/samsung-bixby#models).

#### Primitives

Located in `models/primitives/`, primitives are the simplest building blocks of a Bixby Capsule and can contain simple data like text, numbers or boolean values. Primitives can be used to form structures, inputs or outputs for actions, for example.

In `models/primitives/`, you can find a primitive `NameInput`.

```bxb
text (NameInput) {
  description (A name to display in a welcome message.)
}
```

`NameInput` is a primitive of type `text`, meaning it can hold a range of text based values, in this case names. As mentioned, we don't write any actual code logic here, we rather tell Bixby what type to expect, when we speak of names. The actual values will be entered later in the [training](#training) section.

> Learn more about primitives [here](https://jovo.tech/docs/samsung-bixby#primitives).

#### Actions

In `models/actions/`, you can find all model files for actions. These help Bixby understand how to act upon a specific user prompt and what to expect as inputs and outputs. In this project, you will see two already defined actions, `LaunchAction` and `MyNameIsAction`. As you can probably tell, these are intent-oriented, meaning that for every intent in your Jovo app, you have to define an action with corresponding inputs and output.

```bxb
action (LaunchAction) {
  description (Opens the voice application for the first time.)
  type (Search)
  output (JovoResponse)
}
```

`LaunchAction` is very simple. It is the entry point to your voice application, so it won't expect any inputs. As for its output, in most cases it will be sufficient to use `JovoResponse` here. `JovoResponse` is a predefined Jovo model located in `models/Jovo/structures/` and describes an interface for the communication between your Jovo app and your Bixby capsule. It contains important information like speech output and session data and is the most basic output object you can use.

If you want to return a more specific response, you can create a new structure and extend `JovoResponse` by adding your own structure properties. If your own response does not add any attributes to the JovoResponse, use the role-of feature.

> Learn more about structure inheritance [here](https://jovo.tech/docs/samsung-bixby#structures).

```bxb
action (MyNameIsAction) {
  description (Collects a name from the user and returns a welcome message.)
  type (Search)

  collect {
    input (_JOVO_INPUT_name) {
      type (NameInput)
      min (Required)
      max (One)
    }

    input (_JOVO_PREV_RESPONSE_) {
      type(JovoResponse)
      min (Required)
      max (One)
    }
  }
  output (JovoResponse)
}
```

`MyNameIsAction` is a bit more complicated, as it expects specific inputs to function correctly. Similar to `LaunchAction`, it has a description, an [action type](https://bixbydevelopers.com/dev/docs/reference/type/action.type) and an output field of type `JovoResponse`. However, it features a new property `collect`. This field specifies any number of inputs to be collected and evaluated, before executing the action. Here we have two input properties: `_JOVO_INPUT_name` and `_JOVO_PREV_RESPONSE_`.

`_JOVO_INPUT_name` defines the name input, which will be used to return a welcome message to your user. Every input features a set of properties, describing meta data for the input itself, so Bixby knows how to handle it correctly. The most common properties include `type`, `min` and `max`. While `type` describes the inputs type, which can be any concept, be it a primitive or a structure (in this case the primitive `NameInput`), `min` and `max` specify the inputs cardinality, in this case that exactly one name is required.

All capsule-dependent inputs have to be prefixed with `_JOVO_INPUT_`, followed by your input identifier, which will be available in your Jovo app by accessing `this.$inputs[INPUT_IDENTIFIER]`.

Although `MyNameIsAction` only collects one user input, it requires one more input property: `_JOVO_PREV_RESPONSE_` of type `JovoResponse`. This one is a bit tricky.

To keep session data request-persistent, we need to include it in the conversation between Bixby and Jovo. That's why we have a `_JOVO_SESSION_DATA_` field in our `JovoResponse` model. To include those in a new request, we tell Bixby to require an input property of type `JovoResponse`. When Bixby receives a `JovoResponse`, it stores it into its own cache. Now, when the user triggers `MyNameIsAction`, Bixby will look for any available inputs matching this type. Since the user didn't provide it, Bixby takes it from our LaunchAction, hence including the then returned session data in the new request.

### Resources

The `resources/` folder mainly contains your capsule configuration and locale-oriented dialog and views.

#### Base Configuration

`base/` features only your capsule's main configurations. For our example project, there are only two files present, `capsule.properties` and `endpoints.bxb`.

You can use `capsule.properties` to store your capsules configurations, such as endpoints, permission scopes or environment modes. Here you will paste your webhook url, that will later act as your capsules remote url. This will become important in a few steps.

`endpoints.bxb` defines your action endpoints. As mentioned, Bixby uses your models to outline your capsule and to get an idea of what data to expect and how to handle it. For the actual code logic, Bixby provides two ways for you to implement your actions: local endpoints in the form of javascript files and remote endpoints. Remote endpoints allow you to outsource your code logic to a remote server and provide it's endpoint to Bixby, which, upon a user prompt, will send a post request to your server, where you can handle the action and return a valid response. The platform integration uses that feature to allow communication between the Jovo voice app and your capsule.

```bxb
endpoints {
  action-endpoints {
    action-endpoint (LaunchAction) {
      remote-endpoint ("{remote.url}") {
        method (POST)
      }
    }
    action-endpoint (MyNameIsAction) {
      remote-endpoint ("{remote.url}?intent=MyNameIsIntent") {
        method (POST)
      }
    }
  }
}
```

For every action, you have to provide a remote endpoint, which takes your remote url from `capsule.properties` and sends a post request to the provided url. For your `LaunchAction`, its sufficient to just use the remote url, however, for an intent-based action, such as `MyNameIsAction`, you have to specify which intent to trigger by providing a query parameter `intent` in the endpoint's url.

#### Locale-specific Configuration

Aside from your capsule configuration, `resources/` also contains locale-specific files, such as [training](#training), dialog and locale-specific information about your capsule. In this example project, we only match one locale, `en`.

The `training` file contains all of your capsule's natural language models, which are being used by Bixby to understand your user prompts and how to differentiate between inputs. For every training sample, a goal must be specified, telling Bixby what action to execute on that sample. Furthermore, you can specify inputs with associated nodes or even routes for Bixby to collect additional information.

> Learn more about training [here](https://jovo.tech/docs/samsung-bixby#training).

`capsule-info.bxb` holds all of your capsule's locale-specific information, such as it's display name, a description and certain search keywords.

> Learn more about `capsule-info.bxb` [here](https://bixbydevelopers.com/dev/docs/reference/type/capsule-info).

To be marketplace compliant, your capsule needs to provide hints in a `.hints.bxb` file. Hints are used by Bixby to provide your users with utterance suggestions, so they get an idea of how your capsule works.

> Learn more about hints [here](https://bixbydevelopers.com/dev/docs/reference/type/hints).

#### Layouts

The last thing to cover in `en/` is the `layouts/` folder, which will be used for views and dialog. Bixby uses dialog to communicate back to the user, to inform them about certain results or to request additional information. Views on the other hand form your capsule's visual user interface, capable of providing text, buttons, input fields and lists, for example.

For our example project, we only need one view of type `result-view`, which is located inside `resources/layout/`.

```bxb
result-view {
  match {
    JovoResponse (response)
  }

  message {
    template("#{value (response._JOVO_SPEECH_)}")
  }
}
```

This view matches a result of type `JovoResponse`, meaning that Bixby will try to return this view, whenever a `JovoResponse` is returned by our voice app. In that case, Bixby looks into the views property and tries to match the required data to the response object. One of those properties is our dialog, or the `message` property, which will fetch the speech output from our response by accessing `response._JOVO_SPEECH_`.

The example project doesn't come with any rendered elements, however, you can learn more about adding layouts to your capsule [here](https://jovo.tech/docs/samsung-bixby#layouts).

> You can learn more about dialog and views [here](https://jovo.tech/docs/samsung-bixby#layouts).

#### Deployment Configuration

Last, but not least, `capsule.bxb` acts as a place for metadata about your capsule. It specifies important information about your capsule, such as it's version, library imports, runtime flags and deployment targets.

> Learn more about `capsule.bxb` [here](https://bixbydevelopers.com/dev/docs/reference/type/capsule).

## Next Steps

Now that you have a basic understanding of how Bixby and the platform integration work, we want Bixby to recognize our name, when we launch the capsule and answer the then returned question. It is possible that Bixby understands your name out of the box, however, to make sure it does, we have to modify it's training model.

### Training a new utterance

Training your capsule is similar to the interaction model for Alexa. For every action, you have to provide a number of sample utterances, so Bixby can train on them to understand your user prompts and how to differentiate between inputs. Let's go ahead and create a new training phrase for your name.

For that, go ahead, select the `training` file and add the training sample "my name is {your-name}".

![Training](./img/training-new-sample.png 'Enter a new training sample.')

![Training](./img/training-draft.png 'Enter a new training sample.')

Now, we have to specify what goal we want to achieve with this phrase by entering "MyNameIsAction" into the `Goal` input field. Lastly, we want Bixby to recognize our name. For that, click on your name inside the `NL` text field. A small window should appear, where you can enter the node you want to associate with the selected value.

![Training](./img/training-input.png 'Specify your Goal.')

Since our `MyNameIsAction` requires a property of type `NameInput`, our node should also be of type `NameInput`.

![Training](./img/training-nameinput.png 'Specify an input node.')

Now, compile your modified training model again by clicking on "Done" -> "Save" -> "Compile NL Model".

### Testing the new Training Model

If you go into the simulator again and launch the capsule with "open jovo test capsule", you can now enter the follow-up phrase "my name is {your name}" and the capsule should greet you with a neat little welcome message.

![Testing](./img/simulator-success.png 'Your capsule should now return a welcome message.')

And that's it, you made it to the end. Now you can play around with more inputs, add other actions or start with using Jovo integrations.

Want to know more?

-   [Jovo Bixby Docs](https://jovo.tech/docs/samsung-bixby)
-   [Official Bixby Documentation](https://bixbydevelopers.com/)

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to develop Samsung Bixby Capsules with the platform integration for the Jovo Framework.", "author": "ruben-aegerter", "tags": "Samsung, Bixby, Platform, Integration" }-->
