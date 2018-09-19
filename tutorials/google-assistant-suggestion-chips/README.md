# Implementing Google Assistant Suggestion Chips with Jovo

Suggestion Chips for Google Assistant are a great way to offer guidance and direct the conversation for users that are interacting with your Google Action on their mobile phones. In this blogpost you're going to learn how to implement Suggestion Chips with the Jovo Framework.

- [Introduction to Suggestion Chips](#introduction-to-suggestion-chips)
- [Use Cases](#use-cases)
- [Implementing Suggestion Chips in Jovo](#implementing-suggestion-chips-in-jovo)

## Introduction to Suggestion Chips

![](./img/suggestion-chips-actions-on-google.jpg)

Suggestion Chips are a form of visual output, which (as the name already tells) provide the user with a suggestion how to respond. The user can select each suggestion chip, which will be considered as their answer. You can find the full reference from Google here: [Actions on Google Docs > Responses > Suggestion Chip](https://developers.google.com/actions/assistant/responses#suggestion_chip).

You might ask yourself, why to even bother implementing that feature, since neither the Google Home nor the Google Home Mini have a screen. The answer is the crazy number of smartphones that have the Google Assistant integrated. In May 2017, the most common way for an American adult to use a voice assistant was through their smartphone ([source](http://www.pewresearch.org/fact-tank/2017/12/12/nearly-half-of-americans-use-digital-voice-assistants-mostly-on-their-smartphones/)). Suggestion chips are a great way to help them reach their goals faster by guiding the conversation.

![](./img/line2.png)

## Use Cases

Let's go through some of the use cases of suggestions chips, so you get an overview of the possibilities.

**Example 1: Quiz App** Probably the most basic use case. You ask the user multiple choice questions and provide the possible answers as suggestion chips.

![](./img/chip_03-1.png)

**Example 2: Possible Responses** Suggestion Chips can also be used to hint at possible responses. These could be answers to the question your application asked as well as help commands. 

![](./img/chip_02-1.png)

## Implementing Suggestion Chips in Jovo

![](./img/suggestion-chips-jovo-1024x630.jpg)

In this example, we are just going to modify our [Hello World Sample Voice App](https://github.com/jovotech/jovo-sample-voice-app-nodejs) to display some names as suggestion chips when we're asking our users for their names. You can find the full example code on GitHub: [jovotech/jovo-templates/tutorials/suggestion-chips](https://github.com/jovotech/jovo-templates/tree/master/tutorials/suggestion-chips).

Suggestion chips are really easy to implement in Jovo. Technically it only takes one line of code:

```javascript
this.googleAction().showSuggestionChips(['Elliot', 'J.D', 'Turk']);
this.ask('Hello World! What\'s your name?', 'Please tell me your name.');
```

This is what the result looks like in the Actions on Google Simulator:

![](./img/chip_01-1.png)

If I now click the "Elliot" button, this is treated as if I said "Elliot" going into the 'MyNameIsIntent' intent on Dialogflow.

That's it, you made it to the end!

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to add suggestion chips to your Google Action", "author": "kaan-kilic", "tags": "Google Assistant" }-->
