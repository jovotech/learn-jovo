---
title: 'Introducing Jovo v4.2: React Web Widgets, Alexa AudioPlayer, Snips NLU Server, CLI'
excerpt: 'Learn more about the Jovo update that was released in January 2022.'
date: '2022-01-26'
---

# Jovo v4.2: React Web Widgets, Alexa AudioPlayer, Snips NLU Server, CLI

![Jovo v4.2](./img/jovo-v4-2.png 'Jovo launches version 4.2')

In November 2021, we [launched Jovo `v4`](https://www.jovo.tech/news/jovo-v4), which was our most exciting update so far. Jovo `v4.2` now builds on top of that and comes with many improvements and new integrations. Learn more below.

## Introduction

`v4.2` is all about building on top of the `v4` architecture and showing what can be built with it. The most notable features are:

- [React Web Widgets](#react-web-widgets): Build voice and chat web apps with React and Jovo.
- [Alexa AudioPlayer](#alexa-audioplayer): Build Alexa Skills that play long-form audio, like music and podcasts.
- [Open Source Snips NLU Server](#open-source-snips-nlu-server): Host an NLU server on your own premises.
- [CLI Improvements](#cli-improvements): Many improvements, including a newly added `update` command.
- [Jovo Logger](#breaking-changes): An improved logger. Learn more in the [breaking changes](#breaking-changes) section.

`v4.2` also comes with many smaller improvements and bugfixes. [You can find all merged pull requests here](https://github.com/jovotech/jovo-framework/pulls?q=is%3Apr+is%3Amerged).

A huge thank you for everyone who helped improve Jovo: [Julian](https://github.com/jtfell), [Dominik](https://github.com/dominik-meissner), [Barak](https://github.com/barakd), [Gareth](https://github.com/GarethBeddis), [Max](https://github.com/m-ripper), [Ruben](https://github.com/rubenaeg), and many more who are part of the Jovo community!

## React Web Widgets

We already [launched web starters for Vue.JS](https://www.producthunt.com/posts/jovo-for-web) a while ago. Now, you can also build voice and chat web apps using Jovo and React!

Right now, we offer starters for a web chat widget and a standalone voice experience.

![Jovo React Chat Widget](https://github.com/jovotech/jovo-starter-web-chatwidget-react/raw/main/img/starter-web-chatwidget.gif 'Animation that shows a chat widget at the bottom right that opens if you click it.')

You can find the chat widget starter template here: https://github.com/jovotech/jovo-starter-web-chatwidget-react

![Jovo React Standalone Voice Experience](https://github.com/jovotech/jovo-starter-web-standalone-react/raw/v4/img/starter-web-standalone.gif 'Animation that shows a standalone voice experience with a microphone button at the bottom center')

You can find the standalone voice experience starter template here: https://github.com/jovotech/jovo-starter-web-standalone-react

Thank you [Max](https://github.com/m-ripper) for building these!

## Alexa AudioPlayer

We now support the ability to build Alexa AudioPlayer Skills for podcasts, music, and anything else that features long-form audio. [Learn more in our docs](https://www.jovo.tech/marketplace/platform-alexa/audioplayer).

It features helper [output classes](https://www.jovo.tech/marketplace/platform-alexa/audioplayer#send-audioplayer-responses), [`@Handle` decorators](https://www.jovo.tech/marketplace/platform-alexa/audioplayer#audioplayer-handlers), and a [`this.$alexa.audioPlayer` property](https://www.jovo.tech/marketplace/platform-alexa/audioplayer#audioplayer-property) that makes it easier to access request specific information.

```typescript
import { AudioPlayerPlayOutput } from '@jovotech/platform-alexa';
// ...

someHandler() {
  // ...

  return this.$send(AudioPlayerPlayOutput, {
    message: 'Starting audio',
    audioItem: {
      stream: {
        url: 'https://s3.amazonaws.com/jovo-songs/song1.mp3',
      },
    },
  })
}
```

Thank you [Alex](https://github.com/aswetlow) for building this!

## Open Source Snips NLU Server

We have been offering an [integration for Snips NLU](https://www.jovo.tech/marketplace/nlu-snips), an open source [natural language understanding service](https://www.jovo.tech/docs/nlu), since the release of `v4.0`.

With this update, we're excited to make it even easier to build with Jovo and Snips NLU. We just open sourced a Python based server that implements Snips NLU and even the ability to train models dynamically based on our [dynamic entities feature for Snips](https://www.jovo.tech/marketplace/nlu-snips#dynamic-entities).

You can find the server on GitHub (leave a star if you like it!): https://github.com/jovotech/snips-nlu-server

Thank you [Ruben](https://github.com/rubenaeg) for building this!

## CLI Improvements

The CLI was bumped to `v4.1` and comes with many improvements:

- An improved configuration for the building process of `models`. [You can find the documentation here](https://www.jovo.tech/docs/project-config#models).
- A better user experience with hints that suggest next steps based on the used commands.
- A new command: [`jovo update`](https://www.jovo.tech/docs/update-command).

Thank you [Ruben](https://github.com/rubenaeg) for the updates!

## Getting Started with Jovo

There are several ways how you can get started with Jovo:

- Follow our [getting started guide](https://www.jovo.tech/docs/getting-started) to install the Jovo CLI and create a new project.
- Coming from Jovo `v3`? Follow our [migration guide](https://www.jovo.tech/docs/migration-from-v3).
- Take a look at our `v4` template for [TypeScript](https://github.com/jovotech/jovo-v4-template) and [JavaScript](https://github.com/jovotech/jovo-v4-template-js).

## Breaking Changes

We refactored the [Jovo Logger](https://www.jovo.tech/docs/logging#jovo-logger) to use `loglevel` instead of `tslog`. [You can learn more in this PR](https://github.com/jovotech/jovo-framework/pull/1190).

The CLI improvements for the [Jovo Model project configuration](https://www.jovo.tech/docs/project-config#models) also comes with a breaking change if you've been using the `modelsDirectory` property (default is `models`):

```javascript
// Before
const project = new ProjectConfig({
  modelsDirectory: 'models',
  // ...
});

// After
const project = new ProjectConfig({
  models: {
    enabled: true,
    directory: 'models',
    // ...
  },
  // ...
});
```
