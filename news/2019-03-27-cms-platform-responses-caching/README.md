# New Jovo CMS Features: Platform-Specific Responses and Instant Updates

After launching a new [Jovo CMS integration for Airtable](https://www.jovo.tech/news/2019-03-12-airtable-cms-integration) two weeks ago, we're back with some new updates for the [Jovo CMS integrations](https://www.jovo.tech/docs/cms):

* [Platform-Specific Responses](#platform-specific-responses)
* [Instant Updates](#instant-updates)
* [How to Update](#how-to-update)


*Like what we're doing? [Support us with a star on GitHub](https://github.com/jovotech/jovo-framework/)* 


## Platform-Specific Responses

> [Learn more about platform-specific responses here](https://www.jovo.tech/docs/output/i18n#platform-specific-responses).

When building cross-platform voice apps, it can happen often that the desired response output for an Alexa Skill or a Google Action differs.

An Alexa Skill might want to welcome a new user like "*Welcome to the XYZ Alexa Skill*", where a Google Action might say "*Welcome to the XYZ Google Action*" or even something completely different.

The Jovo [platform-specific responses](https://www.jovo.tech/docs/output/i18n#platform-specific-responses) feature allows you to manage all of this without changing anything in your logic (no `if/else` statements!), by adding `AlexaSkill` and/or `GoogleAction` namespaces to your language resources:

```js
{
    "translation": {
        "welcome": "Welcome.",
        "goodbye": "Goodbye."
    },
    "AlexaSkill": {
        "translation": {
            "goodbye": "Feel free to rate this skill, have a wonderful day."
        }
    },
    "GoogleAction": {
        "translation": {
            "goodbye": "/"
        }
    }
}
```
And you can use the same by adding new rows to your [Google Sheets](https://www.jovo.tech/docs/cms/google-sheets#platform-specific-responses) or [Airtable](https://www.jovo.tech/docs/cms/airtable#platform-specific-responses) content management systems.

This is how it looks like in a Google Sheet:

![Alexa Skill and Google Action Responses in a Google Sheet](https://www.jovo.tech/img/docs/v2/platform-specific-responses-sheets.jpg)


And this is in Airtable:

![Alexa Skill and Google Action Responses in Airtable](https://www.jovo.tech/img/docs/v2/platform-specific-responses-air.jpg)



## Instant Updates

> [Learn more about CMS caching here](https://www.jovo.tech/docs/cms#caching).

When working with a CMS, speed in data retrieval is an important feature. This is why Jovo only retrieves the content from Google Sheets or Airtable once when the app is started and then caches it.

Some community members [raised the issue](https://github.com/jovotech/jovo-framework/issues/394), however, that for testing purposes, it might be helpful to have instant updates of CMS content in the app. We listened and added a `caching` option to our CMS interface which can be set to `false`. This increases load times (data is retrieved with every request), but makes it possible to make and test adjustments (for example, SSML tweaks) quickly.

Learn how to disable CMS caching for:
* [Google Sheets](https://www.jovo.tech/docs/cms/google-sheets#caching)
* [Airtable](https://www.jovo.tech/docs/cms/airtable#caching)

## How to Update

> [Learn more in the Jovo Upgrading Guide](https://www.jovo.tech/docs/installation/upgrading).

To update to the latest version of Jovo, use the following command:

```sh
$ jovo update
```

**Any thoughts? Let us know in the comments below.**


<!--[metadata]: { "description": "New Features for Jovo CMS Integrations: Platform-specific responses and CMS Caching with Instant Updates. Released in March 2019.", "author": "jan-koenig", "tags": "Releases" }-->
