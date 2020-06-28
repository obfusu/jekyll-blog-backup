---
layout: post
title: Xposed - Spoofing IMEI
author: 77ganesh
tags:
---
<!-- [text](url) img->![]() -->

This is a tutorial to spoof your IMEI on your Android phone using Xposed framework.

<br>

### What is Xposed?
Xposed is a framework that lets you modify app behaviour without actually modifying the original app.
You can read more about from below

1. [Xposed](https://forum.xda-developers.com/xposed/xposed-installer-versions-changelog-t2714053)
2. [EdXposed](https://forum.xda-developers.com/xposed/development/official-edxposed-successor-xposed-t4070199)

<br>

### Installing
The recommended way is to install magisk (root), and follow instructions on EdXposed page.
There is a way to use this without root, but I have not tried it yet.

<strong style="color:red">!! DISCLAIMER !!</strong> <br>
Both rooting and using xposed can be dangerous if you don't know what you are doing. Spoofing IMEI can also break many apps as many apps unnecessarily collect and use this to uniquely identify you without your knowledge. I'm not responsible for anything.

<br>

### Getting started

This tutorial is for the impatient ones, and we will be focussing on getting up to speed without explaining in detail why something is done. If you are looking for more explanations, you can checkout [this excellent tutorial](https://github.com/rovo89/XposedBridge/wiki/Development-tutorial) by rovo89, the original author of Xposed.

Prerequisites:
1. Phone with EdXposed already installed
2. Android Pie (API 28)
3. Android Studio & Android SDK

Steps:

1. Open Android Studio -> Create New Project
2. Choose "No Activity"
3. Give desired app name and package name. Lets assume com.sample.spoofer is the package name.
4. In build.gradle (Module: app), add repo
```gradle
repositories {
  jcenter();
}
```
5. In same build.gradle, add dependency
```gradle
provided 'de.robv.android.xposed:api:82'
```
6. Open app->manifests->AndroidManifest.xml
7. Add xposed meta tags. Example manifest after adding

    ```xml
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.sample.spoofer">

        <application
            android:icon="@mipmap/ic_launcher"
            android:label="Imei Spoofer" >
            <meta-data
                android:name="xposedmodule"
                android:value="true" />
            <meta-data
                android:name="xposeddescription"
                android:value="Example to spoof imei" />
            <meta-data
                android:name="xposedminversion"
                android:value="82" />
        </application>
    </manifest>
    ```

8. In app->java->com.sample.spoofer, right click and create a new Java class. Lets name this ImeiSpoofer.
9. Paste below code

    ```java
    package com.sample.spoofer;

    import de.robv.android.xposed.IXposedHookLoadPackage;
    import de.robv.android.xposed.XC_MethodHook;
    import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;
    import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;

    public class ImeiSpoofer implements IXposedHookLoadPackage {
        public void handleLoadPackage (final LoadPackageParam lpparam) throws Throwable {
            if (lpparam.packageName.equals("com.android.settings"))
                return;

            XC_MethodHook returnEmptyString = new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    param.setResult("");
                }
            };
            findAndHookMethod("android.telephony.TelephonyManager", lpparam.classLoader, "getImei", returnEmptyString);
            findAndHookMethod("android.telephony.TelephonyManager", lpparam.classLoader, "getImei", int.class, returnEmptyString);
        }
    }
    ```

10. Build the project and we are done. The output apk will be at app->build->outputs->apk folder.

<br>
### Brief Explanation of the code

As the name suggests, implmenting `IXposedHookLoadPackage` will hook your code with android's package loading code.

While loading package, we use `findAndHookMethod` function to find a specific function in a class, and hook our methods.

Here we have attached hooks to `getImei` function of `android.telephony.TelephonyManager` class which is used for getting IMEI.

We have implemented `beforeHookedMethod`. So our code will be called before actual getImei call of TelephonyManager. We use `param.setResult()` to send our spoofed result and the original return value of getImei won't matter. We can also use `XC_MethodReplacement` instead of `XC_MethodHook` as we just want to replace the function.

Detailed API info of Xposed framework can be found [here](https://api.xposed.info/reference/packages.html)

<br>

### Installing

1. Install your output apk
2. Goto EdXposed Manger -> Modules
3. Enable your module
4. Do soft reboot or full reboot
5. Install any device info app and check your IMEI


Enjoy!

<br>