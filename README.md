# resign-ios-app
This is a bash script to resign an iOS app with a given provisioning profile,
distribution certificate, and, optionally, a new bundle identifier.

## Why?
You may find yourself needing to resign an app due to an expired provisioning
profile or new bundle identifier but do not have access to the project files
(`.xcodeproj` or `.xcworkspace`).

Or maybe you do have access to the project files, but your environment's version
of Xcode cannot build the project within the GUI for whatever reason.

## Assumptions
You have
1. A Mac with Xcode installed
2. An iOS app file (`.ipa`)
3. A valid provisioning profile you wish to resign the app with
(`.mobileprovision`)
4. A valid distribution certificate for the above provisioning profile along
with the private key used to generate the certificate in your Mac's Keychain

## Installation
Clone this repository, navigate into the folder containing the script, and
change the permissions of the script to allow it be executable:
```
$ chmod 755 resign-ios-app
```

## Usage
Invoke the script without any arguments to see an instructional message:
```
$ ./resign-ios-app
Expected at least three arguments.
Something like 'AnApp.ipa' 'AProvisioningProfile.mobileprovision' 'iPhone Distribution: AnOrganization' where
  $1 is an .ipa file
  $2 is a .mobileprovision
  $3 is the name of a distribution certificate in the Keychain
You may pass a fourth argument to serve as a new bundle identifier for the resigned app. If not included, the original bundle identifier will be used.
```

Assuming you have this script, the `.ipa`, and `.mobileprovision` all in the
same folder, and you're in that folder:
```
$ ./resign-ios-app 'AnApp.ipa' 'XC_iOS_comcompanyAnApp.mobileprovision' 'iPhone Distribution: AnOrganization'
Unzipping "AnApp.ipa"...
Extracting .app...
Removing existing _CodeSignature...
Decoding "XC_iOS_comcompanyAnApp.mobileprovision" and writing to ProvisioningProfile.plist...
Extracting "Entitlements" from ProvisioningProfile.plist and writing to Entitlements.plist...
Embedding "XC_iOS_comcompanyAnApp.mobileprovision" into "AnApp.app"...
Resigning "AnApp.app" with existing bundle identifier "com.company.AnApp"
Resigning embedded Swift libraries...
./libswiftCoreImage.dylib: replacing existing signature
./libswiftObjectiveC.dylib: replacing existing signature
./libswiftCore.dylib: replacing existing signature
./libswiftCoreGraphics.dylib: replacing existing signature
./libswiftUIKit.dylib: replacing existing signature
./libswiftMetal.dylib: replacing existing signature
./libswiftCoreData.dylib: replacing existing signature
./libswiftDispatch.dylib: replacing existing signature
./libswiftos.dylib: replacing existing signature
./libswiftCoreFoundation.dylib: replacing existing signature
./libswiftDarwin.dylib: replacing existing signature
./libswiftQuartzCore.dylib: replacing existing signature
./libswiftCoreAudio.dylib: replacing existing signature
./libswiftAVFoundation.dylib: replacing existing signature
./libswiftFoundation.dylib: replacing existing signature
./libswiftCoreMedia.dylib: replacing existing signature
./libswiftsimd.dylib: replacing existing signature
Resigning embedded frameworks...
./BarcodeScanner.framework: replacing existing signature
./SQLite.framework: replacing existing signature
Resigning "AnApp.app" with certificate "iPhone Distribution: AnOrganization"...
Payload/AnApp.app: replacing existing signature
Zipping app data into an .ipa...
Resigning successful. Resigned app to /Users/daniel/resign-ios-app/resigned.ipa
```

Upon a successful resigning, you'll notice some extra files in the folder:
```
$ ls -l
-rw-rw-r--@ 1 daniel  staff  11506405 Nov 19 15:57 AnApp.ipa
-rw-r--r--@ 1 daniel  staff       228 Feb  4  2019 AppThinning.plist
-rw-r--r--  1 daniel  staff       517 Nov 22 18:29 Entitlements.plist
-rw-r--r--@ 1 daniel  staff      7354 Nov 19 10:26 XC_iOS_XC_iOS_comcompanyAnApp.mobileprovision
drwxr-xr-x@ 3 daniel  staff        96 Feb  4  2019 Payload
-rw-r--r--  1 daniel  staff      3382 Nov 22 18:29 ProvisioningProfile.plist
-rw-r--r--  1 daniel  staff  11488043 Nov 22 18:29 resigned.ipa
```

`Payload` and the `.plist` files are artifacts of the script and can be deleted.
`resigned.ipa` should be good to deploy.

## Tested
macOS 10.15.7  
GNU bash, version 5.0.7(1)-release (x86_64-apple-darwin16.7.0)
