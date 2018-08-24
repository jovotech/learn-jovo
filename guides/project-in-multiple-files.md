# Structure your Voice Application in multiple Files

- [Structure your Voice Application in multiple Files](#structure-your-voice-application-in-multiple-files)
    - [Introduction](#introduction)
    - [Seperate File for each State](#seperate-file-for-each-state)
    - [Seperate Handler for each Platform](#seperate-handler-for-each-platform)

## Introduction

Voice application projects can get very big and also increasingly complex. At some point you might loose the overview, which should be the signal to start thinking of structuring your app differently to manage the increasing complexity.

## Seperate File for each State

One of the bigger problems can be the different states and their intents.
The most convenient approaches is to split up each state into a seperate file. You can import each file in your main file `app.js` and add them to your handler the following way:

```javascript
// ./states/play.js
module.exports = {
    playIntent: function() {
        this.tell('hey');
    }
}
```

```javascript
const PLAY_STATE = require('./states/play.js');

app.setHandler({
    'LAUNCH': function() {
        this.toStateIntent('PLAY_STATE', 'playIntent');
    },
    PLAY_STATE,
});
```

The state's name will be the name of the variable you imported the file to.

## Seperate Handler for each Platform

There's also the possiblity to divide the application by platform. Jovo allows you to add three seperate handlers.

The first one is the default handler, which is used with every template. It is used for both Amazon Alexa and Google Assistant and looks like this:

```javascript
app.setHandler({
    // states & intents
});
```

The other two are seperate handlers for the different platforms:

```javascript
app.setAlexaHandler({
    // states & intents
});
app.setGoogleActionHandler({
    // states & intents
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

The request from an Alexa device will be mapped to the Alexa handler and the system will respond with `Hello Alexa user!`, while the request from the Google Assistant will be mapped to the default handler, since a seperate Google Action handler was not defined.
