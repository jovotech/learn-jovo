# Step 5: Structuring the Content in i18n

...

* [Separating Content from Code](#separating-content-from-code)
* [Creating Translation Files with i18next](#creating-translation-files-with-i18next)
   * [i18n Folder Structure](#i18n-folder-structure)
	 * [Migrating the Content to i18n](#migrating-the-content-to-i18n)
* [Accessing the Content in the Code](#accessing-the-content-in-the-code)
* [Next Step](#next-step)


## Separating Content from Code

A lot of reasons:

* It is easier to update the content for non-technical people
* Once you have the structure, you can also localize the content
* You can later make the step and use our [CMS integrations](https://www.jovo.tech/docs/cms) more easily


## Creating Translation Files with i18next

> [Learn more about Jovo and i18n](https://www.jovo.tech/docs/output/i18n).

Jovo offers a built-in integration with [i18next](https://www.i18next.com/), a powerful internationalization framework.

### i18n Folder Structure

Add `i18n` folder to `src`.

![Jovo Folder Structure for i18n files](https://www.jovo.tech/img/docs/v2/folder-structure-i18n.jpg)

Create a `en-US.json` file.

This file can be structured like this:

```js
{
  "translation": {
    "key1": "value1",
    "key2": "value2"
  }
}
```

### Migrating the Content to i18n

Currently, the content is structured either in constants, like this:

```js
const welcomeMessage = `Welcome to the United States Quiz Game!  You can ask me about any of the fifty states and their capitals, or you can ask me to start a quiz.  What would you like to do?`;
const startQuizMessage = `OK.  I will ask you 10 questions about the United States. `;
```

Note how the `startQuizMessage` has an empty space at the end because another message gets appended. We will deal with this later.

Also, some messages are retrieved via method calls, like this:

```js
function getQuestion(counter, property, item) {
  return `Here is your ${counter}th question.  What is the ${formatCasing(property)} of ${item.StateName}?`;
}
```

#### Migrating Constants

```js
{
  "translation": {
    "welcomeMessage": "Welcome to the United States Quiz Game!  You can ask me about any of the fifty states and their capitals, or you can ask me to start a quiz.  What would you like to do?",
    "startQuizMessage": "OK. I will ask you 10 questions about the United States.",
    "exitSkillMessage": "Thank you for playing the United States Quiz Game!  Let's play again soon!",
    "repromptSpeech": "Which other state or capital would you like to know about?",
    "helpMessage": "I know lots of things about the United States.  You can ask me about a state or a capital, and I'll tell you what I know.  You can also test your knowledge by asking me to start a quiz.  What would you like to do?"
  }
}
```

Speechcons?



#### Migrating Functions with Parameters

Variables


```js
"question": "Here is your {counter}th question.  What is the {property) of {StateName}?",
"answer": {
	"Abbreviation": "The {property} of {StateName} is <say-as interpret-as='spell-out'>{property}</say-as>.",
	"default": "The {property} of {StateName} is {property}."
},
"badAnswer": "I'm sorry. {item} is not something I know very much about in this skill.",
"currentScore": "Your current score is {score} out of {counter}.",
"finalScore": "Your final score is {score} out of {counter}.",
"speechDescription": "{StateName} is the {StatehoodOrder}th state, admitted to the Union in {StatehoodYear}. The capital of {StateName} is {Capital}, and the abbreviation for {StateName} is <break strength='strong'/><say-as interpret-as='spell-out'>{Abbreviation}</say-as>. I've added {StateName} to your Alexa app. Which other state or capital would you like to know about?"
```




## Accessing the Content in the Code

this.t

this.$speech.t ?



## Next Step

In the next step, we will learn how to keep streaming additional audio files after the first song finished.

> [Step 2: Streaming Multiple Files in a Row](./step-2-stream-multiple-files.md)

<!--[metadata]: { "description": "Learn about the differences of an Alexa Skill project built with the Jovo Framework compared to the Alexa Skills Kit (ASK) SDK.", "author": "jan-koenig" }-->