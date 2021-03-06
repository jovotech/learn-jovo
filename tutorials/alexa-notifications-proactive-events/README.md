# Send Notifications Using the Alexa Proactive Events API

![Use the Alexa Proactive Events API with Jovo](./img/alexa-proactive-events-notifications.jpg "Learn how to implement Alexa Notifications with the Jovo Framework")

In this tutorial, I want to show you how to send [proactive events](https://www.jovo.tech/marketplace/jovo-platform-alexa#proactive-events) with your Alexa Skills built with Jovo. This will enable you to send notifications to your users both from inside your Alexa Skill as well as outside of a session.

* [Introduction](#introduction)
* [Alexa Skill Permissions and Publications](#alexa-skill-permissions-and-publications)
* [Proactive Event Object](#proactive-event-object)
* [Get the Access Token](#get-the-access-token)
* [Send the Proactive Event](#send-the-proactive-event)
* [Triggering the Proactive Event API from Outside](#triggering-the-proactive-event-api-from-outside)

## Introduction

> Docs: [Alexa Proactive Events API](https://www.jovo.tech/marketplace/jovo-platform-alexa#proactive-events).

The feature to send out notifications was one of the most requested ones in the early days of Alexa Skill development. In 2017 it was announced that the feature would be tested by a limited amount of handpicked developers. Although it was never released to the public fully, it got redesigned to the **Proactive Events API**, which was released in December of 2018.

The Proactive Events API's only use case, as of yet, is to send out notifications, but Amazon plans to expand the API in the future.

These notifications come with restrictions, the biggest one being that you have to implement one of Amazon's [event schemas](https://www.jovo.tech/marketplace/jovo-platform-alexa#event), which themselves come with certain restrictions to the values you're allowed to use.

Just like with the Proactive Events API itself, it seems to be planned to expand the available schemas, so you should keep an eye on the list, even if you don't find anything of use as of yet.

To use the Proactive Events API you have to follow these steps:

* [Alexa Skill Permissions and Publications](#alexa-skill-permissions-and-publications): Add the Proactive Events API and other necessary settings to your Alexa Skill.
* [Proactive Event Object](#proactive-event-object): Prepare the event object containing the data that will be sent.
* [Get the Access Token](#get-the-access-token): Request an access token from Amazon, which you get after authorizing to be able to send out the events.
* [Send the Proactive Event](#send-the-proactive-event)


## Alexa Skill Permissions and Publications

To be able to use the Proactive Events API, you have to add it to the permissions that your Alexa Skill needs to function. Besides that, you also have to specify the event schemas you're implementing, to which your users can subscribe to.

For that, you have to modify your `project.js` file and add the following:

```javascript
// project.js

module.exports = {
    alexaSkill: {
        nlu: 'alexa',
        manifest: {
            permissions: [
                {
                    name: 'alexa::devices:all:notifications:write'
                }
            ],
            events: {
                publications: [
                    {
                        eventName: 'AMAZON.WeatherAlert.Activated'
                    }
                ],
                endpoint: {
                    uri: "<yourEndpoint>" // Simply place your Jovo Webhook URL here
                },
            }
        }
    },
    // Other settings
};
```

The `publications` array should only include the event schemas you're actually implementing. For this tutorial, we only need the `WeatherAlert.Activated` event.

For these changes to take effect, you have to build and deploy your project. In this case, we don't need to redeploy anything besides the `info` files (`skill.json`), which we can do the following way:

```sh
# Build platform specific files for Alexa
$ jovo build --platform alexaSkill

# Deploy only the Skill information for Alexa
$ jovo deploy --platform alexaSkill --target info

# Alternative: Shortcut for both commands
$ jovo build -p alexaSkill --deploy --target info
```

After the deployment finished, you can find the `clientId` and `clientSecret` on the `PERMISSIONS` tab of your Skill on the Alexa Developer Console, which you will need later on:

![Proactive Events API Client ID and Client Secret](img/proactive_events_clientid_clientsecret.png)

Note: Sometimes, it can take a bit for them to appear. If they don't show up right away, try to refresh your browser window.


## Proactive Event Object

The Proactive Event object contains the data you want to send to your users, as well as settings like the `expiryTime` of the notification, which represents the date and time at which the notification will be deleted automatically.

We will create a new intent called `WeatherAlertIntent`, where we will prepare and later on send out the Proactive Event.

```javascript
async WeatherAlertIntent() {
    // Sets timestamp to current date and time
    let timestamp = new Date();
    timestamp = timestamp.toISOString();

    // Sets expiryTime 23 hours ahead of the current date and time
    let expiryTime = new Date();
    expiryTime.setHours(expiryTime.getHours() + 23);
    expiryTime = expiryTime.toISOString();

    const proactiveEventObject = {
        "timestamp": timestamp,
        "referenceId": "test-0001",
        "expiryTime": expiryTime,
        "event": {
            "name": "AMAZON.WeatherAlert.Activated",
            "payload": {
                "weatherAlert": {
                    "source": "localizedattribute:source",
                    "alertType": "TORNADO"
                }
            }
        },
        "localizedAttributes": [
            {
                "locale": "en-US",
                "source": "English Weather Channel"
            }
        ],
        "relevantAudience": {
            "type": "Multicast",
            "payload": {}
        }
    };
}
```

> You can find out more about the Proactive Event object and its attributes [here](https://www.jovo.tech/marketplace/jovo-platform-alexa#proactive-event-object)


## Get the Access Token

Now that we have Proactive Event object prepared, we have to request the access token, that's needed to send the Proactive Event. The access token will be granted after we've authorized ourselves with Amazon using the `clientId` and `clientSecret` we got earlier:

```javascript
async WeatherAlertIntent() {
    // ...

    const accessToken = await this.$alexaSkill.$proactiveEvent.getAccessToken(
        '<your-clientId>',
        '<your-clientSecret>'
    );
}
```

## Send the Proactive Event

With the Proactive Event object and the access token prepared, we can now send the Proactive Event:


```javascript
async WeatherAlertIntent() {
    // ...

    const result = await this.$alexaSkill.$proactiveEvent.sendProactiveEvent(proactiveEventObject, accessToken);
}
```

The complete `WeatherAlertIntent` looks like this:

```javascript
async WeatherAlertIntent() {
    // Sets timestamp to current date and time
    let timestamp = new Date();
    timestamp = timestamp.toISOString();

    // Sets expiryTime 23 hours ahead of the current date and time
    let expiryTime = new Date();
    expiryTime.setHours(expiryTime.getHours() + 23);
    expiryTime = expiryTime.toISOString();

    const proactiveEventObject = {
        "timestamp": timestamp,
        "referenceId": "test-0001",
        "expiryTime": expiryTime,
        "event": {
            "name": "AMAZON.WeatherAlert.Activated",
            "payload": {
                "weatherAlert": {
                    "source": "localizedattribute:source",
                    "alertType": "TORNADO"
                }
            }
        },
        "localizedAttributes": [
            {
                "locale": "en-US",
                "source": "English Weather Channel"
            }
        ],
        "relevantAudience": {
            "type": "Multicast",
            "payload": {}
        }
    };

    const accessToken = await this.$alexaSkill.$proactiveEvent.getAccessToken(
        '<your-clientId>',
        '<your-clientSecret>'
    );

    const result = await this.$alexaSkill.$proactiveEvent.sendProactiveEvent(proactiveEventObject, accessToken);
}
```

That's all you need to send out an event using the Proactive Events API.

But, there is one issue. How are you going to trigger the intent to send out the event? You might find a creative solution where you somehow route through the `WeatherAlertIntent` in the middle of a session, but even then you are bound to there actually being a session in the first place, which pretty much nullifies to actual benefits of the Proactive Events API, namely messaging your user while they're **not** using your Skill. 

There are certain event schemas, where it will be perfectly fine to run the Proactive Events API inside your Alexa Skill. These are schemas that depend on two users, i.e. one user's action is the trigger to send out the notification to the other. For example, the `SocialGameInvite.Available` scheme, where you will send out a notification to user 2 if user 1 invited them.

Other event schemas work a little different. Their trigger comes from an external source, e.g. the `WeatherAlert.Activated`, `MediaContent.Available`, and `SportsEvent.Updated` event schemas. In the next section, we will go over a basic idea on how you might trigger the intent externally.

## Triggering the Proactive Event API from Outside

This section won't help you create the whole trigger but rather show you the part where you trigger your Jovo project, e.g. we won't go over the steps explaining how you might set up an AWS Lambda function, which gets executed as soon as there is a weather alert, but rather the part where we send out a request to our Jovo project to send out the Proactive Event, i.e. trigger the `WeatherAlertIntent`.

To trigger the intent with the code to send out events, in our case the `WeatherAlertIntent`, we have to send a request to our endpoint with the intent set to one executing the Proactive Event.

That intent should not be included in your language model to prevent your user may be randomly triggering it.

Here's a sample request, which you can use:

```javascript
{
    "version": "1.0",
    "session": {
        "new": false,
        "sessionId": "amzn1.echo-api.session.0000000-0000-0000-0000-00000000000",
        "application": {
            "applicationId": "amzn1.echo-sdk-ams.app.000000-d0ed-0000-ad00-000000d00ebe"
        },
        "attributes": {},
        "user": {
            "userId": "amzn1.account.AM3B00000000000000000000000"
        }
    },
    "context": {
        "System": {
            "application": {
                "applicationId": "amzn1.echo-sdk-ams.app.000000-d0ed-0000-ad00-000000d00ebe"
            },
            "user": {
                "userId": "amzn1.account.AM3B00000000000000000000000"
            },
            "device": {
                "deviceId": "amzn1.ask.device.XXXXXA6LX6BOBJF6XNWQM2ZO4NVVGZRFFEL6PMXKWLOHI36IY3B4XCSZKZPR42RAWCBSQEDNGS746OCC2PKR5KDIVAUY6F2DX5GV2SQAXPD7GMKQRWLG4LFKXFPVLVTXHFGLCQKHB7ZNBKLHQU4SJG6NNGA",
                "supportedInterfaces": {
                    "AudioPlayer": {}
                }
            },
            "apiEndpoint": "https://api.amazonalexa.com"
        },
        "AudioPlayer": {
            "offsetInMilliseconds": 0,
            "playerActivity": "IDLE"
        }
    },
    "request": {
        "type": "IntentRequest",
        "requestId": "amzn1.echo-api.request.0000000-0000-0000-0000-00000000000",
        "timestamp": "2015-05-13T12:34:56Z",
        "dialogState": "COMPLETED",
        "locale": "en-US",
        "intent": {
            "name": "WeatherAlertIntent",
            "confirmationStatus": "NONE",
            "slots": {}
        }
    }
}
```

As you can see the `request` object simply has the `intent` set to `WeatherAlertIntent`, besides that, everything is unimportant. The `timestamp`, the `applicationId`, etc. don't matter.

To trigger the intent, you simply use this request as the body of a https request sent to your endpoint:

```javascript
const https = require('https');

async function sendRequest(postData) {
    return new Promise((resolve, reject) => {
        const opt = {
            hostname: 'webhook.jovo.cloud',
            path: '/<your-webhook-path>', // Add your own webhook path here, i.e the part after the ".cloud"
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
                'Content-Length': Buffer.byteLength(postData),
                'Accept-Charset': 'utf-8',
                'Signature': {
                    'SignatureCertChainUrl': 'https://s3.amazonaws.com/echo.api/echo-api-cert.pem'
                }
            },
        };
        const req = https.request(opt, (res) => {
            res.setEncoding('utf8');
            let rawData = '';
            res.on('data', (chunk) => {
                rawData += chunk;
            });
            res.on('end', () => {
                let parsedData;
                if (res.statusCode === 204) { // no content
                    return resolve(res.statusCode);
                }
                try {
                    if (rawData.length > 0) {
                        parsedData = JSON.parse(rawData);
                        return resolve(parsedData);
                    }
                } catch (e) {
                    return reject(JSON.parse(e));
                }
                resolve(res.statusCode);
            });
        }).on('error', (e) => {
            reject(e);
        });
        req.write(postData);
        req.end();
    });
}
```

Simply call the function and parse the stringified request object as a parameter to trigger your Alexa Skill's `WeatherAlertIntent`:

```javascript
const https = require('https');

async function sendRequest(postData) {
    return new Promise((resolve, reject) => {
        const opt = {
            hostname: 'webhook.jovo.cloud',
            path: '/<your-webhook-path>',
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
                'Content-Length': Buffer.byteLength(postData),
                'Accept-Charset': 'utf-8',
                'Signature': {
                    'SignatureCertChainUrl': 'https://s3.amazonaws.com/echo.api/echo-api-cert.pem'
                }
            },
        };
        const req = https.request(opt, (res) => {
            res.setEncoding('utf8');
            let rawData = '';
            res.on('data', (chunk) => {
                rawData += chunk;
            });
            res.on('end', () => {
                let parsedData;
                if (res.statusCode === 204) { // no content
                    return resolve(res.statusCode);
                }
                try {
                    if (rawData.length > 0) {
                        parsedData = JSON.parse(rawData);
                        return resolve(parsedData);
                    }
                } catch (e) {
                    return reject(JSON.parse(e));
                }
                resolve(res.statusCode);
            });
        }).on('error', (e) => {
            reject(e);
        });
        req.write(postData);
        req.end();
    });
}

let postData = {
    "version": "1.0",
    "session": {
        "new": false,
        "sessionId": "amzn1.echo-api.session.0000000-0000-0000-0000-00000000000",
        "application": {
            "applicationId": "amzn1.echo-sdk-ams.app.000000-d0ed-0000-ad00-000000d00ebe"
        },
        "attributes": {},
        "user": {
            "userId": "amzn1.account.AM3B00000000000000000000000"
        }
    },
    "context": {
        "System": {
            "application": {
                "applicationId": "amzn1.echo-sdk-ams.app.000000-d0ed-0000-ad00-000000d00ebe"
            },
            "user": {
                "userId": "amzn1.account.AM3B00000000000000000000000"
            },
            "device": {
                "deviceId": "amzn1.ask.device.XXXXXA6LX6BOBJF6XNWQM2ZO4NVVGZRFFEL6PMXKWLOHI36IY3B4XCSZKZPR42RAWCBSQEDNGS746OCC2PKR5KDIVAUY6F2DX5GV2SQAXPD7GMKQRWLG4LFKXFPVLVTXHFGLCQKHB7ZNBKLHQU4SJG6NNGA",
                "supportedInterfaces": {
                    "AudioPlayer": {}
                }
            },
            "apiEndpoint": "https://api.amazonalexa.com"
        },
        "AudioPlayer": {
            "offsetInMilliseconds": 0,
            "playerActivity": "IDLE"
        }
    },
    "request": {
        "type": "IntentRequest",
        "requestId": "amzn1.echo-api.request.0000000-0000-0000-0000-00000000000",
        "timestamp": "2015-05-13T12:34:56Z",
        "dialogState": "COMPLETED",
        "locale": "en-US",
        "intent": {
            "name": "WeatherAlertIntent",
            "confirmationStatus": "NONE",
            "slots": {}
        }
    }
}

postData = JSON.stringify(postData);

sendRequest(postData).then((result) => {
    console.log(result);
}, (reason) => {
    console.log(reason);
});
```

Depending on the region you are in, you have to change the `apiEndpoint` property of the request:

* North America: `https://api.amazonalexa.com`
* Europe: `https://api.eu.amazonalexa.com`
* Far East: `https://api.fe.amazonalexa.com`

For testing purposes, we use the [Jovo Webhook](https://www.jovo.tech/docs/jovo-webhook#jovo-webhook) as it is the most convenient one for local testing. If you later on host your project on [AWS Lambda](https://aws.amazon.com/lambda/), you might use an [AWS API Gateway](https://aws.amazon.com/api-gateway/) which you point to the Lambda function and send the request to the API endpoint. Technically it works the same for every other cloud service provider. You simply want to send the https request to the endpoint, where your project is hosted on.

To test everything, simply create a new javascript file and add the code above. Run the code and you should receive a notification on your Alexa device.

```text
$ node <yourJsFile>
```

> If the notification didn't show up on your Echo device connected to the dev account, please check if the necessary permission was given for the skill to send notifications. You can find the option under `Skills` > `Dev Skills` > `{Your Skill}` > `Settings`

![Alexa User Permission Settings](img/alexa-skill-user-permission-settings.png)

That's it, you made it to the end!

**Any questions? Please let us know in the comments below 👇. You can also reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how use Alexa's Proactive Events API with Jovo", "author": "kaan-kilic", "tags": "Amazon Alexa", "og-image": "https://www.jovo.tech/img/tutorials/alexa-notifications-proactive-events/alexa-proactive-events-notifications.jpg"  }-->