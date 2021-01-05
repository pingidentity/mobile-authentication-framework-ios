
# Mobile Authentication Framework for iOS Developers

## Overview

This project is an open-source code that provides an example of how to use [PingIdentity Authentication API](https://docs.pingidentity.com/bundle/pingfederate-93/page/qsl1564002999029.html)  *browserless* OAuth flow in iOS projects. It supports selected MFA flows and provides the current state of a flow, as an end-user steps through a PingFederate authentication policy. This native iOS code allows end-user interactions with the API, such as credential prompts, to be handled by an external mobile application.
The repository contains two modules `PingAuthenticationCore` (*Core*) and `PingAuthenticationUI` (*UI*) where *Core* is an independent module and *UI* depends on it.

*Note: those modules are designed to work in conjunction with PingID SDK or PingOne SDK and will not work separately.*

### PingAuthenticationCore
This is the main module that is responsible for communication with the OpenID Connect issuer. It manages flow states and contains all the needed configurations in one place, making it easy to configure and work with. This module is independent, in that it can work without the `PingAuthenticationUI` module, which is actually an example of how to use public APIs of the *Core*.

### PingAuthenticationUI
This is an example module that demonstrates *one of the possible ways* to implement a user interface that will handle all the various states from `PingAuthenticationCore`. This is the reason that this module depends on `PingAuthenticationCore`.

Release notes can be found [here](./release-notes.md).

### Documentation

PingFederate reference documentation is available here: 
* [Introduction to PingFederate](https://docs.pingidentity.com/csh?Product=pf-latest&topicname=tle1564002955874.html)


### Prerequisites

1. In order to use the Mobile Authentication Framework you need to configure an app with Ping Identity SDK. For the full code sample of the Mobile Authentication Framework integrated with the sample app see [PingOne SDK Sample App](https://github.com/pingidentity/pingone-mobile-sdk-ios).

2. Prepare the PingFederate OAuth client (application):
* [Authentication API](https://docs.pingidentity.com/bundle/pingfederate-93/page/qsl1564002999029.html)
* [Installing PingFederate](https://docs.pingidentity.com/csh?Product=pf-latest&topicname=ptg1564002959252.html)
* [Managing OAuth clients](https://docs.pingidentity.com/csh?Product=pf-latest&topicname=hsx1564002992533.html)
* [Integration with the PingFederate Authentication API for PingID SDK](https://apidocs.pingidentity.com/pingid-sdk/guide/pf-adapter/pid_c_SDKadapterForPFauthenticationApi/)
* [Integration with the PingFederate Authentication API for PingOne MFA](https://docs.pingidentity.com/bundle/integrations/page/gaa1600114529505-5.html)

Note: The Mobile Authentication Framework for iOS supports different versions per platform:
* PingID: PingFederate 10.1, PingFederate PingID SDK IDP Adapter 1.8 and above.
* PingOne: PingFederate 10.1, PingOne MFA IdP Adapter 1.0 and above.

### Xcode integration

**Note:**  The Authentication API source code supports the following software versions:

* Xcode 11 and above.
* iOS 10.0 and above.

## Getting Started

1. Drag and drop the `PingAuthenticationCore` and `PingAuthenticationUI` folders into your project files in your app.
2. Check the **Copy items if needed** checkbox and make sure that all your targets are checked in **Add To Target**.
3. Set up the following parameter values in the `Config.swift` configuration class:
* `oidcIssuer`: The base URL for the PingFederate endpoint. 
* `clientId`: The ID of the client as configured in the PingFederate console.
* `responseType`: Valid values are "token" or "code". If it is "code", an extra API call generates the accessToken.
* `scope`: The adapter scope. By default it is set to "openid", and can be modified.
* `responseMode`: The adapter response mode. By default it is set to "pi.flow", and can be modified to support different modes.
* `clientSecret`: The client secret set in the PingFederate console. This is an optional value.
* `grantType`: The grant type. By default it is set to “authorization_code”, for the token exchange code request.

## Public API's

1. First you need to init an object of `PingAuthenticationUI.swift` in your app.
2. Call the `authenticate` method to start an authentication process with the PingFederate adapter.
3. If pairing is needed, the  `continueAuthentication` method must be called on your app, after the successful pairing process.
4. If you don't want to use the UI from this project, you can start an authentication process by calling the `authenticate` method from `PingAuthenticationCore.swift`.

### PingAuthenticationUI

```swift
/// Authenticate
///
/// Starts an authentication process with PingIdentity Authentication API. Calling this method opens a new UIViewController that handles the UI of the authentication process.
///
/// - Parameters:
///   - presenter: a UIViewController that retrieves the result of the authentication process. 
///   - payload: a mobilePayload String retrieved from PingID SDK or PingOne SDK.
///   - completionHandler:  returns serverPayload, accessToken and error.
/// - Returns:
///   - serverPayload: server payload String if pairing is needed.
///   - accessToken: accessToken String from the Authentication API.
///   - error: error object in cases of error.

public func authenticate(presenter: UIViewController, payload: String?, dynamicData: String?, completionHandler: @escaping (_ serverPayload: String, _ accessToken: String, _ error: Error?) -> Void)
```

* `presenter`: The UIViewController instance of the app.
* `payload`: Mobile payload created by the Mobile SDK in the app.
    ** Note: The payload requires the ENVIRONMENT ID, APPLICATION CLIENT ID and the APPLICATION CLIENT SECRET in the Ping web portal to be identical to the PingFederate adapter settings.
* `dynamicData`: String value of any data passed from the app to the Authentication API.
* `serverPayload`: Payload returned by the server in the Authentication API response. 
* `accessToken`: The token returned at the end of a successful authentication process.
* `error`: The error object returned in the authentication response in cases of error.

```swift
/// ContinueAuthentication
///
/// Continue an authentication process in the app and returned to the Authentication API instance to finalize the process.
///
/// - Parameters:
///   - presenter: a UIViewController that retrieves the result of an authentication process.

public func continueAuthentication(presenter: UIViewController)
```

* `presenter`: The UIViewController instance of the app.


### PingAuthenticationCore
**Please note, that if you are using PingAuthenticationUI module as-is you would never need to call this API directly**

```swift
/// Authenticate
///
/// Executes a request according to the RequestParams and delivers AuthenticationState. The authenticate process is set by a certain action and results with statuses from server.
///
/// - Parameters:
///   - requestParams: object that contains all the parameters needed to make an API call. Each action determines the relevant API request that is called in the LogicLayer.
///   - completionHandler: returns serverPayload, accessToken and error.
/// - Returns:
///   - response: response parameters object which contains the API response status and array of available actions.
///   - error: error object in cases of error.
/// - Note:
///   If you want to use your own UI, this method can be called independently, regardless of the PingAuthenticationUI instance.

static public func authenticate(_ requestParams: RequestParams, completionHandler: @escaping (_ response: AuthenticationState?, _ error: Error?) -> Void)
```

* `requestParams`: object that contains all the parameters needed to make an API call. Each action determines the relevant API request that is called in the LogicLayer.
* `authenticationState`: response parameters object which contains the API response status and array of available actions.
* `error`: The error object returned in the authentication response in cases of error.


## Disclaimer

THE SAMPLE CODE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SAMPLE CODE OR THE USE OR OTHER DEALINGS IN
THE SAMPLE CODE.  FURTHERMORE, THIS SAMPLE CODE IS NOT COMMERCIALLY SUPPORTED BY PING IDENTITY BUT QUESTIONS MAY BE ADDRESSED TO PING'S SUPPORT CENTER OR MAY BE OTHERWISE ADDRESSED IN THE RELATED DOCUMENTATION.

Any questions or issues should go to the support center, or may be discussed in the [Ping Identity developer communities](https://community.pingidentity.com/collaborate).
