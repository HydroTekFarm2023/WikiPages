****Add Platforms**:**
- npx cap add ios
- npx cap add android
Platforms have already been added as of August 20th, 2021

Go to android/app/build.gradle and update versionName and versionCode before building app

****To compile app**:**
1. ionic build
2. npx cap copy
3. ionic cap sync

**Open Android Studio:**
- npx cap open android