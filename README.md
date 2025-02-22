# unleash-proxy-client-swift

The unleash-proxy-client-swift makes it easy for native applications and other swift platforms to connect to the unleash proxy. The proxy will evaluate a feature toggle for a given [context](https://docs.getunleash.io/docs/user_guide/unleash_context) and return a list of feature flags relevant for the provided context. 

The unleash-proxy-client-swift will then cache these toggles in a map in memory and refresh the configuration at a configurable interval, making queries against the toggle configuration extremely fast.

## Requirements
- MacOS: 12.15
- iOS: 13

## Installation
Follow the following steps in order to install the unleash-proxy-client-swift:

1. In your Xcode project go to File -> Swift Packages -> Add Package Dependency
2. Supply the link to this repository
3. Set the appropriate package constraints (typically up to next major version)
4. Let Xcode find and install the necessary packages

Once you're done, you should see SwiftEventBus and UnleashProxyClientSwift listed as dependencies in the file explorer of your project.

## Usage
In order to get started you need to import and instantiate the unleash client: 

```
import SwiftUI
import UnleashProxyClientSwift
    
// Setup Unleash in the context where it makes most sense

var unleash = UnleashProxyClientSwift.UnleashClient(unleashUrl: "https://app.unleash-hosted.com/hosted/api/proxy", clientKey: "PROXY_KEY", refreshInterval: 15, appName: "test", environment: "dev")
     
unleash.start()
```

In the example above we import the UnleashProxyClientSwift and instantiate the client. You need to provide the following parameters: 

- unleashUrl: the full url to your proxy instance [String]
- clientKey: the proxy key [String]
- refreshInterval: the polling interval in seconds [Int]
- appName: the application name identifier [String]
- environment: the application env [String]

Running `unleash.start()` will make the first request against the proxy and retrieve the feature toggle configuration, and set up the polling interval in the background.

NOTE: While waiting to boot up the configuration may not be available, which means that asking for a feature toggle may result in a false if the configuration has not loaded. In the event that you need to be certain that the configuration is loaded we emit an event you can subscribe to, once the configuration is loaded. See more in the Events section.

Once the configuration is loaded you can ask against the cache for a given feature toggle: 
```
if unleash.isEnabled(name: "ios") {
    // do something
} else {
   // do something else
}
```

You can also set up [variants](https://docs.getunleash.io/docs/advanced/toggle_variants) and use them in a similar fashion: 
```
var variant = unleash.getVariant(name: "ios")
if variant.enabled {
    // do something
} else {
   // do something else
}
```

### Update context
In order to update the context you can use the following method: 
```
var context: [String: String] = [:]
context["userId"] = "c3b155b0-5ebe-4a20-8386-e0cab160051e"
unleash.updateContext(context: context)
```

This will stop and start the polling interval in order to renew polling with new context values.

## Events

The proxy client emits two different events you can subscribe to: 

- "ready"
- "update"

Usage them in the following manner: 
```
func handleReady() {
    // do this when unleash is ready
}

unleash.subscribe(name: "ready", callback: handleReady)

func handleUpdate() {
    // do this when unleash is updated
}

unleash.subscribe(name: "update", callback: handleUpdate)
```

The ready event is fired once the client has received it's first set of feature toggles and cached it in memory. Every subsequent event will be an update event that is triggered if there is a change in the feature toggle configuration.
