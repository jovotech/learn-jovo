# Introducing Jovo v4: The React for Voice and Chat

![Jovo v4](./img/jovo-v4.png 'Jovo launches version 4')

After 12 months in the making, it's finally here! Jovo `v4` improves every step of the workflow, including a new browser-based Debugger, a new CLI, an output template engine, a component-based system, and much more. Learn more below.

## Introduction

When we [launched Jovo v3](https://www.context-first.com/introducing-jovo-v3-the-voice-layer/) in early 2020, it was a big step towards our vision of enabling companies to build great multimodal experiences that work across devices and platforms. We called Jovo the _Voice Layer_ and added support for voice on platforms like Bixby, the web, and more. Later in 2020, we launched improved support for [Jovo for Web](https://v3.jovo.tech/news/2020-10-29-jovo-for-web-v3-2),

While `v3` was all about expansion (more platforms, more integrations), it was time for us to improve the overall developer experience. Our goal for `v4` was to enable professional teams to build great, robust apps for voice and chat.

## Jovo v4 Features

- [New Debugger](#new-debugger): An all-new Debugger
- [Components](#components): Reusable, isolated elements for a better project structure
- [Output](#output): A multimodal template engine that works across platforms
- [CLI](#cli): Rebuilt from the ground up, including
- [More](#more): Lots of more improvements and updates

### New Debugger

> [Learn more about the Jovo Debugger here](https://www.jovo.tech/docs/debugger).

The new Jovo Debugger comes with lots of improvements:

- Test your app by typing or pressing buttons
- See the whole data flow in the _Lifecycle_ view
- Customize the Debugger with a [`jovo.debugger.js` file](https://www.jovo.tech/docs/debugger-config)

### Components

> [Learn more about components here](https://www.jovo.tech/docs/components).

Jovo `v4` has a component-based system at its core, similar to popular web frameworks like React and Vue. This makes it possible for our users to build reusable elements that can be used across projects.

### Output

> [Learn more about output here](https://www.jovo.tech/docs/output).

Jovo `v4` comes with a first of its kind multimodal template engine that turns [structured output](https://www.jovo.tech/docs/output-templates) into native platform responses.

For example, the output below gets translated into the appropriate format for all platforms that you can find in the [Jovo Marketplace](https://www.jovo.tech/marketplace):

```js
{
  message: 'Do you like pizza?',
  quickReplies: ['yes', 'no']
}
```

It comes with the following features:

- It makes it easier to separate content from logic
- Multimodal: Works for voice, chat, and visual output
- Works across platforms, even on Alexa APL
- Supports validation and automatically trims content based on platform requirements

### CLI

> [Learn more about the Jovo CLI here](https://www.jovo.tech/docs/cli).

The CLI was completely rebuilt from the ground up:

- A new, more modular architecture
- An improved build and deployment process
- Build your own [CLI plugins](https://www.jovo.tech/docs/cli-plugins)

### More

- [Unit Testing](https://www.jovo.tech/docs/unit-testing)
- [New Website](https://www.jovo.tech): Better docs and better search

## Getting Started with v4

There are several ways how you can get started with `v4`:

- Follow our [getting started guide](https://www.jovo.tech/docs/getting-started) to install the Jovo CLI and create a new project.
- Coming from Jovo `v3`? Follow our [migration guide](https://www.jovo.tech/docs/migration-from-v3).
- Take a look at our `v4` template for [TypeScript](https://github.com/jovotech/jovo-v4-template) and [JavaScript](https://github.com/jovotech/jovo-v4-template-js).
