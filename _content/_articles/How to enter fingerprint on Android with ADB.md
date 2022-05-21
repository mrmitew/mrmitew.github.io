---
layout: post
title: How to enter fingerprint on Android with ADB
author: Stefan Mitev
comments: false
date: 20.05.2022
permalink: "/articles/how-to-enter-fingerprint-on-android-with-adb"
tags:
- adb
- snippet
- android
---

Ever wondered how to enter fingerprint on Android using ADB? 

0. Setup your phone for Wi-Fi/USB debugging
1. Navigate to your fingerprint settings
2. Tap on "*Add fingerprint*"
3. Enter in a terminal/command prompt `adb -e emu finger touch <id>` \*
4. You might need to re-execute the command couple of times until the "fingerprint" is fully "scanned"

\* The `<id>` could be basically anything you like.

Example:
```bash
adb -e emu finger touch 123456
```
