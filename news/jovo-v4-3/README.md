---
title: 'Introducing Jovo v4.3: Polly TTS, MongoDB, NLU Performance'
excerpt: 'Learn more about the Jovo update that was released in September 2022.'
date: '2022-09-07'
---

# Jovo v4.3: Polly TTS, MongoDB, NLU Performance

![Jovo v4.3](./img/jovo-v4-3.jpg 'Jovo launches version 4.3')

Jovo `v4.3` builds on top of our [major `v4` release last year](https://www.jovo.tech/news/jovo-v4) and includes many improvements, new integrations, and community plugins. Learn more below.

## Introduction

Since our [`v4.2` release](https://www.jovo.tech/news/jovo-v4-2), there were lots of core features added to the Jovo Framework, as well as integrations and plugins to the [Jovo Marketplace](https://www.jovo.tech/marketplace), for example:

- [TTS](#tts) features to generate speech audio files from text output.
- [MongoDB integration](https://www.jovo.tech/marketplace/db-mongodb) to store user data.
- [Keyword NLU Plugin](https://www.jovo.tech/marketplace/plugin-keywordnlu) that allows you to skip NLU requests for certain keywords
- [Sanity CMS integration](https://www.jovo.tech/marketplace/cms-sanity) for content management

Learn more about all additions below: [New Integrations and Plugins](#new-integrations-and-plugins).

`v4.3` also comes with many smaller improvements and bugfixes. You can find all releases [on GitHub](https://github.com/jovotech/jovo-framework/releases) and the [Jovo News page](https://www.jovo.tech/news).

A huge thank you for everyone who keeps helping to improve Jovo: [Mark](https://github.com/rmtuckerphx), [Ben](https://github.com/theBenForce), [Florian](https://github.com/VialFlorian), [Gianluca](https://github.com/acerbisgianluca), [David](https://github.com/DavidKrell), [Fabian](https://github.com/bayont), [Itay](https://github.com/Itay212121), [Joshua](https://github.com/sadlowskij), [Ruben](https://github.com/rubenaeg), [Program Productions](https://github.com/Programproductions), [Tal](https://github.com/weisslertal), [Stephane](https://github.com/kouz75), [Frank](https://github.com/fboerncke), [Jorge](https://github.com/jrglg) and many more who are part of the Jovo [community](https://www.jovo.tech/community)!



## New Integrations and Plugins

![Jovo Marketplace](./img/jovo-marketplace.jpg 'Screenshot of the Jovo Marketplace, showing a list of available integrations')

A variety of integrations and plugins was added to the [Jovo Marketplace](https://www.jovo.tech/marketplace):

- [TTS](#tts) features to generate speech audio files from text output.
- [Output](#output) plugins that help you craft, convert, and manage content
- [Games](#games) plugins that help you build voice and chat games with Jovo
- [MongoDB Integration](https://www.jovo.tech/marketplace/db-mongodb) to store user data
- [Keyword NLU Plugin](https://www.jovo.tech/marketplace/plugin-keywordnlu) that allows you to skip NLU requests for certain keywords
- [Community Tools plugin](https://www.jovo.tech/marketplace/community-plugin-tools) with helpful features for all types of Jovo projects
- [TimeZone Plugin](https://www.jovo.tech/marketplace/plugin-timezone) as a convenient way to retrieve the user's time zone across platforms
- [VS Code Extension](https://www.jovo.tech/marketplace/jovo-snippets-vscode) with snippets to speed up Jovo development

Thanks a lot to [Mark](https://github.com/rmtuckerphx), [Joshua](https://github.com/sadlowskij), [Frank](https://github.com/fboerncke) and [Jorge](https://github.com/jrglg) for adding the community plugins and integrations!

### TTS

A major new feature is the ability to turn `text` from your output templates into text to speech (TTS) audio files and even store them on a server or cloud provider. Learn more in the [TTS documentation](https://www.jovo.tech/docs/tts).

- [Polly TTS](https://www.jovo.tech/marketplace/tts-polly) to turn text into audio
- [S3 Cache](https://www.jovo.tech/marketplace/ttscache-s3) to store the audio on Amazon S3

Thanks a lot to [Mark](https://github.com/rmtuckerphx) for adding this core feature and the integrations!

### Output

Jovo `v4.3` also comes with integrations that help you prepare and manage output:

- [Sanity CMS Integration](https://www.jovo.tech/marketplace/cms-sanity) to manage content in Sanity
- [Spintax Plugin](https://www.jovo.tech/marketplace/plugin-spintax-output) to use Spintax expressions in your output templates
- [Jexl Plugin](https://www.jovo.tech/marketplace/plugin-jexl-output) to use Jexl expressions in your output templates

Thanks a lot to [Mark](https://github.com/rmtuckerphx) and [Frank](https://github.com/fboerncke) for adding these community plugins and integrations!


### Games

Thanks to [Mark](https://github.com/rmtuckerphx), there are now a variety of plugins that help you build games using Jovo:

- [PlayFab Plugin](https://www.jovo.tech/marketplace/plugin-playfab) to integrate your Jovo app with the PlayFab platform
- [Player Generator Plugin](https://www.jovo.tech/marketplace/plugin-playergenerator) to generate random player names and avatars for your users
- [Badgerific Plugin](https://www.jovo.tech/marketplace/plugin-badgerific) to add badges and achievements to your games


## Breaking Changes

This release does not come with any breaking changes. Happy upgrading!

## Getting Started with Jovo

There are several ways how you can get started with Jovo:

- Follow our [getting started guide](https://www.jovo.tech/docs/getting-started) to install the Jovo CLI and create a new project
- Take a look at out [examples](https://www.jovo.tech/examples)
- Join our [community](https://www.jovo.tech/community)

Thanks a lot for being part of Jovo!
