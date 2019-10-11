# iOS Client

This library can be used with iOS and Mac applications. The source code for the client is available on [Github](https://github.com/SolidStateGroup/bullet-train-ios-client).

## Usage

```swift
BulletTrain.shared.apiKey = "<add your API key from the Bullet Train settings page>"
BulletTrain.shared.getFeatureFlags() { (result) in
    print(result)
}
BulletTrain.shared.hasFeatureFlag(withID: "freeze_delinquent_accounts") { (result) in
    print(result)
}
```