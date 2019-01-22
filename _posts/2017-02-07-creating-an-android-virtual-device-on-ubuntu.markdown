---
layout: post
title: "Creating an Android Virtual Device on Ubuntu 16.04 LTS"
date: 2017-02-07 07:28:29
---

I stumbled upon the [following issue](https://code.google.com/p/android/issues/detail?id=82711) in Android Studio on Ubuntu:

> "The build tools appear to be shipped as 32-bit Linux binaries. 64-bit Linux distros do not necessarily ship the required bits anymore to run 32-bit Linux binaries." — Anonymous bug reporter

After having a look at several answers and packages for the problem ([here](http://stackoverflow.com/questions/3878445/ubuntu-error-failed-to-create-the-sd-card), [here](http://stackoverflow.com/questions/29241640/error-unable-to-run-mksdcard-sdk-tool), and [here](http://askubuntu.com/questions/107230/what-happened-to-the-ia32-libs-package)), I [found the package](http://stackoverflow.com/questions/35555319/installing-android-studio-in-ubuntu-unable-to-run-mksdcard-sdk-tool) that actually worked for me:

```python
sudo apt-get install lib32stdc++6
```

Ta-da!

Edit: A [quick solution](http://stackoverflow.com/questions/36258908/cannot-launch-avd-in-emulator-output-sh-1-glxinfo#36316787) for the missing `glxinfo` issue:

```python
sudo apt-get update && sudo apt-get install mesa-utils
```

Also, a note to myself: don't forget to run Android Studio as a super user, otherwise the builds will fail due to insufficient user permissions!