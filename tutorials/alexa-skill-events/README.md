# How to use Alexa Skill Events with the Jovo Framework

Learn more about the implementation of Alexa Skill Events in your Jovo project to get notified when your users interact with your Skill outside the active usage, for example when they enable your Skill or do account linking.

📋 You can find the full code example of this tutorial here: [jovo-templates/alexa/skillevents](https://github.com/jovotech/jovo-templates/tree/master/alexa/skillevents).

### Contents

- [Introduction to Alexa Skill Events](#introduction-to-alexa-skill-events)
	- [How to enable Skill Events](#how-to-enable-skill-events)
	- [Supported Events](#supported-events)
		- [Skill Enabled/Disabled](#skill-enableddisabled)
		- [Account Linked](#account-linked)
		- [Skill Permission Accepted](#skill-permission-accepted)
		- [Skill Permission Changed](#skill-permission-changed)
- [Implementation with Jovo](#implementation-with-jovo)
	- [Alternative 1: Modify the Skill.json](#alternative-1-modify-the-skilljson)
	- [Alternative 2: Use the Template](#alternative-2-use-the-template)
	- [Receive the Event Requests with Jovo](#receive-the-event-requests-with-jovo)

## Introduction to Alexa Skill Events

For a long time, it has been difficult to track and understand how users engage with an Alexa Skill. When did they enable the Skill? Did they just stop using it or did they even take the action to disable it from the Skill store? Finally, there is a way to track some of the ways how users interact with your Alexa Skill: Events.

[Alexa Skill Events](https://developer.amazon.com/docs/smapi/skill-events-in-alexa-skills.html) can be used to notify you if a certain event occurs, which range from the customer disabling your Skill to them linking their account using the Alexa app. The notification comes in form of a request to your Skill, which you can then access and act on.

### How to enable Skill Events

The tricky part about Alexa Skill Events is that they can't be enabled through the Amazon Developer Portal. To enable them, you need to use ASK CLI (the command line interface of the Alexa Skills Kit), or use the Alexa Skill Management API (SMAPI) directly. An `events` object needs to be added to the `skill.json` file, which looks like this:

```javascript
"events": {
	"endpoint": {
		"uri": "YOUR ENDPOINT"
	},
	"subscriptions": [
		{
			"eventName": "EVENT NAME HERE"
		}
	]
}
```

This specifies the endpoint that is called from the Skill events, and subscribes to a number of events that are specified in the `subscriptions` array.

To enable the Skill Events, the modifications to the skill.json need to be pushed to the Amazon Developer Portal by using the ASK CLI. Find out more how it works in the [implementation section below](#implementation-with-jovo).

### Supported Events

The Alexa Skill Events currently support five interactions, which you can add to your Skill individually. Every single one of these requests contains basic data like user ID and timestamp, which allows you to map every request to an individual user.

The following events are supported:

- [Skill Enabled/Disabled](#skill-enableddisabled)
- [Account Linked](#account-linked)
- [Skill Permission Accepted](#skill-permission-accepted)
- [Skill Permission Changed](#skill-permission-changed)


#### Skill Enabled/Disabled

```javascript
"events": {
	"endpoint": {
		"uri": "YOUR ENDPOINT"
	},
	"subscriptions": [
		{
			"eventName": "SKILL_ENABLED"
		},
		{
			"eventName": "SKILL_DISABLED"
		}
	]
}
```

These are the most basic events. Your Skill gets a notification when someone enables or disables your Skill in the Alexa companion app.

#### Account Linked

```javascript
"events": {
	"endpoint": {
		"uri": "YOUR ENDPOINT"
	},
	"subscriptions": [
		{
			"eventName": "SKILL_ACCOUNT_LINKED"
		}
	]
}
```

This event is triggered after your user has linked their account with your service using the Alexa companion app. The incoming request after the event is triggered will contain the same access token, which you also get with every request after your user linked their account. Learn more about Alexa Account Linking here: [Login with Amazon](https://www.jovo.tech/blog/alexa-login-with-amazon-email/), [Account Linking with Auth0](https://www.jovo.tech/blog/alexa-account-linking-auth0/).

#### Skill Permission Accepted

```javascript
"events": {
	"endpoint": {
		"uri": "YOUR ENDPOINT"
	},
	"subscriptions": [
		{
			"eventName": "SKILL_PERMISSION_ACCEPTED"
		}
	]
}
```

If your user grants permissions (e.g. access to your user's lists or their location) for the first time or if they grant them after they were revoked, this Skill Event is triggered. The request includes the most recently accepted permissions. Check out the sample requests in the [official documentation](https://developer.amazon.com/docs/smapi/skill-events-in-alexa-skills.html#skill-permission-accepted-event) to get a feeling for the JSON structure.

#### Skill Permission Changed

```javascript
"events": {
	"endpoint": {
		"uri": "YOUR ENDPOINT"
	},
	"subscriptions": [
		{
			"eventName": "SKILL_PERMISSION_CHANGED"
		}
	]
}
```

This Skill Event is triggered when your user grants your Skill additional permission or revokes existing ones. The request includes the most recently accepted permissions.

## Implementation with Jovo

Now that we know how the Alexa Skill Events are structured. let's learn how to access the requests in Jovo. To subscribe to events, you have two options:

*   [Alternative 1: Modify the Skill.json in the platforms folder](#alternative-1-modify-the-skill-json)
*   [Alternative 2: Use the Jovo Skill Events Template](#alternative-2-use-the-template)

After this, you can set up the code to [Receive the Event Requests in your Jovo app](#receive-the-event-requests-with-jovo).

### Alternative 1: Modify the Skill.json

If you have already created the platform specific language model files with [jovo build](https://www.jovo.tech/framework/docs/cli#jovo-build), you can find the `skill.json` in the `/platforms/alexaSkill/` folder of your Jovo project. [Learn more about the platforms folder here](https://www.jovo.tech/framework/docs/model/platforms).

The `skill.json` of a newly created project looks like this:

```javascript
{
	"manifest": {
		"publishingInformation": {
			"locales": {
				"en-US": {
					"summary": "Sample Short Description",
					"examplePhrases": [
						"Alexa open hello world"
					],
					"name": "skillEvents",
					"description": "Sample Full Description"
				}
			},
			"isAvailableWorldwide": true,
			"testingInstructions": "Sample Testing Instructions.",
			"category": "EDUCATION_AND_REFERENCE",
			"distributionCountries": []
		},
		"apis": {
			"custom": {
				"endpoint": {
					"sslCertificateType": "Wildcard",
					"uri": "YOUR ENDPOINT"
				}
			}
		},
		"manifestVersion": "1.0"
	}
}
```

To use the Skill Events you have to add the `events` object, which contains an endpoint (in most cases your [Jovo Webhook](https://www.jovo.tech/framework/docs/server/webhook#jovo-webhook) or Lambda function) and the events, which you want to enable. Here's an example:

```javascript
{
	"manifest": {
		"publishingInformation": {
			"locales": {
				"en-US": {
					"summary": "Sample Short Description",
					"examplePhrases": [
						"Alexa open hello world"
					],
					"name": "skillEvents",
					"description": "Sample Full Description"
				}
			},
			"isAvailableWorldwide": true,
			"testingInstructions": "Sample Testing Instructions.",
			"category": "EDUCATION_AND_REFERENCE",
			"distributionCountries": []
		},
		"apis": {
			"custom": {
				"endpoint": {
					"sslCertificateType": "Wildcard",
					"uri": "YOUR ENDPOINT"
				}
			}
		},
		"events": {
			"endpoint": {
				"uri": "YOUR LAMBDA ENDPOINT"
			},
			"subscriptions": [
				{
					"eventName": "SKILL_ENABLED"
				},
				{
					"eventName": "SKILL_DISABLED"
				},
				{
					"eventName": "SKILL_PERMISSION_ACCEPTED"
				},
				{
					"eventName": "SKILL_PERMISSION_CHANGED"
				},
				{
					"eventName": "SKILL_ACCOUNT_LINKED"
				}
			]
		},
		"manifestVersion": "1.0"
	}
}
```

Don't forget to add your own endpoint to the `skill.json` if you copy and paste the example.

After that deploy your changed `skill.json` using the Jovo CLI:

```sh
$ jovo deploy
```

### Alternative 2: Use the Template

There is an even easier way than the one above! If you don't want to manually change the `skill.json` file, you can also just download the [Jovo Template for Alexa Skill Events](https://github.com/jovotech/jovo-templates/tree/master/alexa/skillevents):

```sh
$ jovo new <directory> --template alexa/skillevents
```

This comes with a modified app.json (usually created with [jovo init](https://www.jovo.tech/framework/docs/cli#jovo-init)), which looks like this
:
```javascript
{
	"alexaSkill": {
		"nlu": "alexa",
		"manifest": {
			"events": {
				"endpoint": {
					"uri": "YOUR LAMBDA ENDPOINT"
				},
				"subscriptions": [
					{
						"eventName": "SKILL_ENABLED"
					},
					{
						"eventName": "SKILL_DISABLED"
					},
					{
						"eventName": "SKILL_PERMISSION_ACCEPTED"
					},
					{
						"eventName": "SKILL_PERMISSION_CHANGED"
					},
					{
						"eventName": "SKILL_ACCOUNT_LINKED"
					}
				]
			}
		}
	},
	"endpoint": "YOUR ENDPOINT"
}
```

As you can see, the section is added to the manifest object and is added to the skill.json during the [jovo build command](https://www.jovo.tech/framework/docs/cli#jovo-build).

To get it to work, add your endpoint to both the events object and the endpoint below, and use the following commands:

```sh
# Build alexaSkill platforms folder
$ jovo build

# Deploy to the Developer Portal
$ jovo deploy
```

That's it! The template also comes with a handler that makes it easier for you to access the Alexa Skill Event requests, which you can find in the next section.

### Receive the Event Requests with Jovo

As I said earlier, the incoming requests are mapped to handlers of the Jovo framework. The only thing you have to do is to add them to your handlers in your `app.js` file (which is already done if you use the [Skill Events template](https://github.com/jovotech/jovo-templates/tree/master/alexa/skillevents)). These handlers are placed into the `ON_EVENT` directive state to keep things organized:

```javascript
'ON_EVENT': {
    'AlexaSkillEvent.SkillEnabled': function() {
        console.log('AlexaSkillEvent.SkillEnabled');
    },
    'AlexaSkillEvent.SkillDisabled': function() {
        console.log('AlexaSkillEvent.SkillDisabled');
    },
    'AlexaSkillEvent.SkillAccountLinked': function() {
        console.log('AlexaSkillEvent.SkillAccountLinked');
    },
    'AlexaSkillEvent.SkillPermissionAccepted': function() {
        console.log('AlexaSkillEvent.SkillPermissionAccepted');
    },
    'AlexaSkillEvent.SkillPermissionChanged': function() {
        console.log('AlexaSkillEvent.SkillPermissionChanged');
    },
}
```

Currently this implementation will only log that these events occured, which is not that useful, but I am sure you will find great use cases for the Alexa Skill Events.

You can access the bodies of the event requests with `this.$alexaSkill.getSkillEventBody()`.

That's it, you made it to the end!

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to add skill events to your Alexa Skill", "author": "kaan-kilic", "tags": "Amazon Alexa", "og-image": "https://www.jovo.tech/blog/wp-content/uploads/2018/04/alexa-skill-events-1.jpg" }-->