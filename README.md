# tiny-indexed-db

This is a tiny Promise-based library for working with indexedDB in JavaScript.

# The `unique-string` Library

The additional `unique-string.js` library is used to generate unique strings, which can be used as a primary key, an identifier, of an object data in an `indexedDB` object store. This should be used for prototyping only, as strings _may not_ be unique, even though this is unlikely.

To use it, make sure to import `UniqueString` from the `unique-string.js` file:

```js
import UniqueString from '<your-path>/unique-string' ;
```

To generate a unique string, create a new instance of the `UniqueString` class:

```js
const uid = new UniqueString();
```

Then, use the `generate()` method on the new instance. The method will return a string with randomly generated numbers, special characters and alphabets:

```js
let idString = uid.generate();
```

> **If you want to contribute** to the `unique-string` library, please add your features in the `unique-string.ts` file; the JavaScript file should not be touched.

# The `tiny-idb` Library

To use this, import `DB` from `tiny-idb.js` file:

```js
import DB from '<your-path>/tiny-idb';
```

**Note** that, the `tiny-idb.js` file does not depend on the `unique-string.js` file.

## Creating/initialising an `indexedDB` database

### Syntax:

```js
DB.createDB(name, version, [, stores]);
```

-   `name` - the name of the database.
-   `version` - version of the database.
-   `stores` - an _(optional)_ array of object literals each configuring object store that should be created once the database is successfully created.
-   `DB.createDB()` returns a Promise which resolves with the `indexedDB` database object.

### Example

```js
const db = DB.createDB('NamesDatabase', 1, [
    {
        name: 'namesStore', // name of object store

        config: { keyPath: 'nameID', /* and or autoIncrement: true */},

        // optional
        data: [
            { nameID: 'someID1', body: 'John Doe'},
            { nameID: 'someID2', body: 'Jane Doe'}
        ]
    }
]);

// optionally use the .then() to get access to the database object.
// Or, use async/await instead.
```

-   Create a database with name `NamesDatabase`, at version `1`, and
-   pass an array of object stores to be created once the database is created. [3rd argument]
-   Only one object store is created:
-   `name` - property sets name of the object store,
-   `config` - property defines the keyPath options of the object store.
-   `data` - an _(optional)_ property which defines an array of objects to be added automatically once the object store is successfully created.

## Opening a database after creation

The `DB.openDB()` method is for opening an already created `indexedDB` database:

### Syntax

```js
DB.openDB(name, version);
```

-   `name` - name of the database you want to open.
-   `version` - version of the database you want to access.
-   `DB.openDB()` returns a Promise which resolves with the `indexedDB` database object.

### Example

```js
(async () => {
    // open the database
    const db = await DB.openDB('NamesDatabase', 1);

    // do something with db
    ...
})();
```

### Opening a transaction

To open transaction on a database, use the `DB.transaction()` method.

## Syntax

```js
DB.transaction(dbObject, stores, mode);
```

-   `dbObject` - the `indexedDB` database object returned after opening or creating a database, not the name of the database.
-   `stores` - an array of the stores you want to open the transaction on.
-   `mode` - the access mode of the transaction: `readonly` or `readwrite`.
-   `DB.transaction()` returns an object with a `tx` property: the IDBTransaction object; and a `getStore()` method.
-   the `getStore()` method accepts one argument, the name of the store you want to open on the transaction, and returns a promise with the IDBOBjectStore object:

```js
// borrowing the IIFE from the earlier example
(async () => {
    const db = await DB.openDB('NamesDatabase', 1);

    // create a transaction and get the object store nameStore
    // access mode is set to read only
    const nameStore = await DB.transaction(db, ['nameStore'], 'readonly')
                            .getStore('nameStore');

    // or, you could just get the transaction object back
    const tx = await DB.transaction(db, ['nameStore'], 'readonly').tx;
})();
```

## Reading one object from an object store

To query the database and get one specific object in an object store, use `DB.getObjectData()`.

### Syntax

```js
DB.getObjectData(store, objectKeyPath);
```

-   `store` - the actual IDBOBjectStore object returned by getting an object store on a transaction, this parameter **cannot** be the name of the object store you want to query.
-   `objectKeyPath` - the `keyPath` or unique identifier of the object you want to get.
-   `DB.getObjectData()` returns a Promise which resolves with the object you queried for.

### Example

```js
(async () => {
    const db = await DB.openDB('NamesDatabase', 1);

    // create a transaction and get the object store nameStore
    // access mode is set to read-only
    const nameStore = await DB.transaction(db, ['nameStore'], 'readonly')
                            .getStore('nameStore');

    // get from the nameStore object store, the object with nameID of someID1
    const firstName = await DB.getObjectData(nameStore, 'someID1');
})();
```

## Reading all objects in an object store

This `DB.getAllObjectData()` method is similar to the `DB.getObjectData()` method, except it fetches all the objects in an object store.

### Syntax

```js
DB.getAllObjectData(store);
```

-   `store` - the actual IDBOBjectStore object returned by getting an object store on a transaction, this parameter **cannot** be the name of the object store you want to query.
-   `DB.getAllObjectData()` returns a Promise which resolves with an array of all the objects in the specified object store.

### Example

```js
(async () => {
    const db = await DB.openDB('NamesDatabase', 1);

    // create a transaction and get the object store nameStore
    // access mode is set to read-only
    const nameStore = await DB.transaction(db, ['nameStore'], 'readonly')
                            .getStore('nameStore');

    // get from the nameStore object store, all the available objects
    const allNames = await DB.getAllObjectData(nameStore);
})();
```

## Adding an object in a database

The `DB.addObjectData()` is used to achieve this.

### Syntax

```js
DB.addObjectData(store, dataBody);
```

-   `store` - the actual IDBOBjectStore object returned by getting an object store on a transaction, this parameter **cannot** be the name of the object store you want to query.

-   `dataBody` - the object you want to add; should include a property equal to the `keyPath` property specified on the object store; the keyPath value should be unique from those already in the object store.

-   `DB.addObjectData()` returns a Promise which resolves with an array of objects in the object store, including the object which was just added.

### Example

```js
(async () => {
    const db = await DB.openDB('NamesDatabase', 1);

    // create a transaction and get the object store nameStore
    // access mode is set to readwrite
    const nameStore = await DB.transaction(db, ['nameStore'], 'readwrite')
                            .getStore('nameStore');

    // add another name to the namesStore object store
    const updatedObjects = await DB.addObjectData(nameStore, {
        nameID: 'someID3', // here, you can generate a unique key with the `unique-string` library
        body: 'Some Name'
    });
})();
```

## Updating a specific object in an object store

To do this, use the `DB.updateObjectData()` method.

### Syntax

```js
DB.updateObjectData(store, keyPath, key, newData);
```

-   `store` - the actual IDBOBjectStore object returned by getting an object store on a transaction, this parameter **cannot** be the name of the object store you want to query.
-   `keyPath` - the name of the property which was set to be the keyPath of objects in an object store.
-   `key` - value of that of the keyPath property.
-   `newData` - the new data you're updating with, should contain the same unique key. Use a different unique key if you intend to use a different identifier for updating the object.
-   `DB.updateObjectData()` returns a Promise which resolves with an array of all the objects in the specified object store.

### Example

```js
(async () => {
    const db = await DB.openDB('NamesDatabase', 1);

    // create a transaction and get the object store nameStore
    // access mode is set to readwrite
    const nameStore = await DB.transaction(db, ['nameStore'], 'readwrite')
                            .getStore('nameStore');

    // in the namesStore object store, update an object whose nameID === 'someID1'
    const updatedObjects = await DB.updateObjectData(nameStore, 'nameID', 'someID1', {
        nameID: 'someID1',
        body: 'John Doe Snr.'
    });
})();
```

## Deleting objects from an object store

To achieve this, use the `DB.deleteObjectData()` method.

### Syntax

```js
DB.deleteObjectData(store, dataKey);
```

-   `store` - the actual IDBOBjectStore object returned by getting an object store on a transaction, this parameter **cannot** be the name of the object store you want to query.
-   `dataKey` - a unique key whose object you want to delete from the object store.
-   `DB.deleteObjectData()` returns a Promise which resolves with an array containing two things:

1. the deleted object,
2. an array of all objects remaining in the object store, after the targetted object was deleted.

### Example

```js
(async () => {
    const db = await DB.openDB('NamesDatabase', 1);

    // create a transaction and get the object store nameStore
    // access mode is set to readwrite
    const nameStore = await DB.transaction(db, ['nameStore'], 'readwrite')
                            .getStore('nameStore');

    // in the the namesStore object store, delete an object whose nameID === 'someID2'
    const [deletedObj, remainingObj] = await DB.deleteObjectData(namesStore, 'someID2');
})();
```

_These are all the commands that `tiny-idb.js` can perform, at this moment._
