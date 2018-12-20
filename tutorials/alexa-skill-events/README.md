# How to use Alexa Skill Events with the Jovo Framework

Learn more about the implementation of Alexa Skill Events in your Jovo project to get notified when your users interact with your Skill outside the active usage, for example when they enable your Skill or do account linking.

> You can find the full code example of this tutorial here: [jovo-templates/alexa/skillevents](https://github.com/jovotech/jovo-templates/tree/master/alexa/skillevents).

* [Introduction to Alexa Skill Events](#introduction-to-alexa-skill-events)
  * [Supported Events](#supported-events)
    * [Skill Enabled/Disabled](#skill-enableddisabled)
    * [Account Linked](#account-linked)
    * [Skill Permission Accepted](#skill-permission-accepted)
    * [Skill Permission Changed](#skill-permission-changed)
* [Implementation with Jovo](#implementation-with-jovo)
  * [How to enable Skill Events](#how-to-enable-skill-events)
  * [Receive the Event Requests with Jovo](#receive-the-event-requests-with-jovo)

## Introduction to Alexa Skill Events

For a long time, it has been difficult to track and understand how users engage with an Alexa Skill. When did they enable the Skill? Did they just stop using it or did they even take the action to disable it from the Skill store? Finally, there is a way to track some of the ways how users interact with your Alexa Skill: Events.

[Alexa Skill Events](https://developer.amazon.com/docs/smapi/skill-events-in-alexa-skills.html) can be used to notify you if a certain event occurs, which range from the customer disabling your Skill to them linking their account using the Alexa app. The notification comes in form of a request to your Skill, which you can then access and act on.

### Supported Events

The Alexa Skill Events currently support five interactions, which you can add to your Skill individually. Every single one of these requests contains basic data like user ID and timestamp, which allows you to map every request to an individual user.

The following events are supported:

- [Skill Enabled/Disabled](#skill-enableddisabled)
- [Account Linked](#account-linked)
- [Skill Permission Accepted](#skill-permission-accepted)
- [Skill Permission Changed](#skill-permission-changed)

#### Skill Enabled/Disabled

These are the most basic events. Your Skill gets a notification when someone enables or disables your Skill in the Alexa companion app.

#### Account Linked

This event is triggered after your user has linked their account with your service using the Alexa companion app. The incoming request after the event is triggered will contain the same access token, which you also get with every request after your user linked their account. Learn more about Alexa Account Linking here: [Login with Amazon](https://www.jovo.tech/tutorials/alexa-login-with-amazon-email/), [Account Linking with Auth0](https://www.jovo.tech/tutorials/alexa-account-linking-auth0/).

#### Skill Permission Accepted

If your user grants permissions (e.g. access to your user's lists or their location) for the first time or if they grant them after they were revoked, this Skill Event is triggered. The request includes the most recently accepted permissions. Check out the sample requests in the [official documentation](https://developer.amazon.com/docs/smapi/skill-events-in-alexa-skills.html#skill-permission-accepted-event) to get a feeling for the JSON structure.

#### Skill Permission Changed

This Skill Event is triggered when your user grants your Skill additional permission or revokes existing ones. The request includes the most recently accepted permissions.

## Implementation with Jovo

Now that you know how the Alexa Skill Events work, it's time to learn how to implement them with Jovo:

### How to enable Skill Events

To add the Alexa Skill Events to your app, you need to modify your Alexa Skill's `skill.json` file, which you can find in your under `./platforms/alexaSkill/skill.json`. But, modifying the file directly won't work, because the whole `platforms` folder is generated from scratch every time you run the `jovo build` command, which would mean your modification will be lost.

The correct way, is to add your modification to your project's `project.js` file, which will be used to create the platforms folder, i.e. configurations you make inside that file will be added to your `skill.json` file.

To enable the Skill Events you need to define an endpoint, which will receive the event request, as well as an array containing all the event types you want to

```javascript
module.exports = {
  alexaSkill: {
    nlu: 'alexa',
    manifest: {
      events: {
        endpoint: {
          uri: 'YOUR SKILL EVENT ENDPOINT' // Lambda only
        },
        subscriptions: [
					{
						eventName: 'SKILL_ENABLED'
					},
					{
						eventName: 'SKILL_DISABLED'
					},
					{
						eventName: 'SKILL_PERMISSION_ACCEPTED'
					},
					{
						eventName: 'SKILL_PERMISSION_CHANGED'
					},
					{
						eventName: 'SKILL_ACCOUNT_LINKED'
					}
        ]
      }
    }
  },
  // Other configurations
};
```

After adding the Skill Events to your `project.js` file, you have to build and deploy your platform files:

```sh
# Build alexaSkill platforms folder
$ jovo build

# Deploy to the Developer Portal
$ jovo deploy
```

### Receive the Event Requests with Jovo

The incoming requests are mapped to built-in intents of the Jovo framework. The only thing you have to do is to add them to your handlers in your `app.js` file. These handlers are placed into the `ON_EVENT` directive state to keep things organized:

```javascript
ON_EVENT: {
    'AlexaSkillEvent.SkillEnabled'() {
        console.log('AlexaSkillEvent.SkillEnabled');
    },
    'AlexaSkillEvent.SkillDisabled'() {
        console.log('AlexaSkillEvent.SkillDisabled');
    },
    'AlexaSkillEvent.SkillAccountLinked'() {
        console.log('AlexaSkillEvent.SkillAccountLinked');
    },
    'AlexaSkillEvent.SkillPermissionAccepted'() {
        console.log('AlexaSkillEvent.SkillPermissionAccepted');
    },
    'AlexaSkillEvent.SkillPermissionChanged'() {
        console.log('AlexaSkillEvent.SkillPermissionChanged');
    },
}
```

Currently this implementation will only log that these events occured, which is not that useful, but I am sure you will find great use cases for the Alexa Skill Events.

You can access the bodies of the event requests with `this.$alexaSkill.getSkillEventBody()`.

That's it, you made it to the end!

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to add skill events to your Alexa Skill", "author": "kaan-kilic", "tags": "Amazon Alexa", "og-image": "https://www.jovo.tech/blog/wp-content/uploads/2018/04/alexa-skill-events-1.jpg" }-->