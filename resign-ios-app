#!/usr/bin/env bash
set -e

# Resigns an iOS app with a given provisioning profile, distribution certificate, and, optionally, a new bundle identifier.
# This script expects to be invoked with at least three arguments.
#   $1 is expected to be the path of the .ipa to be resigned
#   $2 is expected to be the path of the .mobileprovision to resign $1 with
#   $3 is expected to be the name of the distribution certificate in the Keychain to resign $1 with
#   $4 is optional. If included, this will be the new bundle identifier of the resigned app. If not included, the original bundle identifier will be used.

greenColor='\033[0;32m'
redColor='\033[0;31m'
noColor='\033[0m'
ipa=$1
provisioningProfile=$2
distributionCertificate=$3
bundleIdentifier=$4

if [[ $# -lt 3 ]]; then
    echo -e "${redColor}Expected at least three arguments.${noColor}"
    echo -e "${redColor}Something like ${noColor}'AnApp.ipa' 'AProvisioningProfile.mobileprovision' 'iPhone Distribution: AnOrganization'${redColor} where${noColor}"
    echo -e "  ${redColor}\$1 is an .ipa file${noColor}"
    echo -e "  ${redColor}\$2 is a .mobileprovision${noColor}"
    echo -e "  ${redColor}\$3 is the name of a distribution certificate in the Keychain${noColor}"
    echo -e "${noColor}You may pass a fourth argument to serve as a new bundle identifier for the resigned app. If not included, the original bundle identifier will be used."
    exit 1
fi

echo "Unzipping \"$ipa\"..."
unzip -q "$ipa"

echo "Extracting .app..."
cd Payload
declare -a appsInPayload=()
appsInPayload=$(find . -name '*app' | sed 's|^\./||')
cd ..
if [[ ${#appsInPayload[*]} -gt 1 ]]; then
  echo "There is more than one .app in Payload/ (${appsInPayload[*]})"
  echo -e "${redColor}Resigning failed${noColor}"
  exit 1
fi
app=$appsInPayload

echo "Removing existing _CodeSignature..."
rm -r -f Payload/"$app"/_CodeSignature

echo "Decoding \"$provisioningProfile\" and writing to ProvisioningProfile.plist..."
security cms -D -i "$provisioningProfile" > ProvisioningProfile.plist 2>&1

echo "Extracting \"Entitlements\" from ProvisioningProfile.plist and writing to Entitlements.plist..."
/usr/libexec/PlistBuddy -x -c 'Print Entitlements' ProvisioningProfile.plist > Entitlements.plist 2>&1

echo "Embedding \"$provisioningProfile\" into \"$app\"..."
cp "$provisioningProfile" Payload/"$app"/embedded.mobileprovision
if [[ -n $bundleIdentifier ]]; then
  echo "Resigning \"$app\" with bundle identifier \"$bundleIdentifier\""
  /usr/libexec/PlistBuddy -x -c "Set :CFBundleIdentifier $bundleIdentifier" Payload/"$app"/Info.plist
else
  echo "Resigning \"$app\" with existing bundle identifier \"$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' Payload/"$app"/Info.plist)\""
fi

if [[ -e Payload/$app/Frameworks ]]; then
  cd Payload/"$app"/Frameworks
  echo "Resigning embedded Swift libraries..."
  swiftLibraries=$(find . -name '*dylib')
  SDK_PATH="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/swift/iphoneos/"
  for dylib in $swiftLibraries; do
    codesign -f -s "$distributionCertificate" "$dylib"
  done
  frameworks=$(find . -name '*framework')
  echo "Resigning embedded frameworks..."
  for framework in $frameworks; do
    codesign -f -s "$distributionCertificate" "$framework"
  done
  cd ../../..
fi

echo "Resigning \"$app\" with certificate \"$distributionCertificate\"..."
codesign -f -s "$distributionCertificate" --entitlements Entitlements.plist Payload/"$app"

echo "Zipping app data into an .ipa..."
zip -q -r resigned.ipa Payload SwiftSupport Symbols

echo -e "${greenColor}Resigning successful. Resigned app to $PWD/resigned.ipa${noColor}"
