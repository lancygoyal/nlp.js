# Microsoft Bot Framework connector

THe MSBF is a connector allowing you to interact with a Bot using an Azure Bot instance.

## Installation

You can install @nlpjs/msbf-connector:

```bash
    npm install @nlpjs/msbf-connector
```

## Usage

The directline connector is an HTTP based connector, because of that it requires the usage of the express-api-server <!--(**TODO**: add link) component -->to be registered in your bot.
This is the main basic configuration recommended in order to use this connector:

```json
{
  "settings": {
    ...
  },
  "use": [
    ...,
    "ExpressApiServer",
    "DirectlineConnector",
    "Bot"]
}
```

Of course, you can add any other configuration you may need.

## Configuration

For configuration details refer to [the proper configuration section](/configuration#msbf-connector)