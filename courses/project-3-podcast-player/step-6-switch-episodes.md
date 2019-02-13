# Step 6: Switching between Episodes

It's time to add the last bit of controls to our Alexa Skill and Google Action podcast player that the user needs. They can already pause and resume the audio stream and now we will allow them to manually switch to the next or previous episode.

* [Updating the Jovo Language Model](#updating-the-jovo-language-model)
   * [Adding Custom Intents](#adding-custom-intents)
* [Updating the Logic](#updating-the-logic)
* [Next Step](#next-step)


## Updating the Jovo Language Model

We mentioned earlier, the Jovo Language Model allows us to maintain only a single file, which can be used to create the platform language models with the help of the Jovo CLI.

Currently our model looks like this:

```javascript
// model/en-US.json

{
	"invocation": "my podcast player",
	"intents": [
		{
			"name": "HelloWorldIntent",
			"phrases": [
				"hello",
				"say hello",
				"say hello world"
			]
		},
		{
			"name": "MyNameIsIntent",
			"phrases": [
				"{name}",
				"my name is {name}",
				"i am {name}",
				"you can call me {name}"
			],
			"inputs": [
				{
					"name": "name",
					"type": {
						"alexa": "AMAZON.US_FIRST_NAME",
						"dialogflow": "@sys.given-name"
					}
				}
			]
		}
	],
	"alexa": {
		"interactionModel": {
			"languageModel": {
				"intents": [
					{
						"name": "AMAZON.CancelIntent",
						"samples": []
					},
					{
						"name": "AMAZON.HelpIntent",
						"samples": []
					},
					{
						"name": "AMAZON.StopIntent",
						"samples": []
					},
					{
						"name": "AMAZON.PauseIntent",
						"samples": []
					},
					{
						"name": "AMAZON.ResumeIntent",
						"samples": []
					}
				]
			}
		}
	},
	"dialogflow": {
		"intents": [
			{
				"name": "Default Fallback Intent",
				"auto": true,
				"webhookUsed": true,
				"fallbackIntent": true
			},
			{
				"name": "Default Welcome Intent",
				"auto": true,
				"webhookUsed": true,
				"events": [
					{
						"name": "WELCOME"
					}
				]
			}
		]
	}
}
```


### Adding Custom Intents

> Docs: [Jovo Language Model - Intents](https://www.jovo.tech/docs/model#intents).

Every custom intent in the Jovo language Model has to have two properties, a name as well as sample phrases:

```javascript
{
    "name": "HelloWorldIntent",
    "phrases": [
        "hello",
        "say hello",
        "say hello world"
    ]
},
```

Our own intents, which we will call Next- and PreviousIntent, will follow the same format, but with a small difference. If you remember, Amazon provides built-in intents for the AudioPlayer, which the user can invoke without us even including them in our language model or using our skill's invocation name. `AMAZON.NextIntent` and `AMAZON.PreviousIntent` also belong to this list and they "override" our custom intents. If custom intents share the same utterances as a built-in intent, in most, if not all, cases the built-in intent will be invoked. So in our case, if the Alexa user says *Alexa, next episode*, the request will include the `AMAZON.NextIntent` rather than our own `NextIntent` and we will encounter an error.

For these cases, we can specify inside our Jovo Language Model, that the intent we're creating is already implemented as a built-in intent in Alexa, in which case the Jovo CLI will use the built-in intent while building the platform files, e.g. `AMAZON.NextIntent` will be used for Alexa and `NextIntent` for Google Action:

```javascript
// model/en-US.json

{
    "name": "NextIntent",
    "alexa": {
        "name": "AMAZON.NextIntent"
    },
    "phrases": [
		"skip",
		"next",
		"next one",
		"skip please",
		"next one please",
		"skip the episode",
		"skip to next episode",
		"skip to the next episode",
		"next episode",
		"would like the next episode",
		"I would like the next episode",
		"I would like to listen to the next episode"
    ]
},
{
    "name": "PreviousIntent",
    "alexa": {
        "name": "AMAZON.PreviousIntent"
    },
    "phrases": [
		"go back",
		"skip back",
		"back",
		"back up",
		"previous episode",
		"go to the previous episode",
		"skip back to previous episode",
		"go back to the previous episode",
		"the previous episode",
		"I'd like to listen to the previous episode",
		"I would like to listen to the previous episode",
		"go back please",
		"skip back please",
		"previous episode please",
		"skip back to the previous episode please"
    ]
},
```

> Tutorial: [Turn an Alexa Interaction Model into a Dialogflow Agent](https://www.jovo.tech/tutorials/alexa-model-to-dialogflow).

Some of these built-in intents are also [expendable](https://developer.amazon.com/docs/custom-skills/implement-the-built-in-intents.html#extending), which is why Jovo will use the phrases we specified to extend the built-in intent by default. But, that only works with some of the intents. In our case is does not, so we specify that we don't want to extend the intent by adding an empty `samples` array:

```javascript
// model/en-US.json

{
    "name": "NextIntent",
    "alexa": {
        "name": "AMAZON.NextIntent",
        "samples": []
    },
    "phrases": [
		"skip",
		"next",
		"next one",
		"skip please",
		"next one please",
		"skip the episode",
		"skip to next episode",
		"skip to the next episode",
		"next episode",
		"would like the next episode",
		"I would like the next episode",
		"I would like to listen to the next episode"
    ]
},
{
    "name": "PreviousIntent",
    "alexa": {
        "name": "AMAZON.PreviousIntent",
        "samples": []
    },
    "phrases": [
		"go back",
		"skip back",
		"back",
		"back up",
		"previous episode",
		"go to the previous episode",
		"skip back to previous episode",
		"go back to the previous episode",
		"the previous episode",
		"I'd like to listen to the previous episode",
		"I would like to listen to the previous episode",
		"go back please",
		"skip back please",
		"previous episode please",
		"skip back to the previous episode please"
    ]
}
```

Our language model should look like this now:

```javascript
// model/en-US.json

{
	"invocation": "my podcast player",
	"intents": [
		{
			"name": "HelloWorldIntent",
			"phrases": [
				"hello",
				"say hello",
				"say hello world"
			]
		},
		{
			"name": "MyNameIsIntent",
			"phrases": [
				"{name}",
				"my name is {name}",
				"i am {name}",
				"you can call me {name}"
			],
			"inputs": [
				{
					"name": "name",
					"type": {
						"alexa": "AMAZON.US_FIRST_NAME",
						"dialogflow": "@sys.given-name"
					}
				}
			]
		},
		{
			"name": "NextIntent",
			"alexa": {
				"name": "AMAZON.NextIntent",
				"samples": []
			},
			"phrases": [
				"skip",
				"next",
				"next one",
				"skip please",
				"next one please",
				"skip the episode",
				"skip to next episode",
				"skip to the next episode",
				"next episode",
				"would like the next episode",
				"I would like the next episode",
				"I would like to listen to the next episode"
			]
		},
		{
			"name": "PreviousIntent",
			"alexa": {
				"name": "AMAZON.PreviousIntent",
				"samples": []
			},
			"phrases": [
				"go back",
				"skip back",
				"back",
				"back up",
				"previous episode",
				"go to the previous episode",
				"skip back to previous episode",
				"go back to the previous episode",
				"the previous episode",
				"I'd like to listen to the previous episode",
				"I would like to listen to the previous episode",
				"go back please",
				"skip back please",
				"previous episode please",
				"skip back to the previous episode please"
			]
		}
	],
	"alexa": {
		"interactionModel": {
			"languageModel": {
				"intents": [
					{
						"name": "AMAZON.CancelIntent",
						"samples": []
					},
					{
						"name": "AMAZON.HelpIntent",
						"samples": []
					},
					{
						"name": "AMAZON.StopIntent",
						"samples": []
                    },
					{
						"name": "AMAZON.PauseIntent",
						"samples": []
					},
					{
						"name": "AMAZON.ResumeIntent",
						"samples": []
					}
				]
			}
		}
	},
	"dialogflow": {
		"intents": [
			{
				"name": "Default Fallback Intent",
				"auto": true,
				"webhookUsed": true,
				"fallbackIntent": true
			},
			{
				"name": "Default Welcome Intent",
				"auto": true,
				"webhookUsed": true,
				"events": [
					{
						"name": "WELCOME"
					}
				]
			}
		]
	}
}
```

That's all we need for now. Now we run the following commands o create and deploy the respective platform's files:

```sh
# Build platform files
$ jovo build

# Deploy to platforms
$ jovo deploy

# Shortcut
$ jovo build --deploy
```

## Updating the Logic

Before we add the NextIntent and PreviousIntent to our handler, we have to first make some adjustments to the configuration of our project. Technically, our Jovo Language Model contains four different intents: `NextIntent`, `PreviousIntent`, `AMAZON.NextIntent` and `AMAZON.PreviousIntent`.

If one of our Alexa users says *Alexa, next* or *Alexa, previous one* it will invoke the `AMAZON.NextIntent` or `AMAZON.PreviousIntent`, while Google Assistant users trigger the `NextIntent` or `PreviousIntent`, which means our handler has to have all four intents:

```javascript
'AMAZON.NextIntent'() {

},

'AMAZON.PreviousIntent'() {

},

NextIntent() {

},

PreviousIntent() {

},
```

That's redundant code. To help with that Jovo offers a simple mapping function for intents. We can specify which intents should be mapped, or routed, to another intent automatically, using the `intentMap` inside the `config.js` file:

```javascript
// src/config.js

module.exports = {

	// Other configurations

    intentMap: {
        'AMAZON.StopIntent': 'END',
		'AMAZON.NextIntent': 'NextIntent',
		'AMAZON.PreviousIntent': 'PreviousIntent'
    }
};
```

> Docs: [Jovo intentMap](https://www.jovo.tech/docs/routing/intents#intentmap).

This way we have to only have a single `NextIntent` and `PreviousIntent` inside our handler.

The actual logic of both intents is fairly easy. We get the current episode's index from the database, get the next/previous episode, save the new index and send out a *play directive*:

```javascript
// src/app.js

NextIntent() {
	let currentIndex = this.$user.$data.currentIndex;
	let nextEpisode = Player.getNextEpisode(currentIndex);
	currentIndex = Player.getEpisodeIndex(nextEpisode);
	this.$user.$data.currentIndex = currentIndex;
	if (this.isAlexaSkill()) {
		this.$alexaSkill.$audioPlayer
			.setOffsetInMilliseconds(0)
			.play(nextEpisode.url, `${currentIndex}`);
		
	} else if (this.isGoogleAction()) {
		this.$googleAction.$mediaResponse.play(nextEpisode.url, nextEpisode.title);
		this.$googleAction.showSuggestionChips(['pause', 'start over']);
		this.ask('Enjoy');
	}
},

PreviousIntent() {
	let currentIndex = this.$user.$data.currentIndex;
	let previousEpisode = Player.getPreviousEpisode(currentIndex);
	currentIndex = Player.getEpisodeIndex(previousEpisode);
	this.$user.$data.currentIndex = currentIndex;
	if (this.isAlexaSkill()) {
		this.$alexaSkill.$audioPlayer
			.setOffsetInMilliseconds(0)
			.play(previousEpisode.url, `${currentIndex}`);
		
	} else if (this.isGoogleAction()) {
		this.$googleAction.$mediaResponse.play(previousEpisode.url, previousEpisode.title);
		this.$googleAction.showSuggestionChips(['pause', 'start over']);
		this.ask('Enjoy');
	}
},
```

While we are at it, we should check for a corner case that might cause an error. What if, for example, the user is currently at the last track and they want to skip ahead to next one? We have to inform them that it's not possible.

Implementing that is easy. Since Javascript will return `undefined` if we try to access an array at an index without a value, our `getPreviousEpisode()` and `getNextEpisode()` methods will also return `undefined` if we are at the first/last episode. So simply before we try to send out a *play directive* we check for that and respond accordingly:

```javascript
// src/app.js

NextIntent() {
	let currentIndex = this.$user.$data.currentIndex;
	let nextEpisode = Player.getNextEpisode(currentIndex);
	if (!nextEpisode) {
		return this.tell('That was the most recent episode. You have to wait until a new episode gets released.');
	}
	currentIndex = Player.getEpisodeIndex(nextEpisode);
	this.$user.$data.currentIndex = currentIndex;
	if (this.isAlexaSkill()) {
		this.$alexaSkill.$audioPlayer
			.setOffsetInMilliseconds(0)
			.play(nextEpisode.url, `${currentIndex}`);
		
	} else if (this.isGoogleAction()) {
		this.$googleAction.$mediaResponse.play(nextEpisode.url, nextEpisode.title);
		this.$googleAction.showSuggestionChips(['pause', 'start over']);
		this.ask('Enjoy');
	}
},
PreviousIntent() {
	let currentIndex = this.$user.$data.currentIndex;
	let previousEpisode = Player.getPreviousEpisode(currentIndex);
	if (!previousEpisode) {
		return this.tell('You are already at the oldest episode.');
	}
	currentIndex = Player.getEpisodeIndex(previousEpisode);
	this.$user.$data.currentIndex = currentIndex;
	if (this.isAlexaSkill()) {
		this.$alexaSkill.$audioPlayer
			.setOffsetInMilliseconds(0)
			.play(previousEpisode.url, `${currentIndex}`);
		
	} else if (this.isGoogleAction()) {
		this.$googleAction.$mediaResponse.play(previousEpisode.url, previousEpisode.title);
		this.$googleAction.showSuggestionChips(['pause', 'start over']);
		this.ask('Enjoy');
	}
},
```

In this case a `return` statement is used to stop the execution of the remaining code.

Before you can test everything out, don't forget to remove the `AMAZON.NextIntent` and `AMAZON.PreviousIntent`, which we added in step four.

## Next Step

In the next step, we will rework the user interaction at the app's launch!

> [Step 7: Reworking the User Interaction at Launch](./step-7-launch-user-interaction.md)

<!--[metadata]: { "description": "Learn how to use a NextIntent and a PreviousIntent to let our user switch between episodes in a Podcast Player Alexa Skill and Google Action.", "author": "kaan-kilic", "og-image": "https://www.jovo.tech/img/courses/project-3-podcast-player/podcast-player-course.jpg" }-->