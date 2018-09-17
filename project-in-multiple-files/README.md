# Structure your Voice App in Multiple Files

Learn how to divide your Jovo voice app logic into separate handlers to keep your code organized in multiple files.

* [Introduction](#introduction)
* [Separate File for each State](#separate-file-for-each-state)
* [Separate Handler for each Platform](#separate-handler-for-each-platform)

## Introduction

Voice application projects can get very big and also increasingly complex. At some point you might lose the overview, which should be the signal to start thinking of structuring your app differently to manage the increasing complexity.

Usually, the app logic of Jovo projects is in the `app.js` file, were intents and states are added to the handler with the `app.setHandler()` method. In this guide, we will explore how to separate the handler into different files, either by state or by platform:

* [Separate File for each State](#separate-file-for-each-state)
* [Separate Handler for each Platform](#separate-handler-for-each-platform)


## Separate File for each State

When you're using many different states and intents and store everything in one file, it can get messy quite fast.

The most convenient approaches is to split up each state into a separate file. You can import each file in your main file `app.js` and add them to your handler the following way:

```javascript
// File in ./states/play.js

module.exports = {
    playIntent: function() {
        this.tell('hey');
    }
}
```

```javascript
// Partial content of app.js

const PLAY_STATE = require('./states/play.js');

app.setHandler({
    'LAUNCH': function() {
        this.toStateIntent('PLAY_STATE', 'playIntent');
    },
    PLAY_STATE,
});
```

The state's name will be the name of the variable you imported the file to.

## Separate Handler for each Platform

There's also the possiblity to divide the application by platform. Jovo allows you to add three separate handlers.

The first one is the default handler, which is used with every template. It is used for both Amazon Alexa and Google Assistant and looks like this:

```javascript
app.setHandler({
    // States & intents
});
```

The other two are separate handlers for the different platforms:

```javascript
app.setAlexaHandler({
    // States & intents
});
app.setGoogleActionHandler({
    // States & intents
});
```

Incoming requests will be first mapped to the platform handlers. If the requested intent was not defined in there, the system will look for it in the universal handler, which works for both platforms.

Here's an example:

```javascript
app.setHandler({
    'LAUNCH': function() {
        this.tell('Hello user!');
    },
});
app.setAlexaHandler({
    'LAUNCH': function() {
        this.tell('Hello Alexa user!');
    },
});
```

The request from an Alexa device will be mapped to the Alexa handler and the system will respond with `Hello Alexa user!`, while the request from the Google Assistant will be mapped to the default handler, since a separate Google Action handler was not defined.

<!--[metadata]: { "description": "Learn how to divide your Jovo voice app logic into separate handlers to keep your code organized in multiple files." }-->
