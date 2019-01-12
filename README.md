# BodyBank Enterprise SDK Readme

## Version
2018-12-21-1
Android SDK Version 0.0.13

## Update History
- 2018-12-19 Added Android SDK

## Install

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

## Usage


### Set up

#### Android
Include the `bodybank_config.json` file into raw resource by locating the file in the directory.
The file is provided after making a contract.

### Mobile SDK

#### Token Provider 
Instantiate the `DefaultTokenProvider` and set `restoreTokenBlock`.
This block is called whenever a token is nearly expired or expired.


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
#### Android
```kotlin
BodyBankEnterprise.defaultTokenProvider()

```
Please modify jwt using this reference.

#### Create New BodyBank Estimation Request

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

#### Android
```kotlin
BodyBankEnterprise.subscribeUpdateOfEstimationRequests(callback = {request, error ->
            print(request)
        })

BodyBankEnterprise.unsubscribeUpdateOfEstimationRequests()
```


#### Get images

#### Android
```kotlin
request?.frontImage?.downloadableURL?.let {
   //Do anything with pre-signed downloadable URL
   //It has expiration time.
}
```



