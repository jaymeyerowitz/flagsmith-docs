---
description: Manage your Feature Flags and Remote Config in your Server Side Applications.
sidebar_label: Server Side
sidebar_position: 2
---

# Server Side SDKs

:::tip

Server Side SDKs can run in 2 different modes: `Local Evaluation` and `Remote Evaluation`. We recommend
[reading up about the differences](overview#remote-and-local-evaluation-modes) first before integrating the SDKS into
your applications.

Once you've got that understood, lets get the SDKs integrated!

:::

## Github Links

All our SDKs are on Github.

<Tabs groupId="language">
<TabItem value="py" label="Python">

https://github.com/Flagsmith/flagsmith-python-client

</TabItem>
<TabItem value="java" label="Java">

https://github.com/Flagsmith/flagsmith-java-client

</TabItem>
<TabItem value="dotnet" label=".NET">

https://github.com/Flagsmith/flagsmith-dotnet-client

</TabItem>
<TabItem value="nodejs" label="NodeJS">

https://github.com/Flagsmith/flagsmith-nodejs-client

</TabItem>
<TabItem value="php" label="PHP">

https://github.com/Flagsmith/flagsmith-php-client

</TabItem>
<TabItem value="go" label="Go">

https://github.com/Flagsmith/flagsmith-go-client

</TabItem>
<TabItem value="rust" label="Rust">

https://github.com/Flagsmith/flagsmith-rust-client

</TabItem>
<TabItem value="elixir" label="Elixir">

https://github.com/Flagsmith/flagsmith-elixir-client

</TabItem>
</Tabs>

## Add the Flagsmith package

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem';

<Tabs groupId="language">
<TabItem value="py" label="Python">

```bash
pip install flagsmith
```

</TabItem>
<TabItem value="java" label="Java">

```xml
# Check https://search.maven.org/artifact/com.flagsmith/flagsmith-java-client
# for the latest version!

# Maven
<dependency>
  <groupId>com.flagsmith</groupId>
  <artifactId>flagsmith-java-client</artifactId>
  <version>5.0.0</version>
</dependency>

# Gradle
implementation 'com.flagsmith:flagsmith-java-client:5.0.0'
```

</TabItem>
<TabItem value="dotnet" label=".NET">

```bash
# Package Manager
Install-Package Flagsmith -Version 4.0.0

#.NET CLI
dotnet add package Flagsmith --version 4.0.0

# PackageReference
<PackageReference Include="Flagsmith" Version="4.0.0" />

# Paket CLI
paket add Flagsmith --version 4.0.0
```

</TabItem>
<TabItem value="nodejs" label="NodeJS">

```bash
# Via NPM
npm i flagsmith-nodejs --save
```

</TabItem>
<TabItem value="php" label="PHP">

```bash
# Requires PHP 7.4 or newer and ships with GuzzleHTTP.
composer require flagsmith/flagsmith-php-client

# You can optionally provide your own implementation of PSR-18 and PSR-16.
# You will also need some implementation of PSR-18 and PSR-17,
# for example Guzzle and PSR-16, for example Symfony Cache.
composer require flagsmith/flagsmith-php-client guzzlehttp/guzzle symfony/cache

# or
composer require flagsmith/flagsmith-php-client symfony/http-client nyholm/psr7 symfony/cache
```

</TabItem>
<TabItem value="go" label="Go">

```bash
go get github.com/Flagsmith/flagsmith-go-client
```

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

:todo

</TabItem>
</Tabs>

## Initialise the SDK

<Tabs groupId="language">
<TabItem value="py" label="Python">

```python
from flagsmith import Flagsmith

flagsmith = Flagsmith(
    environment_key = os.environ.get("<FLAGSMITH_ENVIRONMENT_KEY>")
)
```

</TabItem>
<TabItem value="java" label="Java">

```java
private static FlagsmithClient flagsmith = FlagsmithClient
    .newBuilder()
    .setApiKey(System.getenv("<FLAGSMITH_ENVIRONMENT_KEY>"))
    .build();
```

</TabItem>
<TabItem value="dotnet" label=".NET">

```csharp
using Flagsmith;

static FlagsmithClient _flagsmithClient;

_flagsmithClient = new("<FLAGSMITH_ENVIRONMENT_KEY>");
```

</TabItem>
<TabItem value="nodejs" label="NodeJS">

```javascript
const Flagsmith = require('flagsmith-nodejs');

const flagsmith = new Flagsmith({'<FLAGSMITH_ENVIRONMENT_KEY>'});


flagsmith.init({
 environmentID: '<FLAGSMITH_ENVIRONMENT_KEY>',
});
```

</TabItem>
<TabItem value="php" label="PHP">

```php
use Flagsmith\Flagsmith;

$flagsmith = new Flagsmith('<FLAGSMITH_ENVIRONMENT_KEY>');
```

</TabItem>
<TabItem value="go" label="Go">

:todo

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

:todo

</TabItem>
</Tabs>

## Get Flags for an Environment

<Tabs groupId="language">
<TabItem value="py" label="Python">

```python
# The method below triggers a network request
flags = flagsmith.get_environment_flags()
show_button = flags.is_feature_enabled("secret_button")
button_data = json.loads(flags.get_feature_value("secret_button"))
```

</TabItem>
<TabItem value="java" label="Java">

```java
Flags flags = flagsmith.getEnvironmentFlags();
Boolean showButton = flags.isFeatureEnabled(featureName);
Object value = flags.getFeatureValue(featureName);
```

</TabItem>
<TabItem value="dotnet" label=".NET">

```csharp
# Sync
# The method below triggers a network request
var flags = _flagsmithClient.GetEnvironmentFlags();  # This method triggers a network request
var showButton = flags.IsFeatureEnabled("secret_button");
var buttonData = flags.GetFeatureValue("secret_button").Result;

# Async
# The method below triggers a network request
var flags = await _flagsmithClient.GetEnvironmentFlags();  # This method triggers a network request
var showButton = await flags.IsFeatureEnabled("secret_button");
var buttonData = await flags.GetFeatureValue("secret_button").Result;
```

</TabItem>
<TabItem value="nodejs" label="NodeJS">

```javascript
const flags = await flagsmith.getEnvironmentFlags();
var showButton = flags.isFeatureEnabled('secret_button');
var buttonData = flags.getFeatureValue('secret_button');
```

</TabItem>
<TabItem value="php" label="PHP">

```php
$flags = $flagsmith->getFlags();
$flags->isFeatureEnabled('secret_button')
$flags->getFeatureValue('secret_button')
```

</TabItem>
<TabItem value="go" label="Go">

:todo

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

:todo

</TabItem>
</Tabs>

## Get Flags for an Identity

<Tabs groupId="language">
<TabItem value="py" label="Python">

```python
identifier = "delboy@trotterstraders.co.uk"
traits = {"car_type": "robin_reliant"}

# The method below triggers a network request
identity_flags = flagsmith.get_identity_flags(identifier=identifier, traits=traits)
show_button = identity_flags.is_feature_enabled("secret_button")
button_data = json.loads(identity_flags.get_feature_value("secret_button"))
```

</TabItem>
<TabItem value="java" label="Java">

```java
String identifier = "delboy@trotterstraders.co.uk"
Map<String, Object> traits = new HashMap<String, Object>();
traits.put("car_type", "robin_reliant");

// The method below triggers a network request
Flags flags = flagsmith.getIdentityFlags(identifier, traits);
Boolean showButton = flags.isFeatureEnabled(featureName);
Object value = flags.getFeatureValue(featureName);
```

</TabItem>
<TabItem value="dotnet" label=".NET">

```csharp
var Identifier = "delboy@trotterstraders.co.uk";
var traitKey = "car_type";
var traitValue = "robin_reliant";
var traitList = new List<Trait> { new Trait(traitKey, traitValue) };

# Sync
# The method below triggers a network request
var flags = _flagsmithClient.GetIdentityFlags(Identifier, traitList);
var showButton = flags.IsFeatureEnabled("secret_button");

# Async
# The method below triggers a network request
var flags = await _flagsmithClient.GetIdentityFlags(Identifier, traitList);
var showButton = await flags.IsFeatureEnabled("secret_button");
```

</TabItem>
<TabItem value="nodejs" label="NodeJS">

```javascript
const identifier = 'delboy@trotterstraders.co.uk';
const traitList = { car_type: 'robin_reliant' };

const flags = await flagsmith.getIdentityFlags(identifier, traitList);
var showButton = flags.isFeatureEnabled('secret_button');
var buttonData = flags.getFeatureValue('secret_button');
```

</TabItem>
<TabItem value="php" label="PHP">

```php
$identifier = 'delboy@trotterstraders.co.uk';
$traits = (object) [ 'car_type' => 'robin_reliant' ];

$flags = $flagsmith->getIdentityFlags($identifier, $traits);
$showButton = $flags->isFeatureEnabled('secret_button');
$buttonData = $flags->getFeatureValue('secret_button');
```

</TabItem>
<TabItem value="go" label="Go">

:todo

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

:todo

</TabItem>
</Tabs>

## Managing Default Flags

Default Flags are configured by passing in a function that is called when a Flag cannot be found.

<Tabs groupId="language">
<TabItem value="py" label="Python">

```python
from flagsmith import Flagsmith
from flagsmith.models import DefaultFlag

def default_flag_handler(feature_name: str) -> DefaultFlag:
    """
    Function that will be used if the API doesn't respond, or an unknown
    feature is requested
    """
    if feature_name == "secret_button":
        return DefaultFlag(
            enabled=False,
            value=json.dumps({"colour": "#b8b8b8"}),
            feature_name="secret_button",
        )
    ],
    return DefaultFlag(False, None)

flagsmith = Flagsmith(
    environment_key=os.environ.get("<FLAGSMITH_ENVIRONMENT_KEY>"),
    default_flag_handler=default_flag_handler,
)
```

</TabItem>
<TabItem value="java" label="Java">

```java
private static FlagsmithClient flagsmith = FlagsmithClient
    .newBuilder()
    .setDefaultFlagValueFunction(HelloController::defaultFlagHandler)
    .setApiKey(System.getenv("<FLAGSMITH_ENVIRONMENT_KEY>"))
    .build();

private static DefaultFlag defaultFlagHandler(String featureName) {
    DefaultFlag flag = new DefaultFlag();
    flag.setEnabled(Boolean.FALSE);

    if (featureName.equals("secret_button")) {
        flag.setValue("{\"colour\": \"#ababab\"}");
    } else {
        flag.setValue(null);
    }

    return flag;
}
```

</TabItem>
<TabItem value="dotnet" label=".NET">

```csharp
using Flagsmith;

static FlagsmithClient _flagsmithClient;

_flagsmithClient = new("<FLAGSMITH_ENVIRONMENT_KEY>", defaultFlagHandler: defaultFlagHandler);
static Flag defaultFlagHandler(string featureName)
{
    // Function that will be used if the API doesn't respond, or an unknown
    // feature is requested
    if (featureName == "secret_button")
        return new Flag(new Feature("secret_button"), enabled: false, value: JsonConvert.SerializeObject(new { colour = "#b8b8b8" }).ToString());
    else return new Flag() { };
}
```

</TabItem>
<TabItem value="nodejs" label="NodeJS">

```javascript
const flagsmith = new Flagsmith({
 environmentKey,
 enableLocalEvaluation: true,
 defaultFlagHandler: (str) => {
  return { enabled: false, isDefault: true, value: { colour: '#ababab' } };
 },
});
```

</TabItem>
<TabItem value="php" label="PHP">

```php
$flagsmith = (new Flagsmith('<FLAGSMITH_ENVIRONMENT_KEY>'))
    ->withDefaultFlagHandler(function ($featureName) {
        $defaultFlag = (new DefaultFlag())
            ->withEnabled(false)->withValue(null);
        if ($featureName === 'secret_button') {
            return $defaultFlag->withValue('{"colour": "#ababab"}');
        }

        return $defaultFlag;
    });
```

</TabItem>
<TabItem value="go" label="Go">

:todo

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

:todo

</TabItem>
</Tabs>

## Network Behaviour

The Server Side SDKS share the same network behaviour across the different languages:

### Remote Evaluation Mode Network Behaviour

- A blocking network request is made every time you make a call to get an Environment Flags. In Python, for example,
  `flagsmith.get_environment_flags()` will trigger this request.
- A blocking network request is made every time you make a call to get an Identities Flags. In Python, for example,
  `flagsmith.get_identity_flags(identifier=identifier, traits=traits)` will trigger this request.

### Local Evaluation Mode Network Behaviour

- When the SDK is initialised, it will make an asnchronous network request to retrieve details about the Environment.
- Every 60 seconds (by default), it will repeat this aysnchronous request to ensure that the Environment information it
  has is up to date.

## Configuring the SDK

You can modify the behaviour of the SDK during initialisation. Full configuration options are shown below.

<Tabs groupId="language">
<TabItem value="py" label="Python">

```python
flagsmith = Flagsmith(
    # Your API Token.
    # Note that this is either the `Environment API` key or the `Server Side SDK Token`
    # depending on if you are using Local or Remote Evaluation
    # Required.
    environment_key = os.environ.get("FLAGSMITH_ENVIRONMENT_KEY"),

    # Controls which mode to run in; local or remote evaluation.
    # See the `SDKs Overview Page` for more info
    # Optional.
    # Defaults to False.
    enable_local_evaluation = False,

    # Override the default Flagsmith API URL if you are self-hosting.
    # Optional.
    # Defaults to https://edge.api.flagsmith.com/api/v1/
    api_url = "https://api.yourselfhostedflagsmith.com/api/v1/",

    # The network timeout in seconds.
    # Optional.
    # Defaults to 10 seconds
    request_timeout_seconds = 10,

    # When running in local evaluation mode, defines
    # how often to request an updated Environment document in seconds
    # Optional
    # Defaults to 60 seconds
    environment_refresh_interval_seconds: int = 60,

    # A `urllib3` Retries object to control network retry policy
    # See https://urllib3.readthedocs.io/en/latest/reference/urllib3.util.html#urllib3.util.Retry
    # Optional
    # Defaults to None
    retries: Retry = None,

    # Controls whether Flag Analytics data is sent to the Flagsmith API
    # See https://docs.flagsmith.com/advanced-use/flag-analytics
    # Optional
    # Defaults to False
    enable_analytics: bool = False,

    # You can pass custom headers to the Flagsmith API with this Dictionary.
    # This can be helpful, for example, when sending request IDs to help trace requests.
    # Optional
    # Defaults to None
    custom_headers: typing.Dict[str, typing.Any] = None,

    # You can specify default Flag values on initialisation.
    # Optional
    defaults = [
        # Set a default flag which will be used if the "secret_button"
        # feature is not returned by the API or if the API is not reachable
        DefaultFlag(
            enabled=False,
            value=json.dumps({"colour": "#b8b8b8"}),
            feature_name="secret_button",
        ),
        DefaultFlag(
            enabled=True,
            feature_name="secret_feature",
        )
    ],
)
```

</TabItem>
<TabItem value="java" label="Java">

```java
private static FlagsmithClient flagsmith = FlagsmithClient
    .newBuilder()
    // Your API Token.
    // Note that this is either the `Environment API` key or the `Server Side SDK Token`
    // depending on if you are using Local or Remote Evaluation
    // Required.
    .setApiKey(System.getenv("FLAGSMITH_API_KEY"))

    // Controls which mode to run in; local or remote evaluation.
    // See the `SDKs Overview Page` for more info
    // Optional.
    // Defaults to False.
    .withLocalEvaluation(True)

    // Override the default Flagsmith API URL if you are self-hosting.
    // Optional.
    // Defaults to https://edge.api.flagsmith.com/api/v1/
    .baseUri("https://api.yourselfhostedflagsmith.com/api/v1/")

    // Set environment refresh rate with polling manager.
    // Only needed when local evaluation is true.
    // Optional.
    // Defaults to 60 seconds
    .withEnvironmentRefreshIntervalSeconds(Integer seconds)

    // You can specify default Flag values on initialisation.
    // Optional
    .setDefaultFlagValueFunction(HelloController::defaultFlagHandler)

    // Controls whether Flag Analytics data is sent to the Flagsmith API
    // See https://docs.flagsmith.com/advanced-use/flag-analytics
    // Optional
    // Defaults to False
    .withEnableAnalytics(Boolean enable) {

    // The network timeout in seconds.
    // See https://square.github.io/okhttp/4.x/okhttp/okhttp3/ for details
    // Optional.
    .connectTimeout(<millisecond int>)
    .writeTimeout(<millisecond int>)
    .readTimeout(<millisecond int>)

    // Override the sslSocketFactory
    // See https://square.github.io/okhttp/4.x/okhttp/okhttp3/ for details
    // Optional.
    .sslSocketFactory(SSLSocketFactory sslSocketFactory, X509TrustManager trustManager)

    // Add a custom HTTP interceptor.
    // See https://square.github.io/okhttp/4.x/okhttp/okhttp3/ for details
    // Optional.
    .addHttpInterceptor(Interceptor interceptor)

    // Add retries for HTTP request to the builder.
    // Optional.
    .retries(Retry retries)

    .build();
```

</TabItem>
<TabItem value="dotnet" label=".NET">

```csharp
_flagsmithClient = new(
    # Your API Token.
    # Note that this is either the `Environment API` key or the `Server Side SDK Token`
    # depending on if you are using Local or Remote Evaluation
    # Required.
    environmentKey: "FLAGSMITH_ENVIRONMENT_KEY",

    # Pass in a default Flag Handler method
    # Optional
    defaultFlagHandler: defaultFlagHandler,

    # Override the default Flagsmith API URL if you are self-hosting.
    # Optional.
    # Defaults to https://edge.api.flagsmith.com/api/v1/
    apiUrl: "https://flagsmith.myproject.com"

    # Controls which mode to run in; local or remote evaluation.
    # See the `SDKs Overview Page` for more info
    # Optional.
    # Defaults to False.
    enableClientSideEvaluation = false;

    # Controls whether Flag Analytics data is sent to the Flagsmith API
    # See https://docs.flagsmith.com/advanced-use/flag-analytics
    # Optional
    # Defaults to false
    enableAnalytics: false

    # When running in local evaluation mode, defines
    # how often to request an updated Environment document in seconds
    # Optional
    # Defaults to 60 seconds
    environmentRefreshIntervalSeconds: 60

    # You can pass custom headers to the Flagsmith API with this Dictionary.
    # This can be helpful, for example, when sending request IDs to help trace requests.
    # Optional
    # Defaults to None
    customHeaders: <Dictionary>

    # How often to retry failed HTTP requests
    # Optional
    # Defaults to 1
    retries: 1

    # The network timeout in seconds.
    # Optional.
    # Defaults to null (http client default)
    requestTimeout = null,
)
```

</TabItem>
<TabItem value="nodejs" label="NodeJS">

```javascript
const flagsmith = new Flagsmith({
    /*
    Your API Token.
    Note that this is either the `Environment API` key or the `Server Side SDK Token`
    depending on if you are using Local or Remote Evaluation
    Required.
    */
    '<FLAGSMITH_ENVIRONMENT_KEY>',

    /*
    Controls which mode to run in; local or remote evaluation.
    See the `SDKs Overview Page` for more info
    Optional.
    Defaults to false.
    */
    enableLocalEvaluation: true,

    /*
    Controls which mode to run in; local or remote evaluation.
    See the `SDKs Overview Page` for more info
    Optional.
    Defaults to false.
    */
    apiUrl: 'https://api.yourselfhostedflagsmith.com/api/v1/',

    /*
    Set environment refresh rate with polling manager.
    Only needed when local evaluation is true.
    Optional.
    Defaults to 60 seconds
    */
    environmentRefreshIntervalSeconds: 60,

    /*
    You can specify default Flag values on initialisation.
    Optional
    */
    defaultFlagHandler: str => {
        return { enabled: false, isDefault: true, value: null };
    },

    /*
    Controls whether Flag Analytics data is sent to the Flagsmith API
    See https://docs.flagsmith.com/advanced-use/flag-analytics
    Optional
    Defaults to false
    */
    enableAnalytics: true

    /*
    The network timeout in seconds.
    Optional.
    Defaults to 10 seconds
    */
    requestTimeoutSeconds: 30,

    /*
    Custom http headers can be added to the http client
    Optional
    */
    customHeaders: { 'aHeader': 'aValue' },
});
```

</TabItem>
<TabItem value="php" label="PHP">

```php
$flagsmith = new Flagsmith(
    /*
    Your API Token.
    Note that this is either the `Environment API` key or the `Server Side SDK Token`
    depending on if you are using Local or Remote Evaluation
    Required.
    */
    string $apiKey,

    /*
    Controls which mode to run in; local or remote evaluation.
    See the `SDKs Overview Page` for more info
    Optional.
    Defaults to false.
    */
    string $host = self::DEFAULT_API_URL,

    /*
    Custom http headers can be added to the http client
    Optional
    */
    object $customHeaders = null,

    /*
    Set environment refresh rate with polling manager.
    This also enables local evaluation.
    Optional.
    Defaults to null
    */
    int $environmentTtl = null,

    /*
    Retry Object, instance of Flagsmith\Utils\Retry
    Retry configuration for api calls.
    Defaults to 3 retries for every api call.
    */
    Retry $retries = null,

    /*
    Controls whether Flag Analytics data is sent to the Flagsmith API
    See https://docs.flagsmith.com/advanced-use/flag-analytics
    Optional
    Defaults to false
    */
    bool $enableAnalytics = false,

    /*
    You can specify default Flag values on initialisation.
    Optional
    */
    Closure $defaultFlagHandler = null
);
```

</TabItem>
<TabItem value="go" label="Go">

:todo

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

:todo

</TabItem>
</Tabs>

## Contribute to the SDKs

All our SDKs are Open Source.

<Tabs groupId="language">
<TabItem value="py" label="Python">

https://github.com/Flagsmith/flagsmith-python-client

</TabItem>
<TabItem value="java" label="Java">

https://github.com/Flagsmith/flagsmith-java-client

</TabItem>
<TabItem value="dotnet" label=".NET">

https://github.com/Flagsmith/flagsmith-dotnet-client

</TabItem>
<TabItem value="nodejs" label="NodeJS">

https://github.com/Flagsmith/flagsmith-nodejs-client

</TabItem>
<TabItem value="php" label="PHP">

https://github.com/Flagsmith/flagsmith-php-client

</TabItem>
<TabItem value="go" label="Go">

:todo

</TabItem>
<TabItem value="rust" label="Rust">

:todo

</TabItem>
<TabItem value="elixir" label="Elixir">

https://github.com/Flagsmith/flagsmith-elixir-client

</TabItem>
</Tabs>
