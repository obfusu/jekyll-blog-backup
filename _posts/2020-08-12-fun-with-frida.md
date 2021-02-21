---
layout: post
title: Fun with Frida
author: 77ganesh
tags:
---

If you haven't met [frida](https://frida.re) yet, its about time you check it out. Frida is an excellent dynamic instrumentation toolkit that supports almost everything - Windows, macOS, Linux, iOS and Android.

### WTH is a dynamic instrumentation toolkit?
A dynamic instrumentation toolkit allows you to inspect and modify the behaviour of apps on the fly. This allows you reverse engineer the working of an app - like which backend APIs are called, what personal data is accessed, etc without modifying the app.

<strong style="color:red">!! DISCLAIMER !!</strong> <br>
Reverse engineering apps without permission is illegal. I'm not responsible for anything.

<br>

### With that set, lets get started
We are going to use frida to analyze an Android app

There are ways to use frida without root as mentioned in the [docs](https://frida.re/docs/android/), but we are going to assume a rooted device (or an emulator).

Steps

1. Install frida-cli on your system => `pip install frida-tools`
1. Download frida-server for your device from [releases](https://github.com/frida/frida/releases).
2. Push the binary to device using adb
3. Start frida-server as root on the device

For example, if your device arch is arm64, download frida-server-version-android-arm64.
Latest version at the time of writing is [frida-server-12.11.11-android-arm64](https://github.com/frida/frida/releases/download/12.11.11/frida-server-12.11.11-android-arm64.xz).
Then push the binary to device.

```bash
unxz frida-server.xz
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
```

I don't like running frida-server in the background, so I first spawn an adb shell
```
adb shell
```

Then inside adb shell
```
cd /data/local/tmp
./frida-server
```

Note: If you are using magisk, make sure you disable magisk hide. I wasn't able to get it working with magisk hide enabled.

<br>
### Lets get hacking

Now that both the cli and server are setup, lets try out some commands. Open terminal in your system and type

```
frida-ps -U
```

This should give a list of running processes on the device. 

```frida -U com.sample.battery12345 -l net.js -q | grep java.net```
