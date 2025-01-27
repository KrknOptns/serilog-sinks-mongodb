# ![mongo icon](https://raw.githubusercontent.com/ChangemakerStudios/serilog-sinks-mongodb/dev/assets/mongo-icon.png) Serilog.Sinks.MongoDB

[![NuGet version](https://badge.fury.io/nu/Serilog.Sinks.MongoDB.svg)](https://badge.fury.io/nu/Serilog.Sinks.MongoDB)
[![Downloads](https://img.shields.io/nuget/dt/Serilog.Sinks.MongoDB.svg?logo=nuget&color=purple)](https://www.nuget.org/packages/Serilog.Sinks.MongoDB) 
[![Build status](https://github.com/ChangemakerStudios/serilog-sinks-mongodb/actions/workflows/deploy.yml/badge.svg)](https://github.com/ChangemakerStudios/serilog-sinks-mongodb/actions) 

A Serilog sink that writes events as documents to [MongoDB](http://mongodb.org).

**Package** - [Serilog.Sinks.MongoDB](http://nuget.org/packages/serilog.sinks.mongodb)
**Platforms** - .NET 4.7.2, .NET 6.0, .NET Standard 2.1

## Whats New

#### New in v7.x
* Upgrade MongoDB.Driver to v3.0 - .NET Standard 2.0 support has been removed.

#### New in v6.x
* Upgrade MongoDB.Driver to v2.28.0 (Thanks to [Memoyu](https://github.com/Memoyu))
* Add trace context to LogEntry (Thanks to [fernandovmp](https://github.com/fernandovmp))

#### New in v5.x
* Output structured MongoDB Bson logs by switching to `.MongoDBBson()` extensions. Existing `.MongoDB()` extensions will continue to work converting logs to Json and then to Bson.
* Rolling Log Collection Naming (Thanks to [Revazashvili](https://github.com/Revazashvili) for the PR!). MongoDBBson sink only.
* Expire TTL support. MongoDBBson sink only.

## Installation
Install the sink via NuGet Package Manager Console:

```powershell
Install-Package Serilog.Sinks.MongoDB
```

or via the .NET CLI:

```bash
dotnet add package Serilog.Sinks.MongoDB
```

## Usage Examples
In the examples below, the sink is writing to the database `logs` with structured Bson. The default collection name is `log`, but a custom collection can be supplied with the optional `CollectionName` parameter. The database and collection will be created if they do not exist.

### Basic:
```csharp
using Serilog;

// use BSON structured logs
var log = new LoggerConfiguration()
    .WriteTo.MongoDBBson("mongodb://mymongodb/logs")
    .CreateLogger();

log.Information("This is a test log message");
```

### Capped Collection:

```csharp
// capped collection using BSON structured logs
var log = new LoggerConfiguration()
    .WriteTo.MongoDBBson("mongodb://mymongodb/logs", cfg =>
    {
        // optional configuration options:
        cfg.SetCollectionName("log");
        cfg.SetBatchPeriod(TimeSpan.FromSeconds(1));

        // create capped collection that is max 100mb
        cfg.SetCreateCappedCollection(100);
    })
    .CreateLogger();
```

### Custom Mongodb Settings:
```csharp
// create sink instance with custom mongodb settings.
var log = new LoggerConfiguration()
	.WriteTo.MongoDBBson(cfg =>
    {
		// custom MongoDb configuration
		var mongoDbSettings = new MongoClientSettings
		{
			UseTls = true,			
			AllowInsecureTls = true,
			Credential = MongoCredential.CreateCredential("databaseName", "username", "password"),
			Server = new MongoServerAddress("127.0.0.1")
		};

		var mongoDbInstance = new MongoClient(mongoDbSettings).GetDatabase("serilog");
		
		// sink will use the IMongoDatabase instance provided
		cfg.SetMongoDatabase(mongoDbInstance);
		cfg.SetRollingInternal(RollingInterval.Month);
    })
	.CreateLogger();
```
### JSON (_Microsoft.Extensions.Configuration_)

Keys and values are not case-sensitive. This is an example of configuring the MongoDB sink arguments from _Appsettings.json_:

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Error",
        "System": "Warning"
      }
    },
    "WriteTo": [
      { 
      	"Name": "MongoDBBson", 
        "Args": { 
            "databaseUrl": "mongodb://username:password@ip:port/dbName?authSource=admin",
            "collectionName": "logs",
            "cappedMaxSizeMb": "1024",
            "cappedMaxDocuments": "50000",
            "rollingInterval": "Month"
        }
      } 
    ]
  }
}
```

## Icon

[MongoDB](https://icons8.com/icon/74402/mongodb) icon by [Icons8](https://icons8.com)
