import GithubLink from '@site/src/components/GithubLink';

# The Database class

## Introduction

The Database class allows you to add a persistence layer in your bot. Basically, it's a database component where you can register adapaters and collections.

- An adapter is a class to talk with one database engine
- A collection is a named table in the database, and each collection can be set up to have an adapter. That means that you can set the database to use different database engines for different collections.

## Basic Usage

```javascript
const { dockStart } = require("@nlpjs/basic");
const { Database } = require("@nlpjs/database");

(async () => {
  const dock = await dockStart();
  dock.getContainer().use(Database);
  const database = dock.get("database");
  await database.connect();
  const collection = database.getCollection("items");
  const items = [];
  for (let i = 0; i < 100; i += 1) {
    const item = { num: i, mod: i % 10 };
    items.push(item);
  }
  await collection.insertMany(items);
  const actual = await collection.find({ mod: 3 });
  await database.disconnect();
  console.log(actual);
})();
```

## Registering the Database component

As usual, you can either register the database component programmatically, as in the example above:

```javascript
const dock = await dockStart();
dock.getContainer().use(Database);
```

or using the configuration object:

```json
{
  "settings": {
    //your settings
  },
  "use": [
    //some other plugins
    "database"
  ]
}
```

## Configuration

See [configuration](/configuration#database) section for more details on how to configure the Database component.

## Using multiple adapters

_Database_ allows you to register any number of adapters. By default a <GithubLink to="packages/database/src/memory-adapter.js">MemorydbAdapter</GithubLink>
will be registered as the default adapter. This means that any collection whose adapter has not been explicitly defined, will use the default adapter.
To register additional adapters use the method _registerAdapter_:

```javascript
const database = dock.get("database");
database.registerAdapter("fancyDB", new MyFancyDBAdapter());
```

Once you have your additional adapter registered, you can get some collections to exist in one of the registered adapters:

```javascript
database.registerCollection("myFancyDBCollection", "fancyDB");
```

Then, when operating in a collection the adapter will automatically choose the proper adapter for that collections:

```javascript
//look for an element in the myFancyDBCollection will use the adapter registered as 'fancyDB'
database.findById("myFancyDBCollection", 1);
//as no adapter has been registered for collection 'randomCollection', MemorydbAdapter will be used
database.findById("randomCollection", 1);
```

## Creating a custom adapter

When using an adapter, _Database_ will delegate the operation to the proper adapter. For instance:

```javascript
findById(name, id) {
  //retrieve the adapter associated to that collections, and delegate the execution to it
  return this.getAdapter(name).findById(name, id);
}
```

So, at the end of the day, you'll just need to create an object implementing the proper methods you'll find in
the <GithubLink to="packages/database/src/database.js">Database</GithubLink> class.

## Out of the box adapters

### MemoryDBAdapter

The <GithubLink to="packages/database/src/memory-adapter.js">MemorydbAdapter</GithubLink> uses JSON structures
to emulate an in-memory database. If needed, it can be set to persist those JSON structures into file system.
Check [here](/configuration#memory-adapter) for available configuration options.

### MongoDBAdapter

The <GithubLink to="packages/mongodb-adapter/src/mongodb-adapter.js">MongodbAdapter</GithubLink> allows
you to use any MongoDB compliant database as your persistence layer.
Check [here](/configuration#mongodb-adapter) for available configuration options.