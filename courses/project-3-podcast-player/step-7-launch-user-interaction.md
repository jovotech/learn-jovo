# Step 7: Reworking the User Interaction at Launch

We're almost done. Our podcast player can play multiple episodes in a row, allows our users to pause, resume an episode as well as skip ahead or back to another episode. All that on both platforms with a minimal amount of code.

In this step, we will improve the user interaction at the launch.

* [Updating the User Interaction](#user-interaction)
	* [Adding a FirstEpisodeIntent](#adding-a-firstepisodeintent)
	* [Adding a LatestEpisodeIntent](#adding-a-latestepisodeintent)
	* [Adding a ListIntent](#adding-a-listintent)
	* [Adding a ChooseFromListIntent](#adding-a-choosefromlistintent)
* [Updating the Launch Intent](#updating-the-launch-intent)
* [Updating the Resume Intent](#updating-the-resume-intent)
* [Next Step](#next-step)

## Updating the User Interaction

The idea for the user interaction is the following:

New users should get a list of four, in this case random, episodes to choose from beside the very first episode. Returning users will be asked if they want to continue where they left off or rather listen to the latest one.

What do we need for that?

First of all, we have to add an `FirstEpisodeIntent`. In addition to that, we need an intent, which lists the four random episodes, `ListIntent`, as well as one that gets triggered with by user's answer, `ChooseFromListIntent`. Last but not least, we need a `LatestEpisodeIntent` as well. That's it.

Let's do this step by step:

* [Adding a FirstEpisodeIntent](#adding-a-firstepisodeintent)
* [Adding a LatestEpisodeIntent](#adding-a-latestepisodeintent)
* [Adding a ListIntent](#adding-a-listintent)
* [Adding a ChooseFromListIntent](#adding-a-choosefromlistintent)

### Adding a FirstEpisodeIntent

Probably the easiest one. We first add the intent to our language model just like we did before:

```javascript
// model/en-US.json

{
    "name": "FirstEpisodeIntent",
    "phrases": [
        "episode one",
        "I want episode one",
        "play episode one",
        "start with episode one",
        "I want to listen to the first episode"
    ]
},
```

The intent itself will use the `player.js` file's `getFirstEpisode()` method, save the current index to our database and send out a *play directive*:

```javascript
// src/app.js

FirstEpisodeIntent() {
    let episode = Player.getFirstEpisode();
    let currentIndex = Player.getEpisodeIndex(episode);
    this.$user.$data.currentIndex = currentIndex;

	if (this.isAlexaSkill()) {
		this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(episode.url, `${currentIndex}`);
	} else if (this.isGoogleAction()) {
		this.$googleAction.$mediaResponse.play(episode.url, episode.title);
		this.$googleAction.showSuggestionChips(['pause', 'start over']);
		this.ask('Enjoy');
	}
},
```

### Adding a LatestEpisodeIntent

Works almost completely the same way as `FirstEpisodeIntent`:

```javascript
// model/en-US.json

{
    "name": "LatestEpisodeIntent",
    "phrases": [
        "latest episode",
        "new episode",
        "the latest episode",
        "the new episode",
        "I want the latest episode",
        "I want to listen to the latest episode",
        "I would prefer the latest one",
        "play the latest episode",
        "play the new episode",
        "I would like to listen to the latest episode",
        "stream the latest episode",
        "stream the new episode"
    ]
},
```

```javascript
// src/app.js

LatestEpisodeIntent() {
    let episode = Player.getLatestEpisode();
    let currentIndex = Player.getEpisodeIndex(episode);
    this.$user.$data.currentIndex = currentIndex;

	if (this.isAlexaSkill()) {
		this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(episode.url, `${currentIndex}`);
	} else if (this.isGoogleAction()) {
		this.$googleAction.$mediaResponse.play(episode.url, episode.title);
		this.$googleAction.showSuggestionChips(['pause', 'start over']);
		this.ask('Enjoy');
	}
},
```

### Adding a ListIntent

This is one will be a little trickier, but before we add the intent to our handler, let's add it to the language model first:

```javascript
// model/en-US.json

{
    "name": "ListIntent",
    "phrases": [
        "list episodes",
        "list the episodes",
        "I want the list",
        "I want the list of episodes",
        "episode list",
        "Can I get the list of episodes",
        "Can I have the episode list",
        "choose from a list",
        "choose from list",
        "I choose the list",
        "choose list",
        "list of episodes"
    ]
},
```

Here's a small example on how the interaction should look like in the end:

```text
Platform: Here's a list of episodes: 1. episode 66's title, 2. episode 04's title, 3. episode 23's title, 4. episode 17's title. Which one would you like to listen to?

User: The first one

Platform: *starts playing episode 66*
```

We first need to add a method to our `player.js` file, which returns us `n` random unique indices:

```javascript
// src/player.js

const episodesJSON = require('./episodes.json');

module.exports = {

    // Other functions

    getRandomIndices: function(number) {
        let arr = []
        while (arr.length < number){
            let randomNumber = Math.floor(Math.random() * episodesJSON.length);
            if (arr.indexOf(randomNumber) === -1) {
                arr.push(randomNumber);
            }
        }
        return arr;
    },
}
```

After that, we implement the intent

The intent will first get the random indices and save them in our session attributes. That's a way to store data, that we only need across a single session. Every time the conversation ends, the session attributes are reset so it's not a viable option to store data for longer periods of time. In our case, it works perfectly though.

```javascript
// src/app.js

ListIntent() {
    const indices = Player.getRandomIndices(4);
    this.$session.$data.episodeIndices = indices;
},
```

We will need the indices to determine which episode the user chose, but more on that in a minute.

Besides that, we have to somehow communicate the list to our user in a simple form. The feature best suited for that is the Jovo SpeechBuilder, which is a tool to help us build more complex speech responses easier, by allowing us to assemble the response element by element.

Here's an example:

```javascript
this.$speech
	.addText('Hey, you are the ')
	.addSayAsCardinal('1')
	.addText('visitor today.')
	.addBreak('200ms')
	.addText('How are you?');

this.tell(this.$speech);
```

```text
Hey, you are the first visitor today. *small break* How are you?
```

You can find the full list of features [here](https://www.jovo.tech/docs/output/speechbuilder 'docs/basic-concepts/output/speechbuilder').

In our case, it will look like this:

```javascript
// src/app.js

this.$speech.addText('Here\'s a list of episodes: ');
for (let i = 0; i < indices.length; i++) {
    let episode = Player.getEpisode(indices[i]);
    this.$speech.addSayAsOrdinal(`${i + 1}`).addText(episode.title).addBreak("100ms");
}
this.$speech.addText('Which one would you like to listen to?');
this.ask(this.$speech);
```

We add the pretext, use a for loop to get each episode using its index and add its title to our response and add the question at the end.

The complete intent looks like this:

```javascript
// src/app.js

ListIntent() {
    const indices = Player.getRandomIndices(4);
    this.$session.$data.episodeIndices = indices;

	this.$speech.addText('Here\'s a list of episodes: ');
	for (let i = 0; i < indices.length; i++) {
		let episode = Player.getEpisode(indices[i]);
		this.$speech.addSayAsOrdinal(`${i + 1}`).addText(episode.title).addBreak("100ms");
	}
	this.$speech.addText('Which one would you like to listen to?');
	this.ask(this.$speech);
},
```

### Adding a ChooseFromListIntent

This one is a little bit trickier than the previous one. The idea is to let the user answer with *first*, *second one*, etc., because it would be a little to demanding to expect them to use the episode's title.

For that, our intent will need an input type that recognizes ordinal numbers. With the Jovo Language Model we can specify separate input types for each platform. Since Google does provide a built-in one ordinal numbers, `@sys.ordinal`, we can use that, but for Alexa, we have to create a custom one. 

* [Defining a Custom Input Type](#defining-a-custom-input-type)
* [ChooseFromListIntent Logic](#choosefromlistintent-logic)

#### Defining a Custom Input Type

Every input type needs a `name`, which is used to reference it later on. Besides that it needs possible values and optional synonyms for each value:

```javascript
// model/en-US.json

"inputTypes": [
    {
        "name": "myCityInputType",
        "values": [
            {
                "value": "Berlin"
            },
            {
                "value": "New York",
                "synonyms": [
                    "New York City"
                ]
            }
        ]
    }
],
```

With every request, we will receive the input's name, it's key and value.

For example, if our user said *I am from New York City* and it triggered the `city` input, our request will contain the following information:

```text
name --> city
key --> New York
value --> New York City
```

Let's get to our implementation. The first thing we want to keep in mind is, that we stored our episode list inside an array, which we want to access using the index of the requested episode. Now, we have to somehow convert the utterance *first* to the integer `1`. We can probably manage to do that in our intent, but that's way easier is to handle that inside our language model.

We simply define our input types values as `1`, `2`, `3`, `4`, etc. and each values synonyms as for example `second`, `2nd`, `2.`, `two`. Now if our user says *second* we will also receive the input types key `2` in our request, which we can then use for our array.

Here's how that would look like:

```javascript
// model/en-US.json

"inputTypes": [
    {
        "name": "ordinal",
        "values": [
            {
                "value": "1",
                "synonyms": [
                    "first",
                    "1st",
                    "1.",
                    "one"
                ]
            },
            {
                "value": "2",
                "synonyms": [
                    "second",
                    "2nd",
                    "2.",
                    "two"
                ]
            },
            {
                "value": "3",
                "synonyms": [
                    "third",
                    "3rd",
                    "3.",
                    "three"
                ]
            },
            {
                "value": "4",
                "synonyms": [
                    "fourth",
                    "4th",
                    "4.",
                    "four"
                ]
            }
        ]
    }
],
```

We place the `inputTypes` array at the same level as the rest of the stuff in our language model:

```javascript
// model/en-US.json

{
	"invocation": "my podcast player",
	"intents": [
		// ...
	],
	"inputTypes": [
		// ...
	],
	"alexa": {
        // ...
	},
	"dialogflow": {
        // ...
	}
}
```

Now we can add the `ChooseFromListIntent` and reference our custom input type as well as the Google Action built-in one:

```javascript
// model/en-US.json

{
    "name": "ChooseFromListIntent",
    "phrases": [
        "{ordinal}",
        "{ordinal} one please",
        "{ordinal} one",
        "{ordinal} episode",
        "Play the {ordinal}",
        "Play the {ordinal} one",
        "Play the {ordinal} episode",
        "Play the {ordinal} one",
        "Play the {ordinal} episode",
        "I want the {ordinal} one",
        "I choose the {ordinal} one",
        "I want the {ordinal} episode",
        "I choose the {ordinal} episode",
        "stream the {ordinal} one"
    ],
    "inputs": [
        {
            "name": "ordinal",
            "type": {
                "alexa": "ordinal",
                "dialogflow": "@sys.ordinal"
            }
        }
    ]
},
```

Before we move on to add the intent to our handler, let's delete the `HelloWorldIntent` and `MyNameIsIntent` as well. We don't need them. The current state of our language model looks like this:

```javascript
// model/en-US.json

{
	"invocation": "my podcast player",
	"intents": [
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
		},
		{
			"name": "LatestEpisodeIntent",
			"phrases": [
				"latest episode",
				"new episode",
				"the latest episode",
				"the new episode",
				"I want the latest episode",
				"I want to listen to the latest episode",
				"I would prefer the latest one",
				"play the latest episode",
				"play the new episode",
				"I would like to listen to the latest episode",
				"stream the latest episode",
				"stream the new episode"
			]
		},
		{
			"name": "FirstEpisodeIntent",
			"phrases": [
				"episode one",
				"I want episode one",
				"play episode one",
				"start with episode one",
				"I want to listen to the first episode"
			]
		},
		{
			"name": "ChooseFromListIntent",
			"phrases": [
				"{ordinal}",
				"{ordinal} one please",
				"{ordinal} one",
				"{ordinal} episode",
				"Play the {ordinal}",
				"Play the {ordinal} one",
				"Play the {ordinal} episode",
				"Play the {ordinal} one",
				"Play the {ordinal} episode",
				"I want the {ordinal} one",
				"I choose the {ordinal} one",
				"I want the {ordinal} episode",
				"I choose the {ordinal} episode",
				"stream the {ordinal} one"
			],
			"inputs": [
				{
					"name": "ordinal",
					"type": {
						"alexa": "ordinal",
						"dialogflow": "@sys.ordinal"
					}
				}
			]
		}
	],
	"inputTypes": [
		{
			"name": "ordinal",
			"values": [
				{
					"value": "1",
					"synonyms": [
						"first",
						"1st",
						"1.",
						"one"
					]
				},
				{
					"value": "2",
					"synonyms": [
						"second",
						"2nd",
						"2.",
						"two"
					]
				},
				{
					"value": "3",
					"synonyms": [
						"third",
						"3rd",
						"3.",
						"three"
					]
				},
				{
					"value": "4",
					"synonyms": [
						"fourth",
						"4th",
						"4.",
						"four"
					]
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

#### ChooseFromListIntent Logic

We're done with the trickier part, as this one will be fairly easy again. We first get the indices array from our session attributes, use the input to get the correct index from it, save the index to our database, use it to get the episode data and last but not least send out a *play directive*.

```javascript
// src/app.js

ChooseFromListIntent() {
	const ordinal = this.$inputs.ordinal;
    let episodeIndices = this.$session.$data.episodeIndices;
    let episodeIndex = episodeIndices[parseInt(ordinal.key) - 1];
    this.$user.$data.currentIndex = episodeIndex;
    let episode = Player.getEpisode(episodeIndex);

    if (this.isAlexaSkill()) {
        this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(0).play(episode.url, `${episodeIndex}`);
        
    } else if (this.isGoogleAction()) {
        this.$googleAction.$mediaResponse.play(episode.url, episode.title);
        this.$googleAction.showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
},
```

## Updating the Launch Intent

Now we can use all of the intents to update the user interaction at app's launch. We want to have two different outputs depending if it is a new user or returning one. The easiest way to do that with Jovo is by using one of it's built-in intents called `NEW_USER`, which, as the name says, every new user gets routed to at launch.

So we use the `NEW_USER` intent to ask new users, if they want to choose an episode from a list or rather start with the very first one:

```javascript
// src/app.js

NEW_USER() {
    return this.ask('Would you like to begin listening from episode one or rather choose from a list?');
},
```

Depending on the user's choice, the `ListIntent` or `FirstEpisodeIntent` will be invoked.

At the `LAUNCH` intent we will handle the interaction with returning users. The current logic inside the `LAUNCH` intent is not needed anymore so we have to delete that and replace it with a question similar to the `NEW_USER` intent's one:

```javascript
// src/app.js

LAUNCH() {
    return this.ask('Would you like to resume where you left off or listen to the latest episode?');
},
```

## Updating the Resume Intent

With the updated `LAUNCH` intent, we need to add a `ResumeIntent` for our Google Action as well. Since we can't specify the offset, we will simply start playing the correct episode.

We add the intent to our Jovo Language Model and specify that we still use Amazon's built-in intent for Alexa. We also delete the `AMAZON.ResumeIntent` inside the `alexa` object of our language model.

```javascript
// model/en-US.json

{
    "name": "ResumeIntent",
    "alexa": {
        "name": "AMAZON.ResumeIntent",
        "samples": [],
    },
    "phrases": [
        "resume",
        "continue",
        "resume where I left off",
        "continue where I left off",
        "I want to continue",
        "I want to resume",
        "I want to resume where I left off",
        "I want to continue where I left off"
    ]
},
```

Update the intent map:

```javascript
// src/app.js

let myIntentMap = {
    'AMAZON.NextIntent': 'NextIntent',
    'AMAZON.PreviousIntent': 'PreviousIntent',
    'AMAZON.ResumeIntent': 'ResumeIntent'
};
```

Rename the intent in our handler from `AMAZON.ResumeIntent` to `ResumeIntent` and fix its logic:

```javascript
// src/app.js

ResumeIntent() {
    let currentIndex = this.$user.$data.currentIndex;
    let episode = Player.getEpisode(currentIndex);

    if (this.isAlexaSkill()) {
        let offset = this.$user.$data.offset;
        this.$alexaSkill.$audioPlayer.setOffsetInMilliseconds(offset).play(episode.url, `${currentIndex}`);
        
    } else if (this.isGoogleAction()) {
        this.$googleAction.$mediaResponse.play(episode.url, episode.title);
        this.$googleAction.showSuggestionChips(['pause', 'start over']);
        this.ask('Enjoy');
    }
},
```

We've made big changes to the Jovo Language Model. These changes have to be pushed to developer consoles so run `$ jovo build --deploy` and feel free to test out our implementation.

## Next Step

In the next and final step, we will refactor the project's structure, add small required intents and look ahead what else could be added to the project.

> [Step 8: The Final Steps](./step-8-final-steps.md)

<!--[metadata]: { "description": "Learn how to improve the user interaction of a Podcast Player Alexa Skill and Google Action.", "author": "kaan-kilic", "og-image": "https://www.jovo.tech/img/courses/project-3-podcast-player/podcast-player-course.jpg" }-->