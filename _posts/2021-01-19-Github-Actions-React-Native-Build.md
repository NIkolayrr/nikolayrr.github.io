---
layout: post
title: Simple CI/CD using github actions.
---

The goal is to create an android build for our React Native application everytime we push to master.<br>
To use github actions you first need to create .github/workflows folders into the root of you application
and add a `build.yml` file inside.

![image](/assets/folder-structure.png)

```
// build.yml
name: build release
on:
  push:
    branches:
      - master
jobs:
  install-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install npm dependencies
        run: |
          npm install --no-optional
  build-android:
    needs: install-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        id: build_android
      - name: Install npm dependencies
        run: |
          npm install --no-optional
      - name: Build Android Release
        run: |
          cd android && ./gradlew assembleRelease
      - name: Upload Artifact
        uses: actions/upload-artifact@v1
        with:
          name: app-release.apk
          path: android/app/build/outputs/apk/release/
```

For mac users, a potential error you might encounter is `error fsevents@2.2.0: The platform "linux" is incompatible with this module.` To resolve this you will need to add

```
"optionalDependencies": {
    "fsevents": "*"
 }
```

into your `package.json` and run `npm install --no-optional` in your `build.yml` file. <br>
The end result should be a build in the Artifacts which you can later use when creating a relase.
![image](/assets/artifact.png)
