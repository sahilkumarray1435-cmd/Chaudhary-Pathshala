name: Build Pathshala APK
on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install Cordova
        run: npm install -g cordova

      - name: Install Android SDK
        run: |
          echo "y" | $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager \
            "platforms;android-33" \
            "build-tools;33.0.2" \
            "platform-tools" 2>/dev/null | tail -3

      - name: Create Cordova Project
        run: |
          cordova create PathshalaApp com.chaudhary.pathshala "Chaudhary Pathshala AI"
          cp config.xml PathshalaApp/config.xml
          cp -r www/* PathshalaApp/www/
          cd PathshalaApp
          cordova platform add android
          cordova plugin add cordova-plugin-inappbrowser
          cordova plugin add cordova-plugin-whitelist

      - name: Build APK
        run: |
          cd PathshalaApp
          cordova build android --release -- \
            --keystore=../android.keystore \
            --storePassword=pathshala123 \
            --alias=pathshala \
            --password=pathshala123 || \
          cordova build android

      - name: Find APK
        run: |
          find PathshalaApp -name "*.apk" -type f 2>/dev/null

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: ChaudharyPathshala-APK
          path: |
            PathshalaApp/platforms/android/app/build/outputs/apk/**/*.apk
          retention-days: 30
