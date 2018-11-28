# Step 2: Stream Multiple Files in a Row

Alright, it's time for the more interesting part. Before we can stream multiple files in a row we need at least some understanding how the respective audio players work.

* [The Alexa AudioPlayer](#the-alexa-audioplayer)
    * [AudioPlayer Requests](#audioplayer-requests)
    * [Expected Previous Token](#expected-previous-token)
    * [PlaybackNearlyFinished](#playbacknearlyfinished)
* [The Google Assistant Media Response](#the-google-assistant-media-response)
* [Next Step](#next-step)
  
## The Alexa AudioPlayer

The Alexa AudioPlayer interface is needed to stream long-form audio with your skill, by sending out *directives* to start and stop the audio stream as well as notifying us about changes to the audio playback's state.

The only *directive* we currently need is the play directive, which we already used in the previous step to stream the audio file. After sending that directive the current session ends and our skill will get registered as the most recent entity to play long-form audio. As a result, the user is now able to trigger the Alexa AudioPlayer's built-in intents, which offer the user control capabilities, for example, an intent to pause or resume the audio stream, without using our skill's invocation name. We will handle these intents in a later step.

### AudioPlayer Requests

Currently far more interesting are the AudioPlayer requests, that our skill will receive while streaming the audio file. These requests are used to notify us about changes to the playback state:

* **PlaybackStarted**: sent after starting to stream the audio file
* **PlaybackFinished**: sent after the stream finished
* **PlaybackStopped**: sent if the user manually pauses the audio stream
* **PlaybackFailed**: sent after Alexa encounters an error while attempting to play the audio file.
* **PlaybackNearlyFinished**: sent when the current audio file is almost finished
  
Our skill will receive at least one of the requests no matter what, which means we have to be able to handle the response (which can be just simply ending the session), otherwise, we will encounter an error, which has already happened to us in step one.

To make handling these requests easier for us, Jovo maps them to built-in intents inside the **AUDIOPLAYER** state.

```javascript  
// app/app.js
'AUDIOPLAYER': {
   'AudioPlayer.PlaybackStarted': function() {

   },
   'AudioPlayer.PlaybackNearlyFinished': function() {

   },
   'AudioPlayer.PlaybackFinished': function() {

   },
   'AudioPlayer.PlaybackStopped': function() {

   },
   'AudioPlayer.PlaybackFailed': function() {

   }
},
```

For us, the **PlaybackNearlyFinished** request is the more important one. It is used to enqueue the next audio stream, which will start to play as soon as the current audio stream finishes. Besides the audio files URL and a token, we will need the current stream's token as well, to prevent the following kind of situation:

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

Since our goal in this step is to play another file after the first one finished, the most interesting one of these requests is the **PlaybackNearlyFinished** one. That request is used to notify us that it's time to enqueue the next song. We set the expected token and enqueue the file using its URL and a new token:

```javascript
// app/app.js
'AudioPlayer.PlaybackNearlyFinished': function() {
    const secondSong = 'PLACEHOLDER';
    this.alexaSkill().audioPlayer().setExpectedPreviousToken('token').enqueue(secondSong, 'token');
},
```

Your handler should currently look like this:

```javascript
// app/app.js
app.setHandler({
    'LAUNCH': function () {
        const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
        if (this.isAlexaSkill()) {
            this.alexaSkill().audioPlayer().setOffsetInMilliseconds(0).play(song, 'token');
        } else if (this.isGoogleAction()) {
            this.googleAction().audioPlayer().play(song, 'song one');
        }
        this.tell('Enjoy');
    },
    'AUDIOPLAYER': {
        'AudioPlayer.PlaybackStarted': function () {

        },
        'AudioPlayer.PlaybackNearlyFinished': function () {
            const secondSong = 'PLACEHOLDER';
            this.alexaSkill().audioPlayer().setExpectedPreviousToken('token').enqueue(secondSong, 'token');
        },
        'AudioPlayer.PlaybackFinished': function () {

        },
        'AudioPlayer.PlaybackStopped': function () {

        },
        'AudioPlayer.PlaybackFailed': function () {

        }
    },
});
```

Save the file and run the Jovo Webhook. It's time to test. Open up the debugger and press the **LAUNCH** button, but don't forget to change your device back to an Alexa Echo.

Before we go on and play the second file on Google as well, let's fix the errors we encountered in step one by simply ending the session if one the other AudioPlayer requests come in, since we're not required to return a response:

```javascript
// app/app.js
'AUDIOPLAYER': {
   'AudioPlayer.PlaybackStarted': function() {
        this.endSession();
   },
   'AudioPlayer.PlaybackNearlyFinished': function() {
        const secondSong = 'PLACEHOLDER';
        this.alexaSkill().audioPlayer().setExpectedPreviousToken('token').enqueue(secondSong, 'token');
   },
   'AudioPlayer.PlaybackFinished': function() {
        this.endSession();
   },
   'AudioPlayer.PlaybackStopped': function() {
        this.endSession();
   },
   'AudioPlayer.PlaybackFailed': function() {
        this.endSession();
   }
},
```
  
## The Google Assistant Media Response

The Google Assistant Media Response works a little different than Alexa's AudioPlayer, but there's a similarity. Just like Alexa you get into an AudioPlayer state (not the official name) after starting to stream the audio file using the Media Response interface.

Inside that state, Google takes over and handles stuff like pausing, resuming, skipping ahead X seconds and more for you. Just like with Alexa, you will also be notified, in form of a request, after the audio stream finished, at which point you can simply start streaming the next one. These requests will also be mapped to a built-in intent inside the **AUDIOPLAYER** state:

```javascript
// app/app.js
'AUDIOPLAYER': {
    // Other Alexa intents
    'GoogleAction.Finished': function() {
  
    }
}
```

Inside that state, we simply play the next file using the same command introduced earlier in step one:

```javascript
// app/app.js
'AUDIOPLAYER': {
    // Other Alexa intents
    'GoogleAction.Finished': function() {
        const song = 'PLACEHOLDER';
        this.googleAction().audioPlayer().play(song, 'song one');
        this.tell('Enjoy');
    }
}
```

There is one more thing to do before you can test the Google Action implementation. To be able to receive these notifications you have to signal that you want to stream another file using `this.ask()` instead of `this.tell()` as well as add suggestion chips to your response. Suggestion Chips are simply small pressable buttons, which guide the user to possible responses. 

![Google Action Suggestion Chips](./img/suggestion-chips-actions-on-google.jpg)

You can find a tutorial about them [here](https://www.jovo.tech/tutorials/google-assistant-suggestion-chips).

We will make these changes both to the `LAUNCH` as well as the `GoogleAction.Finished` intent:

```javascript
// app/app.js
'LAUNCH': function () {
    const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
    if (this.isAlexaSkill()) {
        this.alexaSkill().audioPlayer().setOffsetInMilliseconds(0).play(song, 'token');
    } else if (this.isGoogleAction()) {
        this.googleAction().audioPlayer().play(song, 'song one');
        this.googleAction().showSuggestionChips(['pause', 'start over']);
    }
    this.ask('Enjoy');
},
'AUDIOPLAYER': {
    // Other Alexa intents
    'GoogleAction.Finished': function() {
        const song = 'PLACEHOLDER';
        this.googleAction().audioPlayer().play(song, 'song one');
        this.googleAction().showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
}
```

In this case, it is important to not use the `ask()` method if the incoming request is from an Alexa device. While Google uses the ask as a signal to receive the notification, Alexa handles it completely differently. It treats like every other speech output and waits for the user's response, which delays the actual audio stream until after the reprompt. To fix that we simply move the `ask()` statement inside the correct if-block.

```javascript
// app/app.js
'LAUNCH': function () {
    const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
    if (this.isAlexaSkill()) {
        this.alexaSkill().audioPlayer().setOffsetInMilliseconds(0).play(song, 'token');
    } else if (this.isGoogleAction()) {
        this.googleAction().audioPlayer().play(song, 'song one');
        this.googleAction().showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
},
```

Alright, time to test. You probably already know how it works.

If by any chance you forgot to change the device, you might have encountered a bug. The application does not respond to the request coming from Alexa anymore. Why? The reason is pretty straightforward, we never send out a response. In the prior version, the response was sent with the `this.tell()` statement, which is not there anymore, instead we only have the AudioPlayer statement. In theory that would work out. Alexa does not force you to add any kind of speech output to your *play directive*, but you still have to somehow send out the response, which the *directive* was added to.

Fixing it is pretty easy. You add `this.endSession()` to signal the Jovo Framework that it's time to send out the response:

```javascript
// app/app.js
if (this.isAlexaSkill()) {
    this.alexaSkill().audioPlayer().setOffsetInMilliseconds(0).play(song, 'token');
    this.endSession();
}
```

Test it out again to make sure everything is working now.

Before we move on to the next step, here's our handlers current state:

```javascript
// app/app.js
app.setHandler({
    'LAUNCH': function () {
        const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
        if (this.isAlexaSkill()) {
            this.alexaSkill().audioPlayer().setOffsetInMilliseconds(0).play(song, 'token');
            this.endSession();
        } else if (this.isGoogleAction()) {
            this.googleAction().audioPlayer().play(song, 'song one');
            this.googleAction().showSuggestionChips(['pause', 'start over']);
            this.ask('Enjoy');
        }
    },
    'AUDIOPLAYER': {
        'AudioPlayer.PlaybackStarted': function () {
            this.endSession();
        },
        'AudioPlayer.PlaybackNearlyFinished': function () {
            const secondSong = 'PLACEHOLDER';
            this.alexaSkill().audioPlayer().setExpectedPreviousToken('token').enqueue(secondSong, 'token');
        },
        'AudioPlayer.PlaybackFinished': function () {
            this.endSession();
        },
        'AudioPlayer.PlaybackStopped': function () {
            this.endSession();
        },
        'AudioPlayer.PlaybackFailed': function () {
            this.endSession();
        },
        'GoogleAction.Finished': function() {
            const secondSong = 'PLACEHOLDER';
            this.googleAction().audioPlayer().play(secondSong, 'song one');
            this.googleAction().showSuggestionChips(['pause', 'start over']);
            this.ask('Enjoy');
        }
    },
});
```

## Next Step

In the next step, we will prepare our development environment to test our app on an Amazon Alexa or Google Assistant device from then on.

> [Step 3: Preparing the Development Environment](./step-3.md)

<!--[metadata]: { "description": "In this lecture, you learn more about the Alexa AudioPlayer and the Google Media Response interface to play multiple audio files in a row", "author": "kaan-kilic" }-->