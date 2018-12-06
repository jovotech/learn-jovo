# Step 1: Streaming an Audio File

* [Create the Project](#create-the-project)
* [Play an audio file](#play-an-audio-file)
    * [Amazon Alexa](#amazon-alexa)
    * [Google Action](#google-action)
* [Next Step](#next-step)

## Create the Project

To start off we have to create a Jovo project. Therefore if you haven't already, install the [Jovo CLI](https://github.com/jovotech/jovo-cli):

```sh
$ npm i -g jovo-cli
```

Create a new project:

```text
$ jovo new <projectName>
```

## Play an audio file

The heart of our project is located in the `app.js` file inside the app folder. That's the place where we configure our project and define our handler.

Open up the file and we will see that there are already three intents defined in the `setHandler()` function. We delete everything besides the `LAUNCH` intent since we won't need them at this point.

Streaming an audio file works differently on both platforms, so we will go over the Alexa part first and add the Google Action implementation after that.

### Amazon Alexa

```javascript
const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(song, 'token');
this.tell('Enjoy');
```

First, we store the URL, where the audio file is hosted at (has to be an HTTPS endpoint). After that, we set the offset, which defines the timestamp at which Alexa will start playing the file. Let's say you have an audio file which is three minutes long and you set the offset to 10000 (you define the offset in milliseconds). Instead of starting from `0:00`, the file will start playing from `0:10`, i.e. you skip the first ten seconds of the song.

After that, we call the AudioPlayer interface's play function and pass in the URL as well as a token which will discuss later on.

We place that snippet inside your `LAUNCH` intent, so it gets triggered every time our app is launched:

```javascript
// src/app.js
LAUNCH() {
    const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
    this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(song, 'token');
    this.tell('Enjoy');
},
```

The most convenient way to test our application is to run the local Jovo Webhook using the CLI.

```sh
$ jovo run
```

The Webhook URL we receive can be used for local development as an HTTPS endpoint, so we don't have to upload our code to AWS Lambda or any other cloud service.

```text
Example server listening on port 3000!
This is your Webhook url: https://webhook.jovo.cloud/ed7c57e3-a03b-4806-9f8d-23had9213n
```

The Webhook URL is also the gateway to the Jovo Debugger. You simply copy and paste the URL into your browser:

![Jovo Debugger](./img/jovo-debugger.png)

The Jovo debugger improves your developing experience by showing you the most important data (incoming requests, responses, database, etc.) at one place as well as allowing you to test right on the spot.  

For now, press the launch button right at the middle of the page to send a sample request to your Webhook and your app should respond by playing the audio file. Ignore the errors, for now, we will discuss and fix these further down the road.

![Jovo Debugger playing audio](img/jovo-debugger-playing-audio.png)

That was easy, wasn’t it?

Let’s do the same for our Google Action.

### Google Action

We simply add `this.$googleAction.$audioPlayer.play(song, 'song one');`, where the first parameter is the audio files url and the second one the title, to our LAUNCH intent.

```javascript
// src/app.js
const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(song, 'token');
this.$googleAction.$audioPlayer.play(song, 'song one');
this.tell('Enjoy');
```

As you can see there’s a slight difference between the Alexa function and the Google one. We don’t have to specify the offset or a token, but a title. We will explain the reason for that later on.

But, there's still a small issue. We don't want to use both interfaces with every request, because that would cause an error, so we have to first check from which platform the request is being sent:

```javascript
// src/app.js
const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
if (this.isAlexaSkill()) {
    this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(song, 'token');
} else if (this.isGoogleAction()) {
    this.$googleAction.$audioPlayer.play(song, 'song one');
}
this.tell('Enjoy');
```

So our `LAUNCH` intent should look like this now:

```javascript
// src/app.js
LAUNCH() {
    const song = 'https://s3.amazonaws.com/jovo-songs/song1.mp3';
    if (this.isAlexaSkill()) {
        this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(song, 'token');
    } else if (this.isGoogleAction()) {
        this.$googleAction.$audioPlayer.play(song, 'song one');
    }
    this.tell('Enjoy');
},
```

Let’s test the Google Action implementation. First, we have to change the device that the Jovo Debugger simulates, which is set to the Alexa Echo Dot by default, to the Google Assistant Phone so our application receives the right requests. You can find the option on the bottom of the Debugger.

![Jovo Debugger Device](./img/jovo-debugger-device.png)

After that press the `LAUNCH` button and we should hear the song playing.

![Jovo Debugger Playing Audio Google](img/jovo-debugger-playing-audio-google.png)

Alright, we're now able to play a simple audio file on both Amazon Alexa and the Google Assistant. 

## Next Step

In the next step, we will learn how to keep streaming audio files, after the first song finished.

> [Step 2: Stream Multiple Files in a Row](./step-2.md)

<!--[metadata]: { "description": "In this lecture, you learn how to stream an audio file on Amazon Alexa and Google Action", "author": "kaan-kilic" }-->