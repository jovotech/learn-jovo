# Step 8: The Final Steps

We're at the final step where before we improve the project's structure, add small required intents and look ahead at what else could be added to the project.

* [Required Intents](#required-intents)
    * [Help Intent](#help-intent)
    * [CancelIntent](#cancelintent)
    * [StopIntent](#stopintent)
* [Multiple Handlers](#multiple-handlers)
* [Next Steps](#next-steps)

## Required Intents

### Help Intent

In and outside the playback state our user should have the possibility to ask for help.

First we add it to our language model and delete the `AMAZON.HelpIntent` inside the `alexa` object:

```javascript
// model/en-US.json

{
    "name": "HelpIntent",
    "alexa": {
        "name": "AMAZON.HelpIntent",
        "samples": []
    },
    "phrases": [
        "help",
        "help me",
        "can you help me",
        "help please",
        "what can I do"
    ]
},
```

We add the intent to our intent map:

```javascript
// src/config.js

module.exports = {
    // other configurations
    myIntentMap = {
        // other intents
        'AMAZON.HelpIntent': 'HelpIntent'
    };
};
```
After that we add it to our handler:

```javascript
// src/app.js

HelpIntent() {
    this.ask('You can either listen to episode one or the latest episode or choose from a random list of episodes. Which one would you like to do?');
},
```

### CancelIntent

The user also has to be able to exit the app. For that we use the `AMAZON.CancelIntent`, which is already in our language model. On Google Action, the utterances *cancel* and *stop* will automatically close the app.

Inside our handler we simply tell the user *goodbye*.

```javascript
// src/app.js

'AMAZON.CancelIntent'() {
    this.tell('Alright, see you next time!');
},
```

### StopIntent

Amazon also wants us to have the `AMAZON.StopIntent` implemented. Since it technically has the same task as the `AMAZON.CancelIntent`, we will simply map the `AMAZON.StopIntent` to the `AMAZON.CancelIntent` in our intent map:

```javascript
// src/app.js

let myIntentMap = {
    // other intents
    'AMAZON.StopIntent': 'CancelIntent' 
};
```

## Multiple Handlers

Currently our handler inside the `app.js` file contains both platform-independent intents as well as platform-dependent intents, e.g. AudioPlayer requests, Amazon built-in intents, etc. Technically that works, but it's not really organized.

A better solution would be to split it up into three different handlers. A handler where we store the platform-independent intents, an handler specifically for Alexa intents and one for Google Action intents.

Here's how the separation would look like:

Handler | State | Intent 
:--- | :--- | :---
Generic | - | `LAUNCH`
&nbsp; | - | `NEW_USER`
&nbsp; | - | `ListIntent`
&nbsp; | - | `ChooseFromListIntent`
&nbsp; | - | `LatestEpisodeIntent`
&nbsp; | - | `FirstEpisodeIntent`
&nbsp; | - | `NextIntent`
&nbsp; | - | `PreviousIntent`
&nbsp; | - | `ResumeIntent`
&nbsp; | - | `HelpIntent`
Alexa | - | `AMAZON.PauseIntent`
&nbsp; | - | `AMAZON.CancelIntent`
&nbsp; | - | `AMAZON.PauseIntent`
&nbsp; | - | `AMAZON.LoopOnIntent`
&nbsp; | - | `AMAZON.LoopOffIntent`
&nbsp; | - | `AMAZON.RepeatIntent`
&nbsp; | - | `AMAZON.ShuffleOnIntent`
&nbsp; | - | `AMAZON.ShuffleOffIntent`
&nbsp; | - | `AMAZON.StartOverIntent`
&nbsp; | `AUDIOPLAYER` | `AlexaSkill.PlaybackStarted`
&nbsp; | `AUDIOPLAYER` | `AlexaSkill.PlaybackNearlyFinished`
&nbsp; | `AUDIOPLAYER` | `AlexaSkill.PlaybackFinished`
&nbsp; | `AUDIOPLAYER` | `AlexaSkill.PlaybackStopped`
&nbsp; | `AUDIOPLAYER` | `AlexaSkill.PlaybackFailed`
Google | `AUDIOPLAYER` | `GoogleAction.Finished`

```javascript
// src/app.js

app.setHandler({
    LAUNCH() {
        // ...
    },

    NEW_USER() {
        // ...
    },

    ListIntent() {
        // ...
    },
    ChooseFromListIntent() {
        // ...
    },

    LatestEpisodeIntent() {
        // ...
    },

    FirstEpisodeIntent() {
        // ...
    },

    NextIntent() {
        // ...
    },

    PreviousIntent() {
        // ...
    },

    ResumeIntent() {
        // ...
    },

    HelpIntent() {
        // ...
    }
)};

app.setAlexaHandler({
    'AMAZON.CancelIntent'() {
        // ...
    },

    'AMAZON.PauseIntent'() {
        // ...
    },

    'AMAZON.LoopOffIntent'() {
        // ...
    },

    'AMAZON.LoopOnIntent'() {
        // ...
    },

    'AMAZON.LoopOffIntent'() {
        // ...
    },

    'AMAZON.RepeatIntent'() {
        // ...
    },

    'AMAZON.ShuffleOffIntent'() {
        // ...
    },

    'AMAZON.ShuffleOnIntent'() {
        // ...
    },

    'AMAZON.StartOverIntent'() {
        // ...
    },

    AUDIOPLAYER: {
        'AlexaSkill.PlaybackStarted'() {
            // ...
        },

        'AlexaSkill.PlaybackNearlyFinished'() {
            // ...
        },

        'AlexaSkill.PlaybackFinished'() {
            // ...
        },

        'AlexaSkill.PlaybackStopped'() {
            // ...
        },

        'AlexaSkill.PlaybackFailed'() {
            // ...
        }
    }
});

app.setGoogleAssistantHandler({
    AUDIOPLAYER: {
        'GoogleAction.Finished'() {
            // ...
        }
    }
});
```

We can also go one step further and move the platform specific handlers to separate files.

We create an `alexa` folder inside the `app` folder and add a `handler.js` file. Inside that file we store the content of our `AlexaHandler`:

```javascript
// src/alexa/handler.js

const Player = require('../player.js');

module.exports = {
    'AMAZON.CancelIntent'() {
        // ...
    },

    'AMAZON.PauseIntent'() {
        // ...
    },

    'AMAZON.LoopOffIntent'() {
        // ...
    },

    'AMAZON.LoopOnIntent'() {
        // ...
    },

    'AMAZON.LoopOffIntent'() {
        // ...
    },

    'AMAZON.RepeatIntent'() {
        // ...
    },

    'AMAZON.ShuffleOffIntent'() {
        // ...
    },

    'AMAZON.ShuffleOnIntent'() {
        // ...
    },

    'AMAZON.StartOverIntent'() {
        // ...
    },

    AUDIOPLAYER: {
        'AlexaSkill.PlaybackStarted'() {
            // ...
        },

        'AlexaSkill.PlaybackNearlyFinished'() {
            // ...
        },

        'AlexaSkill.PlaybackFinished'() {
            // ...
        },

        'AlexaSkill.PlaybackStopped'() {
            // ...
        },

        'AlexaSkill.PlaybackFailed'() {
            // ...
        }
    }
}
```

We do the same for our `GoogleHandler`. Again, we create a `google` folder inside the `app` folder, add a `handler.js` file and move the content of our `GoogleHandler` over:

```javascript
// src/google/handler.js

const Player = require('../player.js');

module.exports = {
    AUDIOPLAYER: {
        'GoogleAction.Finished'() {
            // ...
        }
    }
}
```

Now we can import both files in our `app.js` file and place them inside the `AlexaHandler` and `GoogleHandler`:

```javascript
// src/app.js

// Rest of the app.js file above

const AlexaHandler = require('./alexa/handler.js');
const GoogleHandler = require('./google/handler.js');

app.setHandler({
    LAUNCH() {
        // ...
    },

    NEW_USER() {
        // ...
    },

    ListIntent() {
        // ...
    },

    ChooseFromListIntent() {
        // ...
    },

    LatestEpisodeIntent() {
        // ...
    },

    FirstEpisodeIntent() {
        // ...
    },

    NextIntent() {
        // ...
    },

    PreviousIntent() {
        // ...
    },

    ResumeIntent() {
        // ...
    },

    HelpIntent() {
        // ...
    }
});

app.setAlexaHandler(AlexaHandler);
app.setGoogleAssistantHandler(GoogleHandler);

// Rest of the app.js file below

```

## Next Steps

You've reached the end of the course. We're done. 

The last thing you have to do is switching out the audio files. These have to hosted on a HTTPS endpoint. We used [AWS S3](https://aws.amazon.com/s3/), but there are a number of other possible services for that.

Besides that, if you want to publish your app on [AWS Lambda](https://aws.amazon.com/lambda/), you have to switch to [DynamoDB](https://aws.amazon.com/dynamodb/), because Lambda does not allow you to make any changes to files on runtime. You can find a guide on how to do that in our [documentation](https://www.jovo.tech/docs/databases#dynamodb).

Obviously, there are also a lot more features we can add to the app, but that would make the course even longer than it already is.

One of those things, we cut because of time constraint, is visual output. Both Alexa and the Google Assistant support smart displays, where you could, for example, also list the random episodes we suggest our user.

Feel free to modify the podcast player as you wish. We would love to see what you come up with!

**If you have any questions or suggestions, you can reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "In this step, we make the final changes to our Podcast Player Alexa Skill and Google Action.", "author": "kaan-kilic" }-->



