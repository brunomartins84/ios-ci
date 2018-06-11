# ios-ci
# Continuous Integration using Jenkins to build iOS applications

# 1) Execute Shell
export NEW_BRANCH=`echo $GIT_BRANCH | awk -Forigin/ '{print $2}'`<br />
git clone -b $NEW_BRANCH http://git.example.com.br/yourproject/ios.git<br />
xcodebuild -workspace /tmp/yourproject/yourproject.xcworkspace -scheme yourscheme -archivePath /tmp/yourproject.xcarchive archive<br />
xcodebuild -exportArchive -archivePath /tmp/yourproject.xcarchive -exportPath /Users/bruno/jobs/yourproject/builds/$BUILD_NUMBER/archive/yourproject/ -exportOptionsPlist /Users/bruno/Downloads/novo-plist/ExportOptions.plist<br />
mv /Users/bruno/jobs/yourproject/builds/$BUILD_NUMBER/archive/yourproject/yourproject.ipa /Users/bruno/jobs/yourproject/builds/$BUILD_NUMBER/archive/yourproject/$BUILD_NUMBER-yourproject.ipa

# 2) Create and Upload Text File (Plugin)
## File Path
/Users/bruno/jobs/yourproject/builds/$BUILD_NUMBER/archive/yourproject/$BUILD_NUMBER-manifest.plist

## Text File Content
```<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>assets</key>
            <array>
                <dict>
                    <key>kind</key>
                    <string>software-package</string>
                    <key>url</key>
                    <string>https://s3.amazonaws.com/yourproject/$BUILD_NUMBER-yourproject.ipa</string>
                </dict>
            </array>
            <key>metadata</key>
            <dict>
                <key>bundle-identifier</key>
                <string>yourbundleidentifiername</string>
                <key>bundle-version</key>
                <string>yourbundleversion</string>
                <key>kind</key>
                <string>software</string>
                <key>title</key>
                <string>yourproject</string>
            </dict>
        </dict>
    </array>
</dict>
</plist>
```

# 3) Execute Shell
export AWS_ACCESS_KEY_ID=yourkeyid
export AWS_SECRET_ACCESS_KEY=youraccesskey
export AWS_DEFAULT_REGION=yourregion
/usr/local/bin/aws s3 cp /Users/bruno/jobs/yourprojectbuild/builds/$BUILD_NUMBER/archive/yourproject/$BUILD_NUMBER-manifest.plist s3://yourproject/
/usr/local/bin/aws s3 cp /Users/bruno/jobs/yourprojectbuild/builds/$BUILD_NUMBER/archive/yourproject/$BUILD_NUMBER-yourproject.ipa s3://yourproject/


# 4) Execute Shell
## Open project directory
cd /Users/bruno/workspace/yourproject

## Last commit variable
export CHANGE_LOG="$(git log -1 --pretty=format:%s $GIT_COMMIT)"

## Defining new variables
export NEW_BRANCH=`echo $GIT_BRANCH | awk -Forigin/ '{print $2}'`<br />
export DATE_TIME=$(date +%d-%m-%Y--%T)

## Post method using curl
curl -H "Content-Type: application/json" -X POST -d '{"name":"'$BUILD_NUMBER'", "link":"itms-services://?action=download-manifest&url=https://s3.amazonaws.com/yourproject/'$BUILD_NUMBER'-manifest.plist","branch":"'"$NEW_BRANCH"'", "production": false, "platform":"ios", "changelog":"'"$CHANGE_LOG"'", "date":"'$DATE_TIME'"}' http://yourprojects.com.br/

# 4) Post-Build Actions
`**/*.*`

brunomartins84[at]yahoo[dot]com[dot]br
