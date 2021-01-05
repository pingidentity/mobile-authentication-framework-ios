# Release Notes

## v1.0.0 - Jan 5th, 2021
Features:

- iOS Swift code sends requests to the PingFederate Authentication API and responds to different statuses by showing possible end-user actions.
- Supports mobile device authorization and registration flows.

Known issues and limitations:

- **Untrusted accessing device with pushless app as authenticating device** (PingID SDK)<br>The flow of an untrusted accessing device while a pushless app is used as the authenticating device is not supported with the PingFederate Authentication API.
- **Disabling device authorization** (PingOne for Customers and PingOne MFA)<br>Disabling device authorization in the PingOne admin UI results in an error for iOS devices.
	
