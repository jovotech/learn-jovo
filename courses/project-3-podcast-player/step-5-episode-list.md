# Step 5: Storing and Retrieving Multiple Episodes

In this step, we will build a system to store and retrieve the episodes of our podcast on Amazon Alexa and Google Assistant.

* [Storing the Episodes](#storing-the-episodes)
* [Retrieving the Episodes](#retrieving-the-episodes)
* [Updating our app.js](#updating-our-appjs)
    * [Enqueue](#enqueue)
    * [Resume](#resume)
* [Next Step](#next-step)

## Storing the Episodes

Until now we've simply hard coded the episodes URLs. Obviously, that's wrong, but it saved us some time while we were figuring out both audio interfaces for Alexa and Google Assistant.

Instead, we will now store everything inside a `JSON` file, called `episodes.json` inside our `src` folder, with the following structure (thanks again to [Bret Kinsella](https://twitter.com/bretkinsella) for providing us with the [Voicebot Podcast](https://voicebot.ai/voicebot-podcasts/) episodes):

```javascript
// src/episodes.json

[
    {
        "title": "Jan König Discusses the Jovo Open Source Framework for Voice App Development",
        "url": "https://traffic.libsyn.com/voicebot/Jan_Konig_on_the_Jovo_Open_Source_Framework_for_Voice_App_Development_-_Voicebot_Podcast_Ep_56.mp3"
    },
    {
        "title": "John Kelvie CEO of Bespoken Discusses How Voice App Testing is Different",
        "url": "https://traffic.libsyn.com/voicebot/John_Kelvie_CEO_of_Bespoken_Talks_Voice_App_Testing_-_Voicebot_Podcast_Ep_55"
    },
    {
        "title": "Shane Mac CEO of Assist Says The Talking Internet is Here",
        "url": "https://traffic.libsyn.com/voicebot/Shane_Mac_CEO_of_Assist_The_Talking_Internet_is_Here_-_Voicebot_Podcast_54.mp3"
    },
    {
        "title": "Tom Hebner Global Innovation Head for Nuance Talks 20 Years in Voice",
        "url": "https://traffic.libsyn.com/voicebot/Tom_Hebner_Global_Innovation_Lead_for_Nuance_Talks_20_Years_in_Voice_-_Voicebot_Podcast_Ep_53.mp3"
    },
    {
        "title": "Pulse Labs Founders Talk Voice App Testing and Alexa Accelerator",
        "url": "https://traffic.libsyn.com/voicebot/Pulse_Labs_Co-founders_Talk_Voice_App_Testing_-_Voicebot_Podcast_Ep_52.mp3"
    }
]
```

This file is an array containing multiple objects, where the very last object, i.e. the index `array.length - 1` (in this case index `4` for five episodes) will contain the very first episode and the very first object, i.e index `0`, will store the most recent episode.

Each object will contain the title and URL of the episode.

Each episode's index can also be used as the unique value needed for the `token` we send out with every Alexa *play directive*. So from now on we will replace every `token` with the index as well as use the episode's title with every Google Action *play directive*.


## Retrieving the Episodes

To retrieve and work with the `JSON` file, we will create an interface called `player.js` inside our `app` folder.

The interface will allow us to access the latest, first, next and previous episode, which we all need for the finished podcast player. The latest one is needed for people who want to directly start with that or have already listened to every other episode. The first one is for people who want to start from the beginning. The next and previous one, are needed if the user manually changes the episode, as well as when they finished one and it's time to enqueue the next.

```javascript
// src/player.js

const episodesJSON = require('./episodes.json');

module.exports = {
    getLatestEpisode: function() {

    },
    getFirstEpisode: function() {

    },
    getNextEpisode: function() {

    },
    getPreviousEpisode: function() {

    }
}
```

For the first two, we simply import the `episodes.json` file and return the content at the correct index:

```javascript
// src/player.js

const episodesJSON = require('./episodes.json');

module.exports = {
    getLatestEpisode: function() {
        return episodesJSON[0];
    },
    getFirstEpisode: function() {
        return episodesJSON[episodesJSON.length - 1];
    },
}
```

The other two will take in an index as a parameter and return the episode at the next/previous index.

```javascript
// src/player.js

module.exports = {
    getNextEpisode: function(index) {
        return episodesJSON[index - 1];
    },
    getPreviousEpisode: function(index) {
        return episodesJSON[index + 1];
    }
}
```

Our `player.js` file should look like this now:

```javascript
// src/player.js

const episodesJSON = require('./episodes.json');

module.exports = {
    getLatestEpisode: function() {
        return episodesJSON[0];
    },

    getFirstEpisode: function() {
        return episodesJSON[episodesJSON.length - 1];
    },

    getNextEpisode: function(index) {
        return episodesJSON[index - 1];
    },

    getPreviousEpisode: function(index) {
        return episodesJSON[index + 1];
    }
}
```

Now we can use our interface to replace the hard-coded episode URLs.

## Updating our app.js

First of all, import the `player.js` file:

```javascript
// src/app.js

const Player = require('./player.js');
```

Now, the first thing to replace is the logic inside the `LAUNCH` intent. Instead of simply saving the episode URL inside the database, we will now save the index. For that, we have to add another simple function to our `player.js` file.

The `getEpisodeIndex` function will return us the correct index at which the episode is stored in our `episodes.json` array.

```javascript
// src/player.js

const episodesJSON = require('./episodes.json');

module.exports = {

    // Other functions

    getEpisodeIndex: function(episode) {
        return episodesJSON.indexOf(episode);
    }
}
```

If it's a new user, let's just play the first episode for now. We will improve the user experience later on.

```javascript
// src/app.js

LAUNCH() {
    let episode;
    if (this.$user.isNew()) {
        episode = Player.getFirstEpisode();
        let currentIndex = Player.getEpisodeIndex(episode);
        this.$user.$data.currentIndex = currentIndex;
    }
    // ...
},
```

Otherwise, we previously resumed playing the episode the user last listened to. Since we're now saving the index instead of the episode URL, we have to make some changes here as well as in the `player.js` file as well.

We have to add a function to the `player.js` interface, that takes the index as a parameter and returns us the episode object.

```javascript
// src/player.js

const episodesJSON = require('./episodes.json');

module.exports = {

    // Other functions

    getEpisode: function(index) {
        return episodesJSON[index];
    }
}
```

Now we use the function to play the episode the user last listened to:

```javascript
// src/app.js

LAUNCH() {
    let episode;
    let currentIndex;
    if (this.$user.isNew()) {
        episode = Player.getFirstEpisode();
        currentIndex = Player.getEpisodeIndex(episode);
        this.$user.$data.currentIndex = currentIndex;
    } else {
        currentIndex = this.$user.$data.currentIndex;
        episode = Player.getEpisode(currentIndex);
    }
    // ...
},
```

There is one more thing to fix before we can test it out. Since the episode variable is now an object instead of a string containing the url, we have to fix the reference from `.play(episode, token)` to `.play(episode.url, token)`:

```javascript
// src/app.js

LAUNCH() {
    // ...
    if (this.isAlexaSkill()) {
        this.$alexaSkill.$audioPlayer
            .setOffsetInMilliseconds(0)
            .play(episode.url, `${currentIndex}`);
        
    } else if (this.isGoogleAction()) {
        this.$googleAction.$mediaResponse.play(episode.url, episode.title);
        this.$googleAction.showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
}
```

Before we test out our new implementation, we have to delete the content of the `../db/db.json` file, since we made changes to the database structure. We simply delete everything and save the blank file.

Alright, now we can test it out. The audio file should start playing without any errors.

### Enqueue

There's still more to update besides the `LAUNCH` intent. It's time to fix the enqueue logic as well.

* [Enqueue on Alexa](#enqueue-on-alexa)
* [Enqueue on Google Assistant](#enqueue-on-google-assistant)


#### Enqueue on Alexa

Let's start with the Alexa part. In the `AlexaSkill.PlaybackNearlyFinished` intent we simply get the current index from the database and use the `getNextEpisode()` method from our `player.js` file. We also check if `episode` has a value since the player will reach the last episode at some point:

```javascript
// src/app.js

'AlexaSkill.PlaybackNearlyFinished'() {
    let currentIndex = this.$user.$data.currentIndex;
    let episode = Player.getNextEpisode(currentIndex);
    let nextIndex = Player.getEpisodeIndex(episode);
    if (episode) {
        this.$alexaSkill.$audioPlayer
            .setExpectedPreviousToken(`${currentIndex}`)
            .enqueue(episode.url, `${nextIndex}`);
    }
},
```

The `AlexaSkill.PlaybackFinished` intent will be used to decrease the index as long as it's bigger than `0`:

```javascript
// src/app.js

'AlexaSkill.PlaybackFinished'() {
    let currentIndex = this.$user.$data.currentIndex;
    if (currentIndex > 0) {
        this.$user.$data.currentIndex = currentIndex - 1;
    }
},
```

#### Enqueue on Google Assistant

For Google we do the same with the small difference that it will be all done inside a single intent:

```javascript
// src/app.js

'GoogleAction.Finished'() {
    let index = this.$user.$data.currentIndex;
    let episode = Player.getNextEpisode(index);
    if (episode) {
        this.$user.$data.currentIndex -= 1;
        this.$googleAction.$mediaResponse.play(episode.url, episode.title);
        this.$googleAction.showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
}
```

### Resume

We also have to fix the `AMAZON.ResumeIntent`:

```javascript
// src/app.js

'AMAZON.ResumeIntent'() {
    let offset = this.$user.$data.offset;
    let currentIndex = this.$user.$data.currentIndex;
    let episode = Player.getEpisode(currentIndex);

    this.$alexaSkill.$audioPlayer
        .setOffsetInMilliseconds(offset)
        .play(episode.url, `${currentIndex}`);
},
```

## Next Step

In the next step we will use the new system to allow our user to manually switch between episodes.

> [Step 6: Manually Switch between Episodes](./step-6-switch-episodes.md)

<!--[metadata]: { "description": "Learn how to store and retrieve podcast episodes in your Podcast Player voice app for Amazon Alexa and Google Assistant.", "author": "kaan-kilic", "og-image": "https://www.jovo.tech/img/courses/project-3-podcast-player/podcast-player-course.jpg" }-->