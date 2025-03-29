---
layout: default
title: Getting Started
permalink: /chapters/0_getting_started
---

# Configuring your environment

In order to get started reverse engineering (RE) websites, we will often need to bypass a number of detection vectors. 
There are a number of ways to create an undetected browser, in this guide we're going to use [Librewolf](https://librewolf.net/) which comes patched out of the box and only needs a few modifications to become fully undetected.

If you have read Blatzar's [scraping-tutorial](https://github.com/Blatzar/scraping-tutorial/tree/master), you will already be familiar with how to bypass devtools detections already, however we will be doing a few extra modifications based on additional detections I have seen in the wild.

### 1. Installing Librewolf
Head over to https://librewolf.net/ and install whichever version is compatible with your system.

### 2. Modifying your about:config
Once installed we are going to navigate to `about:config` in the search bar, this will bring up a page that looks like this:
![image](https://github.com/Ciarands/web-reversing/assets/74070993/01df44dc-601e-41c6-9392-d5d9d6beb951)
We are going to change some settings settings here:
  - `librewolf.console.logging_disabled` to `true`
  - `librewolf.debugger.force_detach` to `true`
  - `webgl.disabled` to `false`
  - `privacy.resistFingerprinting` to `false`
  - `devtools.toolbox.host` to `window`
  - `devtools.source-map.client-service.enabled` to `false`

### 3. Congratulations!
Congratulations, you now have an undetectable browser, you can use any website without arbitrary restriction.
You can test this on https://blog.aepkill.com/demos/devtools-detector/.
