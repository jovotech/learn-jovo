# Unhandled Intent

- [Introduction](#introduction)
- [State Unhandled Intent](#state-unhandled-intent)
- [Global Unhandled Intent](#global-unhandled-intent)
- [Intents to Skip Unhandled](#intents-to-skip-unhandled)

## Introduction

The `Unhandled` intent is used to catch incoming intens, which can't be found inside your `handler`. If you've created a dialogflow agent before you might have seen the `Default Fallback Intent` which is automatically mapped to the `Unhandled` intent as well.

## Unhandled Intent inside a State

The Jovo framework will look for the requested intent inside the current state first. If it can't find the intent there, it will go on searching for intent in the next outer scope and so on. You can stop that process using the [`Unhandled` intent inside a state](#state-unhandled-intent).

Let's say the incoming request contains a `PlayIntent`, but we're currently in the `DoNotPlayState` which does not have a `PlayIntent`, but globally there is one. Without the `Unhandled` state, the system would jump to the global scope and map the request to the `PlayIntent`. But with the `Unhandled` intent it would catch the request and remain in the current state:

```javascript
'DoNotPlayState': {
    'Unhandled': function() {
        // intent will be caught here
    },
    'TryAgainIntent': function() {
        // do stuff here
    }
},
'PlayIntent': function() {
    // do stuff here
}
```

There is also the possiblity to have nested states in which case the intent will be mapped to the first `Unhandled` intent it comes across:

```javascript
'FirstState': {
    'SecondState': {
        'DoNotPlayIntent': function() {
            // do stuff here
        }
    },
    'Unhandled': function() {
        // intent will be caught here
    }
},
'PlayIntent': function() {
    // do stuff here
}
```

## Global Unhandled Intent

The global `Unhandled` intent is the last line of defense. Any requested intent, which can't be found in any of the states, which also do not have `Unhandled` intents themselves, will be mapped to the global `Unhandled` intent.

Let's say the requested intent is called `PlayIntent`, but we don't have that intent neither in one of our states nor in the global scope. In that case the intent will be mapped to the global `Unhandled` intent, because there's also no `Unhandled` intent in one of the states.

```javascript
'Unhandled': function() {
    // intent will be caught here
},
'StateOne': {
    'StateTwo': {
        'DoNotPlayIntent': function() {
            // do stuff here
        }
    },
    'DoNotPlayIntent': function() {
        // do stuff here
    }
},
'DoNotPlayIntent': function() {
    // do stuff here
}
```

## Intents to Skip Unhandled

The Jovo framework also provides the possiblity to define intents, which won't be mapped to the `Unhandled` intent, if it can't be found inside a state. It allows you to make sure that the intent will be always caught globally.

You can define these intents inside your app's config the following way:

```javascript
let myIntentsToSkipUnhandled = [
    'CancelIntent',
    'HelpIntent',
    // other intents
];

const config = {
    intentsToSkipUnhandled: myIntentsToSkipUnhandled,
    // Other configurations
};

const app = new App(config);
```

Let's say we have the configuration above and the app is currently inside `StateTwo` and the `HelpIntent` was requested, but that intent was not defined inside `StateTwo`. Instead of the request being mapped to the `Unhandled` intent inside `StateTwo`, the global `HelpIntent` will be used.

```javascript
'StateTwo': {
    'PlayIntent': function() {
        // do stuff here
    },
    'Unhandled': function() {
        // do stuff here
    }
},
'HelpIntent': function() {
    // intent will be caught here
}