# Change Log
All changes to the SDK will be documented in this file.
This project *tries* to follow the [Semantic Versioning](http://semver.org) style.

## [1.11.0] - 2018-01-22
### Remove
- Firebase services and dependency. Firebase is now optional

### Add
- `setPushToken(String)` method for accepting Firebase tokens

## [1.10.0] - 2018-01-16
### Add
- CampaignListener interface for responding to campaign interactions (this cancels the default campaign behavior)
- public logs for setters and campaign operations.
- Campaign relative path now affects cordova web-browser (works only with local file paths)

### Change
- Rename setListener to addListener

### Fix
- NPE occasionally when no internet connection
- Campaigns are now presented in cordova apps.

## [1.9.7] - 2018-05-01
### Fix
- NullPointerException on fast devices with no internet connection

## [1.9.6] - 2018-02-01
### Change
- Configo layover button for scanning screens is now posted to the UI Thread to avoid race conditions.

## [1.9.5] - 2017-12-26
### Add
- Setter logs
- Campaign logs
- Support for feature disabled state in lists and recyclers

### Fix
- Survey and feedback campaigns crashes

## [1.9.4] - 2017-12-14
### Added
- Survey campaigns
- Tooltip campaigns
- Feature type: disabled/hidden support
