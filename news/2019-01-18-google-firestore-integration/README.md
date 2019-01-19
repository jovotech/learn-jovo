# New Database Integration: Google Cloud Firestore (Firebase)

The Jovo Framework now supports Google Cloud Firestore, the database service offered by Google Firebase. You can now use this integration to easily store user specific data for your Alexa Skills and Google Actions built with Jovo.


## Getting Started with Firestore and Jovo

> Docs: [Google Cloud Firestore Database Integration](https://www.jovo.tech/docs/databases/firestore)

Download the package like this:

```sh
$ npm install --save jovo-db-firestore
```

Firestore can be enabled in the `src/app.js` file like this:

```javascript
const { Firestore } = require('jovo-db-firestore');

// Enable DB after app initialization
app.use(new Firestore());
```

Inside your `config.js` file you have to set your `credential` and your `databaseURL`. You can also optionally set the collection name (default is `UserData`):

```javascript
// config.js file
db: {
    Firestore: {
        credential: require('<path-to-credential-json-file>'),
        databaseURL: '<databaseURL>',
        collectionName: '<collectionName>'
    }
}
```

**Any thoughts? Let us know in the comments below.**


<!--[metadata]: { "description": "The Jovo Framework now supports Google Cloud Firestore, the database service offered by Google Firebase, as first-party integration to store user specific data.", "author": "jan-koenig", "tags": "Database, Firestore" }-->
