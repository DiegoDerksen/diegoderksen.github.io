---
title: "Post: Debloating Nvidia Shield"
last_modified_at: 2024-04-22T21:00:00+02:00
categories:
  - Blog
tags:
  - Home Server
---

Looking to get more out of your Nvidia Shield TV? From improving speed to customizing your interface, there are ways to enhance your experience. In this guide, we'll explore how to speed up your device and install a custom launcher to hide ads, helping you make the most of your entertainment hub. Let's dive in and optimize your Shield TV.

# Prerequisites
* PC (I'll be using MacOS)
* Nvidia Shield TV
* [ADB](https://www.xda-developers.com/install-adb-windows-macos-linux/#how-to-set-up-adb-on-your-computer)

## Install ADB on MacOS:
* Open Terminal and install Homebrew
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
* Install ADB
```
brew install android-platform-tools
```
## Install ADB on Windows
* Download [Android SDK Platform Tools ZIP file for Windows](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
* Extract the contents of this ZIP file into an easily accessible folder (such as C:\platform-tools)
  
# Steps
## 1. Enable ADB on the Shield
### 1a. Enable developer options
* Press Menu on the Android homescreen
* Go to "Device Preferences"
* Locate "About"
* Click 8 times on "Build"
* You should be seeing "You are now a developer"

### 1b. Enable ADB over Wifi
* Press Menu on the Android homescreen
* Go to "Device Preferences"
* Go to to "Developer options"
* Inside developer options, turn on "Network debugging"
* Note the IP address of the Shield to access it through ADB

### 1c. Connect to shield through ADB
After installing ADB on your PC/Mac (If you're on Windows go to the extracted ADB folder open up a Command Prompt/Terminal instance from the same directory as this ADB executable. This can be done by holding Shift and right-clicking within the folder and then clicking Open command window here or Open PowerShell window here)
* Use the terminal/cmd to type the following command
```
adb connect (IP-of-your-Shield)
```
* You'll see a new pop-up on the Shield to allow ADB/Network Debugging. Click on "OK"
  
To confirm that you're connected to the Shield, use the following command
```
adb devices
```

## 2. Installing a custom launcher
* Go to the Google Play Store on the Shield
* Search ["Projectivy Launcher"](https://play.google.com/store/apps/details?id=com.spocky.projengmenu)
* Click on Install and open the launcher
* Allow all permissions when it asks and allow in accessibility settings
* We are going to disable the default launcher in step 3.

## 3. Debloating
Now comes the fun part. Removing all the useless and unwanted apps pushed by Nvidia and Google.
Inside the Terminal/CMD while being connected to your shield with ADB copy and paste the following code blocks

**Remove Nvidia bloat**
```bash
adb shell pm uninstall -k --user 0 com.nvidia.ota & adb shell pm uninstall -k --user 0 com.nvidia.ocs & adb shell pm uninstall -k --user 0 com.nvidia.diagtools & adb shell pm uninstall -k --user 0 com.nvidia.shieldtech.hooks & adb shell pm uninstall -k --user 0 com.nvidia.feedback & adb shell pm uninstall -k --user 0 com.nvidia.shield.registration & adb shell pm uninstall -k --user 0 com.nvidia.SHIELD.Platform.Analyser & adb shell pm uninstall -k --user 0 com.nvidia.shield.remote.server & adb shell pm uninstall -k --user 0 com.nvidia.shieldtech.proxy & adb shell pm uninstall -k --user 0 com.nvidia.factorybundling & adb shell pm uninstall -k --user 0 com.nvidia.shieldbeta & adb shell pm uninstall -k --user 0 com.nvidia.shield.registration & adb shell pm uninstall -k --user 0 com.nvidia.shield.nvcustomize & adb shell pm uninstall -k --user 0 com.nvidia.NvCPLUpdater & adb shell pm uninstall -k --user 0 com.nvidia.benchmarkblocker & adb shell pm uninstall -k --user 0 android.autoinstalls.config.nvidia & adb shell pm uninstall -k --user 0 com.nvidia.hotwordsetup & adb shell pm uninstall -k --user 0 com.nvidia.enhancedlogging & adb shell pm uninstall -k --user 0 com.nvidia.shield.ask & adb shell pm uninstall -k --user 0 com.nvidia.stats & adb shell pm uninstall -k --user 0 com.nvidia.shield.appselector & adb shell pm uninstall -k --user 0 com.nvidia.beyonder.server & adb shell pm uninstall -k --user 0 com.nvidia.developerwidget & adb shell pm uninstall -k --user 0 com.nvidia.NvAccSt & adb shell pm uninstall -k --user 0 com.nvidia.shield.remotediagnostic
```
**Remove Google bloat** 
```bash
adb shell pm uninstall -k --user 0 com.google.android.speech.pumpkin & adb shell pm uninstall -k --user 0 com.google.android.tts & adb shell pm uninstall -k --user 0 com.google.android.videos & adb shell pm uninstall -k --user 0 com.google.android.tvrecommendations & adb shell pm uninstall -k --user 0 com.google.android.syncadapters.calendar & adb shell pm uninstall -k --user 0 com.google.android.backuptransport & adb shell pm uninstall -k --user 0 com.google.android.partnersetup & adb shell pm uninstall -k --user 0 com.google.android.inputmethod.korean & adb shell pm uninstall -k --user 0 com.google.android.inputmethod.pinyin & adb shell pm uninstall -k --user 0 com.google.android.apps.inputmethod.zhuyin & adb shell pm uninstall -k --user 0 com.google.android.tv & adb shell pm uninstall -k --user 0 com.google.android.tv.frameworkpackagestubs & adb shell pm uninstall -k --user 0 com.google.android.tv.bugreportsender & adb shell pm uninstall -k --user 0 com.google.android.backdrop & adb shell pm uninstall -k --user 0 com.google.android.leanbacklauncher.recommendations & adb shell pm uninstall -k --user 0 com.google.android.feedback
```
**Notice:** Only disable the following if another launcher is installed.
{: .notice--danger}

```
adb shell pm disable -k --user 0 com.google.android.tvlauncher
adb shell pm disable -k --user 0 com.google.android.leanbacklauncher
```
**Remove Android bloat**
```bash
adb shell pm uninstall -k --user 0 com.android.gallery3d & adb shell pm uninstall -k --user 0 com.android.dreams.basic & adb shell pm uninstall -k --user 0 com.android.printspooler & adb shell pm uninstall -k --user 0 com.android.feedback & adb shell pm uninstall -k --user 0 com.android.keychain & adb shell pm uninstall -k --user 0 com.android.cts.priv.ctsshim & adb shell pm uninstall -k --user 0 com.android.cts.ctsshim & adb shell pm uninstall -k --user 0 com.android.providers.calendar & adb shell pm uninstall -k --user 0 com.android.providers.contacts & adb shell pm uninstall -k --user 0 com.android.se
```
## 3b. Removing more bloat
If you're like me and use Jellyfin or Plex and don't use the device to play games, you can also remove the following packages

**Remove Plex Media Server**
```bash
adb shell pm uninstall -k --user 0 com.plexapp.mediaserver.smb
```
**Remove Netflix**
```bash
adb shell pm uninstall -k --user 0 com.netflix.ninja
```
**Remove Amazon**
```bash
adb shell pm uninstall -k --user 0 com.amazon.amazonvideo.livingroom
adb shell pm uninstall -k --user 0 com.amazon.amazonvideo.livingroom.nvidia
```
**Remove YouTube Music**
```bash
adb shell pm uninstall -k --user 0 com.google.android.youtube.tvmusic
```
**Remove Google Play Games**
```bash
adb shell pm uninstall -k --user 0 com.google.android.play.games
```
**Remove Nvidia GeForce NOW for Shield TV**
```bash
adb shell pm uninstall -k --user 0 com.nvidia.tegrazone3
```
### Re-install packages
If you removed something on accident or want to get an app back, use the following command
```bash
adb shell cmd package install-existing [package name]
```
--------------------------------

I hope this guide has provided you with the tools and knowledge to revitalize your Nvidia Shield, transforming it into a faster and more efficient device. May your entertainment experiences be smoother and more enjoyable than ever before.