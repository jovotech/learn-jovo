---
title: 'Introducing Jovo v4.3: Polly TTS, MongoDB, NLU Performance'
excerpt: 'Learn more about the Jovo update that was released in September 2022.'
date: '2022-09-07'
---

# Jovo v4.5: Dependency Injection, Intent Scoping

![Jovo v4.5](https://www.jovo.tech/img/jovo-v4.png 'Jovo launches version 4.5')

Jovo `v4.5` builds on top of our [major `v4` release last year](https://www.jovo.tech/news/jovo-v4). Learn more below.


## Introduction

Jovo `v4.5` includes the following larger updates:

- [New Features](#new-features)
  - [Dependency Injection](#dependency-injection)
  - [Intent Scoping](#intent-scoping)
  - [Native NLU Data](#native-nlu-data)
  - [Component Inheritance](#component-inheritance)
- [New Examples](#new-examples)

`v4.5` also comes with many other improvements and bugfixes. You can find all releases [on GitHub](https://github.com/jovotech/jovo-framework/releases) and the [Jovo News page](https://www.jovo.tech/news).

A huge thank you for everyone who keeps helping to improve Jovo: [Mark](https://github.com/rmtuckerphx), [Palle](https://github.com/palle-k), [Nico](https://github.com/NLueg), [Florian](https://github.com/VialFlorian), [David](https://github.com/DavidKrell), [Pete](https://github.com/Programproductions), [Gianluca](https://github.com/acerbisgianluca), and many more who are part of the Jovo [community](https://www.jovo.tech/community)!


## New Features

The `v4.5` release comes with many improvements and 4 larger additions:

- [Dependency Injection](#dependency-injection)
- [Intent Scoping](#intent-scoping)
- [Native NLU Data](#native-nlu-data)
- [Component Inheritance](#component-inheritance)

Here are some more pull requests that are part of latest releases:

- [Add billingMode option on DynamoDbConfig to allow on-demand mode](https://github.com/jovotech/jovo-framework/pull/1437) by [Florian](https://github.com/VialFlorian)
- [Allow history items as class instances](https://github.com/jovotech/jovo-framework/pull/1435) by [Alex](https://github.com/aswetlow)
- [Add app data + allow app and request data in unit tests](https://github.com/jovotech/jovo-framework/pull/1421) by [Jan](https://github.com/jankoenig)
- [Add Alexa Lists feature](https://github.com/jovotech/jovo-framework/pull/1418) by [Nico](https://github.com/NLueg)


### Dependency Injection

[Dependency injection](https://www.jovo.tech/docs/service-providers-dependency-injection) is a feature that can be helpful to access instantiated service providers (and values) in components and output classes:

```typescript
// src/services/OrderService.ts

import { Injectable } from '@jovotech/framework';
// ...

@Injectable()
class OrderService {
  async performOrder() {
    // ...
  }
}
```

They can be added to the app config:

```typescript
// src/app.ts

import { OrderService } from './services/OrderService';
// ...

const app = new App({
  providers: [
    OrderService,
    // ...
  ],
});
```

And then accessed in any component our output class using the `constructor()`:

```typescript
// src/components/OrderPizzaComponent.ts

@Component()
class OrderPizzaComponent extends BaseComponent {
  constructor(
    jovo: Jovo,
    options: UnknownObject | undefined,
    private readonly orderService: OrderService
  ) {
    super(jovo, options);
  }

  @Intents('ConfirmOrderIntent')
  async confirmOrder() {
    try {
      await this.orderService.performOrder();
      // ...
    } catch (e) {
      // ...
    }
  }
}
```

This feature is especially helpful if you want to mock API calls in [unit testing](https://www.jovo.tech/docs/service-providers-dependency-injection#unit-testing).

Thank you [Palle](https://github.com/palle-k) for contributing this great feature!


### Intent Scoping

As part of our goal to make it easier for you to improve [NLU performance](https://www.jovo.tech/docs/nlu#performance), we added a feature called intent scoping.

Some NLU services support the ability to prioritize certain intents. This helps if you have a large language model, but want to listen to specific input.

In Jovo, you can now pass a list of intents using the [`listen` output property](https://www.jovo.tech/docs/output-templates#listen):

```typescript
{
  message: `Which city do you want to visit?`,
  listen: {
    intents: [ 'CityIntent' ],
  },
}
```

This feature currently works with [Snips NLU](https://www.jovo.tech/marketplace/nlu-snips).


### Native NLU Data

For more transparency and easier debugging, we added a `native` property to the [NLU](https://www.jovo.tech/docs/nlu) data that gets added to the [`$input` property](https://www.jovo.tech/docs/input). It includes the full API response from the NLU service:

```typescript
{
  type: 'TEXT',
  text: 'My name is Max',
  nlu: {
    intent: 'MyNameIsIntent',
    entities: {
      name: {
        value: 'Max',
        // ...
      },
    },
    native: {
      // Raw API response from the NLU service
    },
  },
}
```

### Component Inheritance

Components can now [inherit](https://www.jovo.tech/docs/components#inheritance) handlers from their superclass. This is useful for components that offer similar workflows, like a help handler.

```typescript
import { BaseComponent } from '@jovotech/framework';

abstract class ComponentWithHelp extends BaseComponent {
  abstract showHelp(): Promise<void>;
  
  async repeatLastResponse() {
    // ...
  }
  
  @Intents('HelpIntent')
  async help() {
    await this.showHelp();
    await this.repeatLastResponse();
  }
}

@Component()
class YourComponent extends ComponentWithHelp {
  async showHelp() {
    // ...
  }
}
```

Thank you [Palle](https://github.com/palle-k) for contributing this great feature!



## New Examples

We also added 2 [Jovo examples](https://www.jovo.tech/examples): 

- [Terraform Template](https://github.com/Programproductions/lambda-alexa-jovo-stages) by [Pete Whelan](https://github.com/Programproductions)
- [Custom Platform Example](https://www.youtube.com/watch?v=p7WI5-Yyyxw) by [Mark Tucker](https://github.com/rmtuckerphx)


## Breaking Changes

The [dependency injection](#dependency-injection) feature comes with a small breaking change.

It changes the constructor signature for `BaseComponent` and `BaseOutput`. By changing from `options?: TYPE` to `options: undefined | TYPE`, subsequent parameters are not required to be optional.

In your `constructor()`, make sure to add `undefined` as a potential value for `options`, for example:

```typescript
@Component()
class OrderPizzaComponent extends BaseComponent {
  constructor(
    jovo: Jovo,
    options: ComponentOptions<UnknownObject> | undefined,
    private readonly orderService: OrderService
  ) {
    super(jovo, options);
  }

  @Intents('ConfirmOrderIntent')
  async confirmOrder() {
    try {
      await this.orderService.performOrder();
      // ...
    } catch (e) {
      // ...
    }
  }
}
```

Learn more in its [pull request](https://github.com/jovotech/jovo-framework/pull/1461).

Also, in [`v4.4`](https://github.com/jovotech/jovo-framework/releases/tag/2022-11-07-minor), we dropped support for Node `v12`.


## Getting Started with Jovo

There are several ways how you can get started with Jovo:

- Follow our [getting started guide](https://www.jovo.tech/docs/getting-started) to install the Jovo CLI and create a new project
- Take a look at our [examples](https://www.jovo.tech/examples)
- Join our [community](https://www.jovo.tech/community)

Thanks a lot for being part of Jovo!
