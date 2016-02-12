# Overview

**The Parse hosted backend will be fully retired on January 28, 2017. If you are migrating an existing Parse app, please carefully read through this entire guide. You will need to go through the [migration guide](/ParsePlatform/parse-server/wiki/Migrating-an-Existing-Parse-App) or your app will stop working after the retirement date.**

Parse Server is an open source version of the Parse backend that can be deployed to any infrastructure that can run Node.js. You can find the source on the [GitHub repo](/ParsePlatform/parse-server).

* Parse Server is not dependent on the hosted Parse backend.
* Parse Server uses MongoDB directly, and is not dependent on the Parse hosted database.
* You can migrate an existing app to your own infrastructure.
* You can develop and test your app locally using Node.

## Prerequisites

* Node 4.1
* MongoDB version 2.6.X or 3.0.X
* Python 2.x (For Windows users, 2.7.1 is the required version)
* For deployment, an infrastructure provider like Heroku or AWS

## Installation

Start using Parse Server by grabbing the npm module:

```bash
npm install -g parse-server
```

Or, you can specify parse-server in your packages.json file.

# Usage

Parse Server is meant to be mounted on an [Express](http://expressjs.com/) app. Express is a web framework for Node.js. The fastest way to get started is to clone the [Parse Server repo](/ParsePlatform/parse-server), which at its root contains a sample Express app with the Parse API mounted.

The constructor returns an API object that conforms to an [Express Middleware](http://expressjs.com/en/api.html#app.use). This object provides the REST endpoints for a Parse app. Create an instance like so:

```js
var api = new ParseServer({
  databaseURI: 'mongodb://your.mongo.uri',
  cloud: './cloud/main.js',
  appId: 'myAppId',
  fileKey: 'myFileKey',
  masterKey: 'mySecretMasterKey',
  clientKey: 'myClientKey',
  restAPIKey: 'myRESTAPIKey',
  javascriptKey: 'myJavascriptKey',
  dotNetKey: 'myDotNetKey',
});
```

The parameters are as follows:

* `databaseURI`: Connection string URI for your MongoDB.
* `cloud`: Path to your app’s Cloud Code.
* `appId`: A unique identifier for your app.
* `fileKey`: A key that specifies a prefix used for file storage. For migrated apps, this is necessary to provide access to files already hosted on Parse.
* `masterKey`: A key that overrides all permissions. Keep this secret.
* `clientKey`: The client key for your app.
* `restAPIKey`: The REST API key for your app.
* `javascriptKey`: The JavaScript key for your app.
* `dotNetKey`: The .NET key for your app.

The Parse Server object was built to be passed directly into `app.use`, which will mount the Parse API at a specified path in your Express app:

```js
var express = require('express');
var ParseServer = require('parse-server').ParseServer;

var app = express();
var api = new ParseServer({ ... });

// Serve the Parse API at /parse URL prefix
app.use('/parse', api);

var port = 1337;
app.listen(port, function() {
  console.log('parse-server-example running on port ' + port + '.');
});
```

And with that, you will have a Parse Server running on port 1337, serving the Parse API at `/parse`.

# Database

Parse Server uses [MongoDB](https://www.mongodb.org/) as the database for your application. If you have not used MongoDB before, we highly recommend familiarizing yourself with it first before proceeding.

The Mongo requirements for Parse Server are:

* MongoDB version 2.6.X or 3.0.X
* The [failIndexKeyTooLong](https://docs.mongodb.org/manual/reference/parameters/#param.failIndexKeyTooLong) parameter must be set to `false`.
* An SSL connection is recommended (but not required).
* We strongly recommend that your MongoDB servers be hosted in the US-East region for minimal lantecy.

If this is your first time setting up a production MongoDB instance, we recommend using [MongoLab](http://www.mongolab.com), a database-as-a-service which has options to scale up as needed.

For a production app with non-trivial traffic, we recommend going with MongoLab's [M1 plan](https://mongolab.com/plans/pricing/#dedicated-cluster-plans) or higher, which provides 40GB of space. If you are migrating an existing Parse app, a good rule of thumb is to get an instance with 10X the space you currently are using with Parse.

When using MongoDB with your Parse app, there are some differences with the hosted Parse database:

* You need to manage your indexes yourself. Hosted Parse automatically adds indexes based on the incoming query stream.
* You need to size up your database as your data grows.

# Keys

Parse Server does not require the use of client-side keys. This includes the client key, JavaScript key, .NET key, and REST API key. The Application ID is sufficient to secure your app.

However, you have the option to specify any of these four keys upon initialization. Upon doing so, Parse Server will enforce that any clients passing a key matches. The behavior is consistent with hosted Parse.

# Using Parse SDKs with Parse Server

To use a Parse SDK with Parse Server, change the server URL to your Parse API URL (make sure you have the [latest version of the SDKs](https://parse.com/docs/downloads)). For example, if you have Parse Server running locally mounted at /parse:

**iOS**

```objc
[Parse initializeWithConfiguration:[ParseClientConfiguration configurationWithBlock:^(id<ParseMutableClientConfiguration> configuration) {
   ...
   
   configuration.applicationId = @"YOUR_APP_ID";
   configuration.clientKey = @"YOUR_APP_CLIENT_KEY";
   configuration.server = @"http://localhost:1337/parse";
   
   ...
   
}]];
```

**Android**

```java
Parse.initialize(new Parse.Configuration.Builder(myContext)
    .applicationId("YOUR_APP_ID")
    .clientKey("YOUR_APP_CLIENT_KEY")
    .server("http://localhost:1337/parse")


    ...
          
    .build()
);
```

**JavaScript**

```js
Parse.initialize("YOUR_APP_ID", "YOUR_APP_CLIENT_KEY");
Parse.serverURL = 'http://localhost:1337/parse'
```

**.NET**

```csharp
ParseClient.initialize(new ParseClient.Configuration {
    ApplicationId = "YOUR_APP_ID",
    WindowsKey = "YOUR_APP_DOTNET_KEY",
    Server = "http://localhost:1337/parse"
});
```
