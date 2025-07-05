# Build & Run App

- Clone the repository https://github.com/ditinexhosting/360hris-mobile-app
- Set env.json in root directory
- Run `npm install --force`

### Android

- Set `google-services.json` file under `andoid/app`
- Set `release.keystore` file under `android/app` (Optional : only required for production release)
- Run with `npm start`
- Run with `npm run android`

Other android release commands :

```
npm run android -- --mode="release"
npx react-native build-android --mode=release
```

### iOS

- Set `GoogleService-Info.plist` file under `ios/GoogleService-Info.plist`
- Install `certificate.p12` in local machine.
- Open Xcode > Signing & Capabilities, uncheck "Automatically manage signin" and install mobileprovision file.
- Run with `npm start`
- Run with `npm run ios`

Other commands :

```
xed -b ios
```

# DEPLOYMENT

## CI / CD Pipeline Firebase App Distribution

- Create Playstore & Applestore accounts with correct available package names.
- Go to Firebase console of client and create app 360hris
- Go to project setting and create android / ios apps. Make sure to provide correct package names.
- Create a service account to authenticate and upload apk/ipa files via github-action. [Reference](https://firebase.google.com/docs/app-distribution/authenticate-service-account?platform=android)

### Android Setup

- Create release keystore file

```
keytool -genkey -v -keystore your_keystore_name.jks -keyalg RSA -keysize 2048 -validity 10000 -alias your_alias_name
```

- Add secrets in github secret
- Use github-action to deploy

### iOS Setup

- Open Keychain Access in Mac (/Library/Keychains/System.keychain).
- Certificate Assistant > Request certificate from a certificate authority
- Create certificate then use it to create mobileprovision file in Apple Store Dashboard.
- Download the mobileprovision file. [Reference](https://medium.com/@fadilah.arifki29/deployment-ios-in-app-store-react-native-d845e99e8329)
- From Keychain, export .p12 certificate. [Refernce](https://www.andrewhoog.com/post/how-to-build-an-ios-app-with-github-actions-2023/#tip-finding-the-correct-signing-certificate-and-provisioning-profile)
- Convert env.json, certificate, GoogleService-Info.plist and provision file into base64 code to add in github secret. JSON/XML or any non string is not supported in pipeline while running in mac.
  `base64 -i Certificates.p12| pbcopy`
- Add the secrets in github secret
- User fastlane for build
- Use github-action to deploy

### Github Secret Keys

```
DEV_ANDROID_GOOGLE_SERVICE_JSON : Content of google-services.json
DEV_ANDROID_RELEASE_KEYSTORE_BASE64 : Content of release.keystore
DEV_ENV_BASE64 : Content of env.json in base64 format
DEV_FIREBASE_ANDROID_APP_ID : Can be found in Firebase project settings
DEV_FIREBASE_IOS_APP_ID : Can be found in Firebase project settings
DEV_FIREBASE_SERVICE_CREDENTIAL : Content of file hris-b6cd2-6713e9ef382a.json
DEV_IOS_GOOGLE_SERVICE_INFO_PLIST_BASE64 : Content of GoogleService-Info.plist in base64 format
DEV_IOS_MOBILE_PROVISION_BASE64 : Content of provision file in base64 format
DEV_IOS_P12_BASE64 : Content of Certificate.p12 in base64 format
```
