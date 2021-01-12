---
layout: post
title: How I started my journey in KDE (open-source) and stayed
description: Here is how I started in open source and made a difference
date: 2020-07-04
hero_image: /img/KDEConnect.jpg
hero_height: is-medium
hero_darken: true
image: /img/KDEConnect.jpg
tags: open_source KDEConnect android
canonical_url: https://www.anjani.live/2020/07/04/KDE-Contrib.html
---

Open-source software affects your life more than you know. It is a community purely driven by passion, hobbyists and 
many industry professionals who want to work for more choices for users when it comes to privacy, features and freedom.

I've been a hobbyist from my early school days. And now I've finally contributed code to a community that ships quality
open-source software. [The KDE Community](https://kde.org/) is 24 years old and currently works on more than [a thousand active 
projects](https://invent.kde.org/explore/groups) and I've made my way into [KDEConnect](https://kdeconnect.kde.org/).
In this post I've written about all the code I've contributed to this project.

In-short, __KDEConnect__ attempts to unify your Android device and your desktop if they are connected in the same local network, say
your home Wi-Fi. Some notable features are Sending files with ease, Notification sync, Clipboard sync, etc. 

### How did I get in?

To get into any open-source project, you need atleast these:-

* An open-source software that interests you (you really gotta love it).
* A hacker mentality.
* Know _git_.
* You know the techstack your project demands.
* You know how to talk professionally and politely.
* Attitude to take criticism.

You can acheive all of these with some time and work. I joined the Telegram group where all the devs hang out and discuss the
ongoing work and payed attention to what are the current issues with the project. The best first bet you can make for choosing work
is to find a small bug that is reproducable. It is tempting to suggest a major change in the project in the beginning but the chances are
you'll get ignored. I did this mistake. [Take a look](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/115).
So look for a bug in the issue tracker of the project and find one which is feasible to work on. You'll know it
when you see it.

### What I found

I found two issues. The first is not exactly an issue, rather a feature request. 

#### Adding support for quick actions in the Android app

It was a feature request but it was a really small addition and when I submitted it, the devs really liked it and paved way
for my future work. 

KDEConnect Android app has a persistent notification indicator but didn't really did anything besides telling if your phone is 
connected to another device which was not very useful. 

I added two action buttons in the notification, _Send Files_ and _Run Command_. So now, after my addition users could now 
use these two features without opening KDEConnect App!!! and all it really took was this much.

```java
 if (deviceIds.size() == 1) {
    // Adding two action buttons only when there is a single device connected.
    // Setting up Send File Intent.
    Intent sendFile = new Intent(this, SendFileActivity.class);
    sendFile.putExtra("deviceId", deviceIds.get(0));
    PendingIntent sendPendingFile = PendingIntent.getActivity(this, 1, sendFile, PendingIntent.FLAG_UPDATE_CURRENT);
    notification.addAction(0, getString(R.string.send_files), sendPendingFile);
 
    // Checking if there are registered commands and adding the button.
    Device device = getDevice(deviceIds.get(0));
    RunCommandPlugin plugin = (RunCommandPlugin) device.getPlugin("RunCommandPlugin");
    if (plugin != null && !plugin.getCommandList().isEmpty()) {
        Intent runCommand = new Intent(this, RunCommandActivity.class);
        runCommand.putExtra("deviceId", deviceIds.get(0));
        PendingIntent runPendingCommand = PendingIntent.getActivity(this, 2, runCommand, PendingIntent.FLAG_UPDATE_CURRENT);
        notification.addAction(0, getString(R.string.pref_plugin_runcommand), runPendingCommand);
    }
 }
```

And the result was this...

<p align="center">
  <img src="/img/KDEContrib/PR1.png" width="400" title="Me">
</p>

[Here is the full merge request](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/126).

#### Fixing Clipboard sync on Android 10

It is a very very very productive feature as the clipboard of your phone and desktop are kept in sync. It is extremely usefull
for links and small text sharing. It broke on Android 10 because Google restricted access of clipboard data from background.
Now, you can only access the clipboard while your app is in focus. This is a huge win for privacy but it broke a lot of apps
dependent on background clipboard access.

People found a good _hack_ around this. The _hack_ has two steps:-

* Launch an invisible activity when clipboard data changed.
* Read the data and close immediately.

This is so fast that it is not noticeable. To achieve step 1, the most viable solution at that time was to launch the invisible 
activity manually. Adding a third action button in the notification is handy and adds just one extra step in using this feature.

It required a lot of code and testing. [Here is the full merge request](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/127).

And the result is this...

<p align="center">
  <img src="/img/KDEContrib/PR2.png" width="400" title="Me">
</p>

This hack, is very cheeky!!! And it was a lot fun to see this feature working again. Everyone loved this.

### How I stayed

The more you write code, the more it needs maintainance and potentially introduce more bugs. I polished the Clipboard
sync feature more. I had forgotten to clear a trace of the _invisible activity_ from the app recents screen. It was easy to 
fix. Added this in the manifest.

```xml
<activity
    android:name="org.kde.kdeconnect.Plugins.ClibpoardPlugin.ClipboardFloatingActivity"
    android:theme="@style/Theme.Transparent"
    android:excludeFromRecents="true"/>
```

[Here is the full merge request](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/131).

Then I added support to send clipboard data from phone to all connected devices rather than sending to just one. 
[Here is the full merge request](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/142).

#### I introduced a major bug :(

Before releasing the app on Play Store, some people were testing my implemented features and reported that _Send Clipboard_
button in the notification when clicked in a OnePlus device, crashed the app. And not only it crashed but their devices froze
and they had to restart the phone. I was dumbstruck by this news. The code changes were reverted because of this. Something was
special with OnePlus devices. Also I had to rely on logs sent by testers as I didn't had a OnePlus device to test myself and
figure out what was wrong. 

Luckily, a tester agreed to work with me on this. After sometime, we figured that _invisible activity_ didn't had a layout
file. Well I had done my research on this. Android documentation says an activity without a layout file is legal. But 
OnePlus ROMs turned out to be different on this. So I rolled out a patch adding a layout file for it and some days later
it was released on Play Store.
[Here is the full merge request](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/145).

This experience made me realise the power of open-source. We talk on internet, we almost never meet and we still make
good software possible. This is __beautifull__.

#### My GSoC aspirations

I was really going and going. I put all my time into this, looking for bugs, reading other's code. 
I had applied for [Google Summer of Code](https://summerofcode.withgoogle.com/) in KDEConnect. Though my project
was not selected, I kept working on the project. Seeing this, devs offered me a developer account in KDE. So, YAY!!!

After GSoC results, I am still a part of the group. I worked on improved support of dark theme on Android app.
[Here is the full merge request](https://invent.kde.org/network/kdeconnect-android/-/merge_requests/154).

After doing this work for 6 months, I feel very proud of myself. You know every step was a hurdle and I figured a
way to jump over it. Open-source work is great. You are giving away your work for free, but you're loosing nothing. You
collaborate with project leaders who are true professionals in the industry. You learn from their style. 
If you're a student and want some work experience, a large open-source community will give you a hell of an experience.

Cheers!!!
