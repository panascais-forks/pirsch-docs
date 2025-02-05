---
title: "JavaScript SDK"
date: 2022-10-30
draft: false
weight: 4
description: "SDK for JavaScript."
---

The JavaScript SDK can be found on [GitHub](https://github.com/pirsch-analytics/pirsch-js-sdk).

## Installation

```Bash
npm i pirsch-sdk
```

## Create a Client

To use the [API]({{<ref "api-sdks/api.md">}}), you need to create a client on the settings page first and use the client ID, the secret, and hostname to set up the SDK.

```JavaScript
// Import the Pirsch client.
import { Pirsch } from "pirsch-sdk";

// Create a client with the hostname, client ID, and client secret you have configured on the Pirsch dashboard.
const client = new Pirsch({
    hostname: "example.com",
    protocol: "https", // used to parse the request URL, default is https
    clientId: "<client_id>",
    clientSecret: "<client_secret>"
});
```

The `client_id` is optional if you use a client with a single access token (starting with `pa_`). Clients using this kind of access tokens can only send data (page views, events, or keep alive sessions). Both types should treat the secret/access token as a password.

From here on we can make API calls through the `client`. It will automatically update the access token using the credentials you provided.

## Send a Page Hit

To monitor your website traffic, you need to send hits to Pirsch. This is done by calling the `hit` method from a handler function.

```JavaScript
import { createServer } from "node:http";

createServer((request, response) => {
    // Send the hit to Pirsch. hitFromRequest is a helper function that returns all required information from the request.
    // You can also built the Hit object on your own and pass it in.
    client.hit(client.hitFromRequest(request)).catch(error => {
        // Something went wrong, check the error output.
        console.error(error);
    });

    // Render your website...
    response.write("Hello from Pirsch!");
    response.end();
}).listen(8765);
```

`hit` takes a `Hit` object as an input and sends all relevant data to Pirsch. `hitFromRequest` is a helper method that returns a hit object for the given NodeJS http request. Depending on your framework, you might need to fill the object yourself.

Note that the handler above accepts all requests made by a client and will therefore lead to a lot of different paths being tracked. Usually you would make sure that only the page itself gets monitored by checking the requested path.

```JavaScript
import { URL } from "node:url";

// In this example, we only want to track the / path and nothing else.
// We parse the request URL to read and check the pathname.
const url = new URL(request.url || "", "https://example.com");

if (url.pathname === "/") {
    // Send the hit to Pirsch. hitFromRequest is a helper function that returns all required information from the request.
    // You can also built the Hit object on your own and pass it in.
    client.hit(client.hitFromRequest(request)).catch(error => {
        // Something went wrong, check the error output.
        console.error(error);
    });
}
```

You can send a hit whenever you want. If you have a page with dynamic content for example, you can check if the content was found and send a hit in that case, or otherwise ignore it.

## Send an Event

You can send [events]({{<ref "dashboard/events.md">}}) to Pirsch including custom metadata fields and a duration. This is done by calling the `event` method from a handler function.

```JavaScript
import { createServer } from "node:http";
import { URL } from "node:url";

// Import the Pirsch client.
import { Pirsch } from "pirsch-sdk";

// Create a client with the hostname, client ID, and client secret you have configured on the Pirsch dashboard.
const client = new Pirsch({
    hostname: "example.com",
    protocol: "http", // used to parse the request URL, default is https
    clientId: "<client_id>",
    clientSecret: "<client_secret>"
});

// Create your http handler and start the server.
createServer((request, response) => {
    // In this example, we only want to track the / path and nothing else.
    // We parse the request URL to read and check the pathname.
    const url = new URL(request.url || "", "http://example.com");

    if (url.pathname === "/") {
        // Send the hit to Pirsch. hitFromRequest is a helper function that returns all required information from the request.
        // You can also built the Hit object on your own and pass it in.
        client.event("Event Name", client.hitFromRequest(request), 42, { meta: "data", clicks: 19 }).catch(error => {
            console.log(error);
        });
    }

    // Render your website...
    response.write("Hello from Pirsch!");
    response.end();
}).listen(8765);
```

`event` takes the event name, the hit object, the duration and a metadata map and sends all relevant data to Pirsch.

## Accessing Your Data

The client offers methods to access your statistics. Before you can use them, read the domain and construct the filter. The filter requires the domain ID and filter range to be set (start and end date). Here is an example on how to read the domain belonging to the client and constructing a simple filter (make sure you handle errors).

```JavaScript
// Import the Pirsch client.
import { Pirsch } from "pirsch-sdk";
import { PirschApiError } from "pirsch-sdk/common";

// Create a client with the hostname, client ID, and client secret you have configured on the Pirsch dashboard.
const client = new Pirsch({
    hostname: "example.com",
    protocol: "http", // used to parse the request URL, default is https
    clientId: "<client_id>",
    clientSecret: "<client_secret>"
});

// all client methods might return an APIError, make sure to handle that...
const domains = await client.domain();

if (domains instanceof PirschApiError) {
    console.error(domains.message);
    return;
}

const domain = domains[0]; // the returned list will contain only the client domain

const filter = {
    id: domain.id,
    from: new Date("2021-06-19"),
    to: new Date("2021-06-26")
};
```

You can now use the filter to select results.

```JavaScript
// Import the Pirsch client.
import { Pirsch } from "pirsch-sdk";
import { PirschApiError } from "pirsch-sdk/common";

// Create a client with the hostname, client ID, and client secret you have configured on the Pirsch dashboard.
const client = new Pirsch({
    hostname: "example.com",
    protocol: "http", // used to parse the request URL, default is https
    clientId: "<client_id>",
    clientSecret: "<client_secret>"
});

const visitors = await client.visitors(filter);

if (visitors instanceof PirschApiError) {
    console.error(visitors.message);
    return;
}

// do something with visitors
```

Please refer to the GitHub repository or inline documentation for more methods and data types.
