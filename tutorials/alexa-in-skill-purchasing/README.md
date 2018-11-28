# Use Alexa In-Skill-Purchasing (ISP) with Jovo
 
In this tutorial, you will learn how to sell digital goods with your Alexa Skill by using [Alexa's in-skill purchasing (ISP)](https://developer.amazon.com/alexa-skills-kit/make-money/in-skill-purchasing) with Jovo.

📋 You can find the full code example of this tutorial here: [jovo-templates/alexa/isp](https://github.com/jovotech/jovo-templates/tree/master/alexa/isp).

### Contents

* [Introduction to Alexa In-Skill Purchasing](#introduction-to-alexa-in-skill-purchasing)
* [Implementing Alexa ISP](#implementing-alexa-isp)
    * [Add Products with ASK CLI](#add-products-with-ask-cli)
    * [Update your Language Model](#update-your-language-model)
    * [Implement ISP in your Code](#implement-isp-in-your-code)
        * [Upsell](#upsell)
        * [Purchase Request](#purchase-request)
        * [Refund](#refund)
        * [ON_PURCHASE](#onpurchase)
    * [Inventory Management](#inventory-management)


## Introduction to Alexa In-Skill Purchasing

In May 2018, Amazon [introduced](https://developer.amazon.com/blogs/alexa/post/5d852c9c-8cdf-45c1-9b68-e2f02af26c89/make-money-with-alexa-skills) the ability for Skill developers to make money through in-skill purchasing. It allows you to sell premium content either through one-time purchases, consumables or subscriptions. 

The Alexa ISP API works independently from your own skill. You do not handle the transactions yourself, but you send a directive just like with the Dialog Interface. Alexa will use predefined values from your products `json` file, which we will create in a minute, to fulfill the transaction. The moment you send out one of these directives the current session ends. Once the transaction is finished, your Skill will get a request with data about the transaction at which point you can resume where the user left off.   

## Implementing Alexa ISP

The Alexa in-skill purchases API offers three kinds of interactions with our user:

* **Upsell**: we proactively offer the user our products
* **Purchase request**: the user asks to buy something
* **Refund**: the user wants to return a product.

Before we can implement the interactions in our code base, we have to first create the items we want to sell and implement the respective intents in our language model.

![Alexa In-Skill-Purchasing](./img/workflow-in-skill-purchases.png)

* [Add Products with the ASK CLI](#add-products-with-the-ask-cli)
* [Update the Language Model](#update-the-language-model)
* [Implement ISP in your Code](#implement-isp-in-your-code)

### Add Products with ASK CLI

To add items, we need the ASK CLI (version `1.4.3` or later). If you did not install or initialize the ASK CLI yet, check out this [quickstart guide](https://developer.amazon.com/docs/smapi/quick-start-alexa-skills-kit-command-line-interface.html#step-2-install-and-initialize-ask-cli).

To run the ASK CLI's commands we have to move from the projects root folder to the `alexaSkill` folder:

```text
$ cd platforms/alexaSkill
```

We're going to add an entitlement, which is a one-time purchase item, and call it `frozen_sword` using the `ask add isp` command:

```text
$ ask add isp

? List of in-skill product types you can choose (Use arrow keys)
❯ Entitlement 
  Subscription 

? List of in-skill product templates you can choose (Use arrow keys)
> Entitlement_Template

? Please type in your new in-skill product name:
 frozen_sword
In-skill product frozen_sword.json is saved to ./isps/entitlement/frozen_sword.json
```

After that, there should be a `frozen_sword.json` file in your `alexaSkill/isps` folder. The content of that `json` file is used to define the price, release date, description as well as the prompts Alexa will use when she handles the transaction and much much more. You can find examples and a small description for each field [here](https://developer.amazon.com/docs/smapi/isp-schemas.html).

Here's how our product's file looks like:

```javascript
{
  "version": "1.0",
  "type": "ENTITLEMENT",
  "referenceName": "frozen_sword",
  "publishingInformation": {
    "locales": {
      "en-US": {
        "name": "Frozen Sword",
        "smallIconUri": "https://s3.amazonaws.com/jovocards/logo108.png",
        "largeIconUri": "https://s3.amazonaws.com/jovocards/logo512.png",
        "summary": "A sword once used by Arthas.",
        "description": "Use the overpowered Frozen Sword",
        "examplePhrases": [
          "Alexa, buy the frozen sword"
        ],
        "keywords": [
          "frozen sword",
          "sword",
          "frozen"
        ],
        "customProductPrompts": {
          "purchasePromptDescription": "Do you want to buy the Frozen Sword?",
          "boughtCardDescription": "Congrats. You successfully purchased the Frozen Sword!"
        }
      }
    },
    "distributionCountries": [
      "US"
    ],
    "pricing": {
      "amazon.com": {
        "releaseDate": "2018-05-14",
        "defaultPriceListing": {
          "price": 0.99,
          "currency": "USD"
        }
      }
    },
    "taxInformation": {
      "category": "SOFTWARE"
    }
  },
  "privacyAndCompliance": {
    "locales": {
      "en-US": {
        "privacyPolicyUrl": "https://www.yourcompany.com/privacy-policy"
      }
    }
  },
  "testingInstructions": "This is an example product.",
  "purchasableState": "PURCHASABLE"
}
```

Additionally, we will add a subscription, so run the `ask add isp` command again and add the `premium_pass`:

```text
$ ask add isp

? List of in-skill product types you can choose (Use arrow keys)
  Entitlement 
❯ Subscription 

? List of in-skill product templates you can choose (Use arrow keys)
> Subscription_Template

? Please type in your new in-skill product name:
 premium_pass
In-skill product premium_pass is saved to ./isps/subscription/premium_pass.json
```

Here's the template for the subscription:

```javascript
{
  "version": "1.0",
  "type": "SUBSCRIPTION",
  "referenceName": "premium_pass",
  "subscriptionInformation": {
    "subscriptionPaymentFrequency": "MONTHLY",
    "subscriptionTrialPeriodDays": 30
  },
  "publishingInformation": {
    "locales": {
      "en-US": {
        "name": "Premium Pass",
        "smallIconUri": "https://s3.amazonaws.com/jovocards/logo108.png",
        "largeIconUri": "https://s3.amazonaws.com/jovocards/logo512.png",
        "summary": "Get access to all the premium content",
        "description": "Get content updates two weeks earlier than everbody else",
        "examplePhrases": [
          "Alexa, buy premium pass"
        ],
        "keywords": [
          "premium pass",
          "pass",
          "premium"
        ],
        "customProductPrompts": {
          "purchasePromptDescription": "Do you want to buy the premium pass?",
          "boughtCardDescription": "Congrats. You successfully purchased the premium pass"
        }
      }
    },
    "distributionCountries": [
      "US"
    ],
    "pricing": {
      "amazon.com": {
        "releaseDate": "2018-05-14",
        "defaultPriceListing": {
          "price": 0.99,
          "currency": "USD"
        }
      }
    },
    "taxInformation": {
      "category": "SOFTWARE"
    }
  },
  "privacyAndCompliance": {
    "locales": {
      "en-US": {
        "privacyPolicyUrl": "https://www.yourcompany.com/privacy-policy"
      }
    }
  },
  "testingInstructions": "This is an example product.",
  "purchasableState": "PURCHASABLE"
}
```

Since the file structure for a `consumable` is the same as for a `entitlement`, we will skip adding that.

Save the files and deploy everything with the ASK CLI:

```text
$ ask deploy
```

In the next step, we have to update the language model.

### Update your Language Model

As I described earlier, there are three kinds of interactions we have to implement in our Skill, but only two of these need to be added to the language model because the buy and refund request are the only ones initiated by the user.

All of the elements, that we're going to add now, should be placed inside in the `alexa` object of our Jovo Language Model (`models/en-US.json`) to keep everything as clean as possible. 

First of all, we will add a custom slot type for our product names. The important thing here is that the `id` of each value is the `reference_name` of our product, because we need it later on to initiate the transaction.


```javascript
// rest of the language model
"alexa": {
    "interactionModel": {
        "languageModel": {
            "intents": [
                /*
                 * intents
                 */
            ],
            "types": [
                {
                    "name": "PRODUCT_NAMES",
                    "values": [
                        {
                            "id": "frozen_sword",
                            "name": {
                                "value": "frozen sword"
                            }
                        },
                        {
                            "id": "premium_pass",
                            "name": {
                                "value": "premium pass"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
```


Next up the intents. Amazon provides sample intents for both buy and refund requests.:

```javascript
// rest of the language model
"alexa": {
    "interactionModel": {
        "languageModel": {
            "intents": [
                /*
                 * other intents
                 */
                {
                    "name": "RefundSkillItemIntent",
                    "samples": [
                        "return {ProductName}",
                        "refund {ProductName}",
                        "want a refund for {ProductName}",
                        "would like to return {ProductName}"
                    ],
                    "slots": [
                        {
                            "name": "ProductName",
                            "type": "PRODUCT_NAMES"
                        }
                    ]
                },
                {
                    "name": "BuySkillItemIntent",
                    "samples": [
                        "buy",
                        "shop",
                        "buy {ProductName}",
                        "purchase {ProductName}",
                        "want {ProductName}",
                        "would like {ProductName}"
                    ],
                    "slots": [
                        {
                            "name": "ProductName",
                            "type": "PRODUCT_NAMES"
                        }
                    ]
                }
            ],
            "types": [
                {
                    "name": "PRODUCT_NAMES",
                    "values": [
                        {
                            "id": "frozen_sword",
                            "name": {
                                "value": "frozen sword"
                            }
                        },
                        {
                            "id": "premium_pass",
                            "name": {
                                "value": "premium pass"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
```

That's everything we need for our language model. We just have to run the `jovo build --deploy` command to upload our changes to the Amazon Developer Console and after that, we can begin to implement our code.

### Implement ISP in your Code

As we discussed earlier, our only task is to **start** the transaction, since Alexa will handle the transaction herself. After the transaction finished, we will receive a request notifying us about the outcome, i.e. the purchase was successful or not. Inside that request, there will also be a `token`, which we can define while starting the transaction, to help us resume the Skill at the place the user left off.

To access the Alexa ISP API in Jovo we use `this.$alexaSkill.inSkillPurchase()`.

Before we start any kind of transaction or refund process, we always retrieve the product first:

```javascript
this.$alexaSkill
    .inSkillPurchase()
    .getProductByReferenceName(productReferenceName, (error, product) => {

    });
```

The data we get looks like this:
```javascript
{ productId: 'amzn1.adg.product.994e8ec0-2f73-4130-837a-0bb06abe1ded',
  referenceName: 'frozen_sword',
  type: 'ENTITLEMENT',
  name: 'Frozen Sword',
  summary: 'A sword once used by Arthas.',
  entitled: 'NOT_ENTITLED',
  entitlementReason: 'NOT_PURCHASED',
  purchasable: 'PURCHASABLE',
  activeEntitlementCount: 0,
  purchaseMode: 'TEST' 
}
```

Using that data we can check if the user already owns the product:

```javascript
this.$alexaSkill
    .inSkillPurchase()
    .getProductByReferenceName(productReferenceName, (error, product) => {
        if (error) {
            console.log(error);
        }
        if (product.entitled === 'ENTITLED') {
            // user already owns it
        } else {
            // user does not own it
        }
    });
```

#### Upsell

The upsell method is used to proactively offer the user our products. To send out the request, we need three things:

* the reference name of the product
* a message/prompt
* a token

For simplicity reasons and because these things are different for every individual skill, we will just hard-code the reference name and use a placeholder token.

For your own skill, you should try to build a system, which offers each product at the right time, e.g. offer the user extra lives after they have died, so you don't annoy your user.

For the message/prompt, Amazon offers the following guidelines:

* Don't include the price. The products price, subscription term or free trial length are obtained from the product's `json` file during the transaction.
* End the message with a question to determine if the user is interested, but don't specifically ask if they want to buy it. That will be handled in the transaction by Alexa.
* Explain the benefit the user would have from that product at the current situation.

In general, you should also avoid suggesting multiple products in a row or suggestion products too often, so you don't interrupt and annoy the user.

Alright, now that we discussed all that, we can start the transaction using the  `upsell()` method, which takes the `productId`, `prompt` and `token` as parameters, after we've checked if the user already owns the product:

```javascript
UpsellIntent() {
    let productReferenceName = 'frozen_sword';
    this.$alexaSkill
    .inSkillPurchase()
    .getProductByReferenceName(productReferenceName, (error, product) => {
        if (error) {
            console.log(error);
        }
        if (product.entitled === 'ENTITLED') {
            this.tell('You have already bought this item.');
            return;
        } else {
            let prompt = 'The frozen sword will help you on your journey. Are you interested?';
            let token = 'testToken';
            this.$alexaSkill.inSkillPurchase().upsell(product.productId, prompt, token);
        }
    });
},
```

#### Purchase Request

Now we can get to the first user requested intent, `BuySkillItemIntent`. 

There are two scenarios at which the intent gets invoked. In the first one, the user does not specify which item they want to buy, i.e. there is no value for our input variable, where we just suggest them some.

```javascript
BuySkillItemIntent() {
    if (!this.$inputs.productName) {
        this.ask('You can choose either the premium pass. or frozen sword. Which are you interested in?');
    }
}
```

In the other scenario the user specifies the product and we use the input's id as our product's `reference_name` to check whether they already own it. If that's not the case, we start the transaction:

```javascript
BuySkillItemIntent() {
    if (!this.$inputs.productName) {
        this.ask('You can choose either the premium pass. or frozen sword. Which are you interested in?');
    }
    let productReferenceName = productName.id;
    this.$alexaSkill
        .inSkillPurchase()
        .getProductByReferenceName(productReferenceName, (error, product) => {
            if (error) {
                console.log(error);
            }
            if (product.entitled === 'ENTITLED') {
                this.tell('You have already bought this item.');
                return;
            }
            let token = 'testToken';
            this.$alexaSkill.inSkillPurchase().buy(product.productId, token);
        });
},
```

#### Refund

Next, the `RefundSkillItemIntent`. Same procedure, get the input's id, use it to get the product, check if the user even owns the product and start the refund process:

```javascript
RefundSkillItemIntent() {
    let productReferenceName = this.$inputs.productName.id;
    this.$alexaSkill
        .inSkillPurchase()
        .getProductByReferenceName(productReferenceName, (error, product) => {
            if (error) {
                console.log(error);
                // Continue, where you left off
            }
            if (product.entitled !== 'ENTITLED') {
                this.tell('You have not bought this item yet.');
            }
            let token = 'testToken';
            this.$alexaSkill.inSkillPurchase().cancel(product.productId, token);
        });
},
```

#### ON_PURCHASE

As we discussed earlier, the initial session ends after we start a transaction (`upsell`, `buy` or `refund`). After it's finished your Skill will get another request, which will looks like this for example:

```javascript
{
    "type": "Connections.Response",
    "requestId": "string",
    "timestamp": "string",
    "name": "Upsell",
    "status": {
        "code": "string",
        "message": "string"  
    },
    "payload": {
        "purchaseResult":"ACCEPTED",    
        "productId":"string",   
        "message":"optional additional message"
    },
    "token": "string"
}
```

The important parts of that request are:

Name | Description
:--- | :---
`name` | Either `Upsell`, `Buy` or `Refund`. Used to determine which kind of transaction took place
`payload.purchaseResult` | Either `ACCEPTED`, `DECLINED`, `ALREADY_PURCHASED` or `ERROR`. Used to determine the outcome of the transaction
`payload.productId` | The product in question
`token` | The token used to resume the skill where it left off

The incoming request will be mapped to the Jovo built-in `ON_PURCHASE` intent, where you can resume the Skill at the point before the transaction. The `token` you can send is supposed to help you with that.

```javascript
ON_PURCHASE() {
    const name = this.$request.name;
    const productId = this.$alexaSkill.inSkillPurchase().getProductId();
    const purchaseResult = this.$alexaSkill.inSkillPurchase().getPurchaseResult();
    const token = this.$request.token;

    if (purchaseResult === 'ACCEPTED') {
        this.tell('Great! Let\'s use your new item');
    } else {
        this.tell('Okay. Let\'s continue where you left off.');
    }
},
```

### Inventory Management

With consumables, you have to maintain one more thing, the user's inventory. While Alexa keeps track of the number of purchases per item, `activeEntitlementCount`, which gets incremented with every purchase and decremented with every refund, you have to track how many times the user has used the consumable. Simply storing the data inside a database would be enough in this case.

Because you are offering consumables, the `userId` of your user stays the same even if they disable and re-enable the skill later on, so keep that in mind if you clean up the user data after they've disabled your skill using the [Alexa Skill Events](https://www.jovo.tech/tutorials/alexa-skill-events).

The incoming `AlexaSkillEvent.SkillDisabled` request will have the `userInformationPersistenceStatus` property that has either the value `PERSISTED` or `NOT_PERSISTED`, which you can use to decide, whether you can clean up the data or not.

That's the basic implementation of Alexa in-skill-purchasing (ISP) with Jovo.

**Any questions? You can reach us on [Twitter](https://twitter.com/jovotech) or [Slack](https://www.jovo.tech/slack).**

<!--[metadata]: { "description": "Learn how to make money with Alexa Skills by implementing the Alexa In-Skill-Purchasing (ISP) feature with Jovo", "author": "kaan-kilic", "tags": "Amazon Alexa"}-->
