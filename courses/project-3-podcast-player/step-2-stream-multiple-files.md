# Step 2: Streaming Multiple Files in a Row

In the previous [Step 1: Streaming an Audio File](./step-1-stream-audio-file.md), we played a single audio file on both Amazon Alexa and Google Assistant. Let's dive deeper. Before we can stream multiple files in a row we need at least some understanding how the respective audio players work.

* [The Alexa AudioPlayer](#the-alexa-audioplayer)
    * [AudioPlayer Requests](#audioplayer-requests)
    * [Expected Previous Token](#expected-previous-token)
    * [PlaybackNearlyFinished](#playbacknearlyfinished)
* [The Google Assistant Media Response](#the-google-assistant-media-response)
* [Next Step](#next-step)
  
## The Alexa AudioPlayer

> [Learn more about the Alexa AudioPlayer interface](https://www.jovo.tech/docs/amazon-alexa/audioplayer).

The Alexa AudioPlayer interface is needed to stream long-form audio with your skill, by sending out *directives* to start and stop the audio stream as well as notifying us about changes to the audio playback's state.

The only *directive* we currently need is the `play` directive, which we already used in the previous step to stream the audio file. After sending that directive, the current session ends and our skill will get registered as the most recent entity to play long-form audio. As a result, the user is now able to trigger the Alexa AudioPlayer's built-in intents, which offer the user control capabilities, without using our skill's invocation name, for example:

* `Alexa, pause` or `Alexa, stop` trigger the `AMAZON.PauseIntent`
* `Alexa, resume` triggers the `AMAZON.ResumeIntent`
* `Alexa, next` triggers the `AMAZON.NextIntent`

We will handle these intents in a later step.

### AudioPlayer Requests

Currently far more interesting are the AudioPlayer requests that our skill will receive while streaming the audio file. These requests are used to notify us about changes to the playback state:

* `PlaybackStarted`: Sent after starting to stream the audio file.
* `PlaybackFinished`: Sent after the stream finished.
* `PlaybackStopped`: Sent if the user manually pauses the audio stream (for example by saying `Alexa, ...`).
* `PlaybackFailed`: Sent after Alexa encounters an error while attempting to play the audio file.
* `PlaybackNearlyFinished`: Sent when the current audio file is almost finished.
  
Our skill will receive at least one of the requests no matter what, which means we have to be able to handle the response (which can be just simply ending the session), otherwise, we will encounter an error, which has already happened to us in step one.

To make handling these requests easier for us, Jovo maps them to built-in intents inside an `AUDIOPLAYER` state:

```javascript
// src/app.js

AUDIOPLAYER: {
   'AlexaSkill.PlaybackStarted'() {

   },

   'AlexaSkill.PlaybackNearlyFinished'() {

   },

   'AlexaSkill.PlaybackFinished'() {

   },

   'AlexaSkill.PlaybackStopped'() {

   },

   'AlexaSkill.PlaybackFailed'() {

   }
},
```

For us, the `PlaybackNearlyFinished` request is the more important one. It is used to enqueue the next audio stream, which will start to play as soon as the current audio stream finishes. Besides the audio files URL and a token, we will need the current stream's token as well, to prevent the following kind of situation:

### Expected Previous Token

```text
Episode A is currently playing and the system enqueues episode B, which is the next one in chronological order.

Before episode A finished, the user invokes one of the built-in intents and switches to episode B by themselves.

Now we have episode B currently playing and enqueued as the next file as well.
```

To prevent that we specify the expected previous token when we enqueue the next audio file. In that case, it would work like this:

```text
Episode A is currently playing and the system enqueues episode B. Before A is finished the user switches to episode B by themselves.

After episode B ended, the system compares the current stream's token with the expected token we specified when we first enqueued episode B.

It realizes that they are different and it does not play episode B again, but rather enqueues the next episode, which would be episode C.
```

Currently we simply use the string `token` as our token, since we don't have a system in place that we could use for that. We will fix that at the end of the project.

### PlaybackNearlyFinished

Since our goal in this step is to play another file after the first one finished, the most interesting audio player request is the `PlaybackNearlyFinished` one. That request is used to notify us that it's time to enqueue the next song. We set the expected token and enqueue the file using its URL and a new token:

```javascript
// src/app.js

'AlexaSkill.PlaybackNearlyFinished'() {
    const secondEpisode = 'https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55';

    this.$alexaSkill.$audioPlayer
        .setExpectedPreviousToken('token')
        .enqueue(secondEpisode, 'token2');
},
```

Your handler should currently look like this:

```javascript
// src/app.js

app.setHandler({
    LAUNCH() {
        const episode = 'https://traffic.libsyn.com/voicebot/Jan_Konig_on_the_Jovo_Open_Source_Framework_for_Voice_App_Development_-_Voicebot_Podcast_Ep_56.mp3';

        if (this.isAlexaSkill()) {
            this.$alexaSkill.$audioPlayer
                .setOffsetInMilliseconds(0)
                .play(episode, 'token');
        } else if (this.isGoogleAction()) {
            this.$googleAction.$mediaResponse
                .play(episode, 'Episode 56');
        }

        this.tell('Enjoy');
    },

    AUDIOPLAYER: {
        'AlexaSkill.PlaybackStarted'() {

        },

        'AlexaSkill.PlaybackNearlyFinished'() {
            const secondEpisode = 'https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55';

            this.$alexaSkill.$audioPlayer
                .setExpectedPreviousToken('token')
                .enqueue(secondEpisode, 'token2');
        },

        'AlexaSkill.PlaybackFinished'() {

        },

        'AlexaSkill.PlaybackStopped'() {

        },

        'AlexaSkill.PlaybackFailed'() {

        }
    },
});
```

Save the file and run the Jovo Webhook. It's time to test. Open up the Jovo Debugger and press the **LAUNCH** button, but don't forget to change your device back to an Alexa Echo.


## The Google Assistant Media Response

> [Learn more abou the Google Action Media Response interface](https://www.jovo.tech/docs/google-assistant/media-response).

The Google Assistant Media Response works a little different than Alexa's AudioPlayer, but there's a similarity. Just like Alexa you get into an AudioPlayer state (not the official name) after starting to stream the audio file using the Media Response interface.

Inside that state, Google takes over and handles stuff like pausing, resuming, skipping ahead X seconds, and more for you. Just like with Alexa, you will also be notified, in form of a request, after the audio stream finished, at which point you can start streaming the next one. 

These requests will also be mapped to a built-in intent inside the `AUDIOPLAYER` state:

```javascript
// src/app.js

AUDIOPLAYER: {

    // Other Alexa intents

    'GoogleAction.Finished'() {
  
    }
}
```

Inside that state, we play the next file using the same command introduced earlier in step one:

```javascript
// src/app.js

AUDIOPLAYER: {

    // Other Alexa intents

    'GoogleAction.Finished': function() {
        const secondEpisode = 'https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55';

        this.$googleAction.$mediaResponse.play(secondEpisode, 'Episode 55');
        this.tell('Enjoy');
    }
}
```

There are two more things to do before you can test the Google Action implementation:

* Add `this.tell()` in `this.ask()` for the Google Action (not the Alexa Skill).
* Add Suggestion Chips.

To be able to receive these notifications, you have to signal that you want to stream another file. For this, you have to use `this.ask()` instead of `this.tell()`. However, it is important to not use the `ask()` method if the incoming request is from an Alexa device. While Google uses the ask as a signal to receive the notification, Alexa handles it completely differently. It treats like every other speech output and waits for the user's response, which delays the actual audio stream until after the reprompt. To fix that, we can move the `ask()` statement inside the correct if-block, and add `tell()` to the one for Alexa.

Suggestion Chips are small pressable buttons, which guide the user to possible responses. 

![Google Action Suggestion Chips](./img/suggestion-chips-actions-on-google.jpg)

> Tutorial: [Implementing Google Assistant Suggestion Chips with Jovo]](https://www.jovo.tech/tutorials/google-assistant-suggestion-chips).

We will make these changes both to the `LAUNCH` as well as the `GoogleAction.Finished` intent:

```javascript
// src/app.js

LAUNCH() {
    const episode = 'https://traffic.libsyn.com/voicebot/Jan_Konig_on_the_Jovo_Open_Source_Framework_for_Voice_App_Development_-_Voicebot_Podcast_Ep_56.mp3';

    if (this.isAlexaSkill()) {
        this.$alexaSkill.$audioPlayer
            .setOffsetInMilliseconds(0)
            .play(episode, 'token')
            .tell('Enjoy');
    } else if (this.isGoogleAction()) {
        this.$googleAction.$mediaResponse.play(episode, 'Episode 56');
        this.$googleAction.showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
},

AUDIOPLAYER: {

    // Other Alexa intents

    'GoogleAction.Finished'() {
        const secondEpisode = 'https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55';

        this.$googleAction.$mediaResponse.play(secondEpisode, 'Episode 55');
        this.$googleAction.showSuggestionChips(['pause', 'start over']);
        this.tell('Enjoy');
    }
}
```

Alright, time to test. You probably already know how it works.

Before we move on to the next step, here's our handler's current state:

```javascript
// src/app.js

app.setHandler({
    LAUNCH() {
        const episode = 'https://traffic.libsyn.com/voicebot/Jan_Konig_on_the_Jovo_Open_Source_Framework_for_Voice_App_Development_-_Voicebot_Podcast_Ep_56.mp3';

        if (this.isAlexaSkill()) {
            this.$alexaSkill.$audioPlayer
                .setOffsetInMilliseconds(0)
                .play(episode, 'token')
                .tell('Enjoy');
        } else if (this.isGoogleAction()) {
            this.$googleAction.$mediaResponse.play(episode, 'Episode 56');
            this.$googleAction.showSuggestionChips(['pause', 'start over']);
            this.ask('Enjoy');
        }
    },

    AUDIOPLAYER: {
        'AlexaSkill.PlaybackStarted'() {
            
        },

        'AlexaSkill.PlaybackNearlyFinished'() {
            const secondEpisode = 'https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55';

            this.$alexaSkill.$audioPlayer
                .setExpectedPreviousToken('token')
                .enqueue(secondEpisode, 'token2');
        },

        'AlexaSkill.PlaybackFinished'() {
            
        },

        'AlexaSkill.PlaybackStopped'() {
            
        },

        'AlexaSkill.PlaybackFailed'() {
            
        },

        'GoogleAction.Finished'() {
            const secondEpisode = 'https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55';

            this.$googleAction.$mediaResponse.play(secondEpisode, 'Episode 55');
            this.$googleAction.showSuggestionChips(['pause', 'start over']);
            this.ask('Enjoy');
        }
    },
});
```

## Next Step

In the next step, we will prepare our development environment to test our app on an Amazon Alexa or Google Assistant device from then on.

> [Step 3: Preparing the Development Environment](./step-3-development-environment.md)

<!--[metadata]: { "description": "Learn more about the Alexa AudioPlayer and the Google Media Response interface to play multiple audio files in a row.", "author": "kaan-kilic", "og-image": "https://www.jovo.tech/img/courses/project-3-podcast-player/podcast-player-course.jpg" }-->
