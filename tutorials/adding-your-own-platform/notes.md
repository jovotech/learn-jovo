# Notes

* Generally each platform has to send a request to the Jovo project's endpoint and expect a JSON response back.
* This is going to be a rather long tutorial. Adding a platform to the Jovo Framework is not really a difficult task but it simply takes some time to assemble all the parts.
* General facts about the Jovo Framework:
  * written in Typescript
  * lerna repository
  * Working with the Jovo repo:
    * `npm run devsetup`, `npm run clean`, `npm run tslint`, `npm run test`, etc.
  * We're happy to help you with your integration. You either create an Issue on Github and tag (@kaan, @max, @ruben) or reach us on Slack (link zu Slack und @kaan, @max, @ruben)
* List of the current platforms.
* Why do we choose Autopilot as an example?
  * it's a small platform (not too many features)
  * the current implementation consists of only the necessary files to integrate a platform in the first place.
  * Other platforms like Alexa are way to big to go over in a tutorial, but we will mention some of its parts here and there to explain stuff that might pop up while integrating another platform.
* How does the Jovo Framework process a request and send out a response?
  * middleware chain (focus on the ones that are accessed by platforms)
  * a little more detailed explanation of the output object.
  * Each platform also has its own set of action sets. (`$init`, `$request`, etc.)
* What does each platform integration have to do?
  * Receive and process request
  * support routing by defining the request type e.g. LAUNCH, INTENT, END, etc.
  * allow creation of output object in handler e.g. ask, tell, audio player, cards, etc.
  * create response JSON out of output object
  * allow unit testing (TestSuite)
* Structure of source code:
  * General repository structure (clients, integrations, platforms, jovo-core, jovo-framework)
  * General package structure:
```js
src/
test/
.npmignore
.prettierrc.js
LICENSE
package-lock.json
package.json
README.md
tsconfig.json
tslint.json
```

  * Platform integration structure:
    * **General Plugin (e.g. Autopilot.ts)**: That's the plugin file of the whole platform. It combines the all the other modules and hooks up to the different middlewares executed throughout a response.
    * The `core` folder contains the backbone of your platform:
      * **Jovo implementation (e.g. AutopilotBot.ts)**: That's your platforms implementation of the Jovo object (better known as `this` in your handler.). Besides references to other objects like `$response` and `$speech`, it contains helper functions. Some of these are abstract functions from the Jovo class but most are platform specific helpers. So there is no real guideline there. Simply add what you think might be useful.
      * **The Request class (e.g. AutopilotRequest.ts)**: The class should contain the same properties the JSON request has. Two important functions are `fromJSON()` and `toJSON()`. `fromJSON` creates a request object from the JSON request. `toJSON()` creates the JSON request from the request object.
      * **The Response class (e.g. AutopilotResponse.ts)**: Same as the request object just for the response.
      * **The User class (e.g. Autopilotuser.ts)**: There is a base user class which you can extend with your platform integration. The Alexa User implements features like shopping lists, location data, reminders, etc.. If there are no additional features, you can simply use a basic implementation. Check out the `AutopilotUser.ts` class for an example.
      * **The SpeechBuilder Class (e.g. AutopilotSpeechBuilder.ts)**: Again, while there is a base SpeechBuilder class, you can provide an extended one here, which supports platform specific features.
      * **The Request- & ResponseBuilder (e.g. AutopilotRequestBuilder.ts, AutopilotResponseBuilder.ts)** are classes used by the Jovo TestSuite to allow user's to run unit tests for your platform.
    * The `modules` folder contains all of the files needed to process requests and and prepare the response:
      * **PlatformCore.ts**: The core module takes care of initializing the Autopilot Jovo object, request object, helps with the routing (type of request), initializes the session data and also is tasked with parsing the core parts of the output object (e.g. ask, tell) into the actual response JSON. All in all it takes care of all the core functionality needed to process a request.
      * besides that the modules folder contains all the output capabilities of a platform. For example, there is a AudioPlayer module which handles all of the audio functionality. Most of the times, these modules have their own object which can be accesses using the platform's jovo object, e.g. `this.$autopilotBot.$audioPlayer`. Another example would be cards, which also have their own module, but not their own object accessed using the jovo object. These can be accesses from the jovo object directly, e.g. `this.$autopilotBot.showStandardCard()`. 
    * The `services` folder contains everything that is working with an API. Each separate API has it's own class. For example, the `AlexaContact.ts` handles calls to Alexa's Contact API. 
```js
  |── core/
    |── AutopilotBot.ts
    |── AutopilotRequest.ts
    |── AutopilotRequestBuilder.ts
    |── AutopilotResponse.ts
    |── AutopilotResponseBuilder.ts
    |── AutopilotSpeechBuilder.ts
    └── AutopilotUser.ts
  |── modules/
    |── AudioPlayer.ts
    |── AutopilotCore.ts
    └── AutopilotNLU.ts
  |── Autopilot.ts
  └── index.ts
```
* Testing:
  * Every platform has to fulfill the `FrameworkBase` tests. These test the base capabilities that are the same for every platform (routing, basic output, session, user, etc.). (Where do we find them? Platforms have their own modified ones lol)
  * Try to add as many as possible tests for all your platform specific stuff as well. That makes maintaining the platform integration easier for us.
* Next Steps: add CLI support (language model, platform files, deployment etc.)