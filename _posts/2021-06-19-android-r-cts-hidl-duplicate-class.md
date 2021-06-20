---
layout: post
title: "Investigation on Android R Hidl duplicate class issue"
date: 2021-06-19 13:43:51
categories: Android
tags: Android Ninja
---

I work for a phone manufacturer. Recently the ROM team upgrade AOSP version from Android Q to Android R. After the upgrade, a tester contacted me complaining a duplicate class cts issue. I spend a few days investigating it. Hope this blog whould help any person who encounters a similar one.





* content
{:toc}

# Investigation on Android R Hidl duplicate class issue

I work for a phone manufacturer. Recently the ROM team upgrade AOSP version from Android Q to Android R. After the upgrade, a tester contacted me complaining a duplicate class cts issue. I spend a few days investigating it. Hope this blog whould help any person who encounters a similar one.

The module name has been modified to stay anonymous.

## 1. What's the issue?

I maintains a hidl interface `tianshi.hardware.fun.V1_0.IMoreFun`and its implementation, which extends an AOSP hidl `android.hardware.fun.V1_0.IFun`. Since my company has a policy of make minimum modifications to AOSP code, `IMoreFun` is packed in another jar `tsServcies` rather than the AOSP `services`. 

The CTS issue is `android.hardware.fun.V1_0.IFun` is packed twice in `tsServcies` and `services`.

```java
services
  android.hardware.fun.V1_0.IFun
tsServices
  android.hardware.fun.V1_0.IFun // duplicate class
  tianshi.hardware.fun.V1_0.IMoreFun
```

## 2. Solution

This problem can be solved modify `IMoreFun` declaration in `tsServices` Android.bp from `java` to `java-shallow`.

```json
java_library {
    name: "tsServices",
    installable: true,

    srcs: [
        "java/main.java",
    ],

    static_libs: [
      	// before fix
        // "tianshi.hardware.fun-V1.0-java"
        "tianshi.hardware.fun-V1.0-java-shallow"
    ],

    libs: [
        "android.hidl.manager-V1.0-java",
        "framework-tethering.stubs.module_lib",
    ]
}
```

Explanation:

- `tianshi.hardware.fun-V1.0-java` adds object and all its dependency to `tsServices`
- `tianshi.hardware.fun-V1.0-java-shallow`adds object, but no dependency to `tsServices`

The shallow version of hidl is introduced in Android 10, see[HIDL JAVA Overview](https://source.android.google.cn/devices/architecture/hidl-java). 

## 3. My wrong attempt to dismiss this issue

If solve the problem is what you need. You can skip this and fellowing sessions.

I spend a few days investigating this issue. I organised my notes here.

### 3.1 Why this issue occurs after Android R

As explained in [this blog](https://blog.csdn.net/u014175785/article/details/115333461), in android R, AOSP fixed a bug, only checking the first dex in jar. After the fix, all dexs in jar is checked.

### 3.2 Does this issue has any functional effect?

No.

As explained in [this blog](https://juejin.cn/post/6844903541987868680), `/system/framework` jars are preloaded when system server process starts and shared accross all processes. When user need to use these classes,  dalvik loads and returns the class from the first dex that has the class.

Since `IFun` in `services` and `tsServices` are converted from the same hidl. the class is same. Duplicate class wouldn't cause any issue.

### 3.3 How to check if `IMoreFun` inducts `IFun` to `tsServices`

```bash
$ alias ninja=/home/tianshi/Data/aosp/prebuilts/build-tools/linux-x86/bin/ninja
$ ninja -f ./out/combined-aosp_x86_64.ninja -t commands tsServices | grep merge_zips | grep IFun
```

### 3.4 Obtain compile commands for a target

```bash
$ alias ninja=/home/tianshi/Data/aosp/prebuilts/build-tools/linux-x86/bin/ninja
$ ninja -f ./out/combined-aosp_x86_64.ninja -t commands tsServices 
```

Under condition your build script is not changed,  these commands can be used for fast build.

### 3.5 List all targets related to a target

```bash
$ alias ninja=/home/tianshi/Data/aosp/prebuilts/build-tools/linux-x86/bin/ninja
$ ninja -f ./out/combined-aosp_x86_64.ninja -t targets | grep hardware.fun | grep -v -E '(vts|out)' | grep java
android.hardware.fun-V1.0-java: phony
android.hardware.fun-V1.0-java-shallow: phony
android.hardware.fun-V1.0-java-shallow-soong: phony
android.hardware.fun-V1.0-java-shallow-target: phony
android.hardware.fun-V1.0-java-soong: phony
android.hardware.fun-V1.0-java-target: phony
```

PS: the issue is fixed the moment I see `java-shallow`.

### 3.6 Obtain Classs list in a jar/dex

Build and use hexdump tool.

```bash
$ alias dexdump=/home/tianshi/Data/aosp/out/soong/host/linux-x86/bin/dexdump
$ dexdump classes.dex  | grep 'Class descriptor' 
  Class descriptor  : 'Ltianshi/hardware/fun/V1_0/IMoreFun;'
  Class descriptor  : 'Ltianshi/hardware/fun/V1_0/IMoreFun$Proxy;'
  Class descriptor  : 'Ltianshi/hardware/fun/V1_0/IMoreFun$Stub;'
```



## 4. AAR (After Action Review)

I spends days trying to collect evidences to convince reviewer to dismiss this issue. Turns out it can be fixed only modify a few lines. :cry: I went a few wrong trails.

* I only checked soong build parameters of HIDL and java library. If I checked the way `tsService` references `IMoreFun`, I should have resolved this issue easily.
* `java-shallow` is not common target suffix. When normal target name can not slove the problem, I should check ninja target list, see if any target can be useful.
* Proving issue unfixable is much much costy than fix it. Do not declare any issue is unfixable unless you collect enough evidence.

## 5. Other Useful References

soong reference:

https://ci.android.com/builds/submitted/7475120/linux/latest/view/soong_build.html


Step-by-step guide of adding hidl:

https://blog.csdn.net/sinat_18179367/article/details/95940030

Android Class loader

https://github.com/huanzhiyazi/articles/issues/30
