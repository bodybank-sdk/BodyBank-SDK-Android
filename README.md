# BodyBank Enterprise SDK Readme

## Version
2018-12-21-1
SDK Version 0.0.35

## Update History
- 2018-12-21 Added failOnAutomaticEstimationFailure
- 2018-12-19 Added Android SDK

## Install
### iOS
Use cocoapods.

```
pod 'BodyBankEnterprise'
```

### Android
Use gradle.

```
allProjects {
    repositories {
        mavenCentral()
    }
}

dependencies{
    implementation 'com.bodybank:bodybank_enterprise:0.0.+'
}
```

### Javascript
Use npm/yarn. (Not distributed yet)

```
yarn add bodybank-enterprise
```

## Usage


### Set up
#### iOS
Include the `bodybank-config.json` file into main bundle by dragging the file into project navigator on Xcode.
The file is provided after making a contract.

#### Android
Include the `bodybank_config.json` file into raw resource by locating the file in the directory.
The file is provided after making a contract.


#### Javascript
Place the `bodybank-config.json` file into an accessible directory and provide the path in initializer.

### Mobile SDK

#### Token Provider 
Instantiate the `DefaultTokenProvider` and set `restoreTokenBlock`.
This block is called whenever a token is nearly expired or expired.
#### iOS
```swift
let tokenProvider = DefaultTokenProvider()
tokenProvider.setRestoreTokenBlock(block: {[unowned self] callback in
    let token = BodyBankToken()
    //Refresh token and create a BodyBank token from jwt_token & identity_id
    //Then call a callback with (token, error)
    //For example, if you want to get a token using firebase function
    let functions = Functions.functions()
    functions.httpsCallable("getBodyBankJWTToken").call {(result, error) in
        if let error = error{
            print(error.localizedDescription)
            callback(nil, error)
        }else{
            var token = BodyBankToken()
            if let jwtToken = (result?.data as? Dictionary<String, Any>)?["jwt_token"] as? String{
                token.jwtToken = jwtToken
            }
            if let identityId = (result?.data as? Dictionary<String, Any>)?["identity_id"] as? String{
                token.identityId = identityId
            }
            callback(token, nil)
        }
    }
})

```

#### Android
```
 val tokenProvider =  DefaultTokenProvider()
 tokenProvider.restoreTokenBlock = { callback ->
            FirebaseFunctions.getInstance().getHttpsCallable("getBodyBankJWTToken").call()
                .addOnCompleteListener { task ->
                    task.exception?.let {
                        callback(null, Error(it))
                    } ?: {
                        val token = BodyBankToken()
                        task.result?.data?.let {
                            (it as? Dictionary<String, Any>)?.let {
                                (it["jwt_token"] as? String)?.let {
                                    token.jwtToken = it
                                }
                                (it["identity_id"] as? String)?.let {
                                    token.identityId = it
                                }
                            }
                        }
                        callback(token, null)
                    }()
                }
        }
```

#### DirectTokenProvider for development use only

Instantiate the `DirectTokenProvider` which is a descendent of `DefaultTokenProvider`
#### iOS
```swift
    let tokenProvider = DirectTokenProvider(apiUrl: "https://api.<SHORT IDENTIFIER>.enterprise.bodybank.com", apiKey: "API KEY")
    tokenProvider.tokenDuration =  86400
    tokenProvider.userId = "unique user id"
    try! BodyBankEnterprise.initialize(tokenProvider: tokenProvider)
```

#### Android
```
    let tokenProvider = DirectTokenProvider( apiUrl = "https://api.<SHORT IDENTIFIER>.enterprise.bodybank.com", apiKey = "API KEY")
    tokenProvider.tokenDuration =  86400
    tokenProvider.userId = "unique user id"
    BodyBankEnterprise.initialize(context = applicationContext, tokenProvider = tokenProvider)
```

This class calls direct http request to the api server to refresh the token.
Please be careful that this embeds API Key in the app is vulnerable to API Key leakage.
Please implment your own `TokenProvider` or `restoreTokenBlock` on `DefaultTokenProvider`

Everytime identityId changes, do
```swift
BodyBankEnterprise.clearCredentials()
```

#### SDK initialization
#### iOS
Initialize BodyBank in `AppDelegate.swift`
```
import BodyBankEnterprise

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool{
    let tokenProvider = DefaultTokenProvider()
    tokenProvider.setRestoreTokenBlock(block: {[unowned self] callback in
        let token = BodyBankToken()
        //Refresh token and specify jwt_token & identity_id
        callback(token, nil)
    })
    BodyBankEnterprise.initialize(tokenProvider: tokenProvider)
    return true
}
```

#### Android
Initialize BodyBank in `Application` subclass

```kotlin
class Application : MultiDexApplication() {

    override fun onCreate() {
        super.onCreate()
        MultiDex.install(this)
        BodyBankEnterprise.initialize(context = applicationContext, tokenProvider = DefaultTokenProvider())
        BodyBankEnterprise.defaultTokenProvider()?.restoreTokenBlock = { callback ->
            FirebaseFunctions.getInstance().getHttpsCallable("getBodyBankJWTToken").call()
                .addOnCompleteListener { task ->
                    task.exception?.let {
                        callback(null, Error(it))
                    } ?: {
                        val token = BodyBankToken()
                        task.result?.data?.let {
                            (it as? Dictionary<String, Any>)?.let {
                                (it["jwt_token"] as? String)?.let {
                                    token.jwtToken = it
                                }
                                (it["identity_id"] as? String)?.let {
                                    token.identityId = it
                                }
                            }
                        }
                        callback(token, null)
                    }()
                }
        }

    }
}

```
#### Modifying JWT Token after initialization
The active Default Token Provider can be fetched by
#### iOS/Android
```swift
BodyBankEnterprise.defaultTokenProvider()

```
Please modify jwt using this reference.

#### Create New BodyBank Estimation Request

#### iOS
```swift
var params = EstimationParameter()
params.frontImage = Estimation.shared.frontImage
params.sideImage = Estimation.shared.sideImage
params.heightInCm = Estimation.shared.heightInCm
params.weightInKg = Estimation.shared.weight!
params.failOnAutomaticEstimationFailure = true
params.age = 30
params.gender = .male
BodyBankEnterprise.createEstimationRequest(estimationParameter: params, callback: { (request, errors) in
    if error == nil{
        print(request?.id)
    }else{
        print(error)
    }
})
```

#### Android

```kotlin
var params = EstimationParameter()
params.frontImage = Estimation.shared.frontImage
params.sideImage = Estimation.shared.sideImage
params.heightInCm = Estimation.shared.heightInCm
params.weightInKg = Estimation.shared.weight!
params.age = 30
params.gender = Gender.male
BodyBankEnterprise.createEstimationRequest(params) { request, errors ->
    errors?.let{
       print(it)
    } ?:{
        print(request)
    }()
}
```
#### Get BodyBank Estimation Requests
#### iOS
```swift
// List estimation requests
BodyBankEnterprise.listEstimationRequests(limit: 20, nextToken: nil, callback: { (requests, nextToken, errors) in
    print(requests)
})

// Get estimation request and result detail
BodyBankEnterprise.getEstimationRequest(id: id, callback: { (request, errors) in
    print(request)
})

```

#### Android
```kotlin
BodyBankEnterprise.listEstimationRequests(null, null, { requests, nextToken, errors ->
    print(requests)
})
        
BodyBankEnterprise.getEstimationRequest(id) { request, errors ->
    print(request)
}
```


#### Subscribe & Unsubscribe Changes

#### iOS
```swift
BodyBankEnterprise.subscribeUpdateOfEstimationRequests(callback: { (request, errorss) in
    print(errors)
})


BodyBankEnterprise.unsubscribeEstimationRequests()

```
#### Android
```kotlin
BodyBankEnterprise.subscribeUpdateOfEstimationRequests(callback = {request, error ->
            print(request)
        })

BodyBankEnterprise.unsubscribeUpdateOfEstimationRequests()
```


#### Get images

#### iOS
```swift
let url = request.frontImage?.downloadableURL
//Do anything with pre-signed downloadable URL
//It has expiration time.

```

#### Android
```kotlin
request?.frontImage?.downloadableURL?.let {
   //Do anything with pre-signed downloadable URL
   //It has expiration time.
}
```



