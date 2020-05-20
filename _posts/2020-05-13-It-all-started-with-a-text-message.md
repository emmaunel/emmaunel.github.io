---
title: It all started with a text message
date: 2020-05-13 14:10:00 +0800 # Come back for time 7:47:26
categories: [Reversing]
tags: [android]
seo:
  date_modified: 2020-03-07 20:43:16 -0500
---

## Update

The real company `itsme` contacted me on twitter and they verifed that they didn't created an android app for their product. So they reported the app to google and it has been taking down successfully.

## Summary

People are receiving text messages saying that their friends invited them to chat. The messages makes them install an application. Opening the app, it shows a quick walkthrough of the app and asks for a verification code. With the right code, you are instructed to install another application is constantly spamming the user with ads.   

## Quick vocabulary

- APK: Android Package is the package file format used by the Android operating system for distribution and installation of mobile apps.
- Activities: An activity is a single, focused thing that the user can do.
- Emulator: simulates Android devices on your computer
- jadx-gui: Dex to Java decompiler
- Android manifest: The manifest file describes essential information about your app.

I got a text message from a random number saying that my friends wanted to chat to on another app(`itsme`). I initially though this was weird since my friends would tell me if they wanted to switch to another app. Since the link was going to apple store. I ignored it.

<img src="/assets/img/itsme/Screenshot_20200513-121303_Messages.jpg" alt="drawing" width="350"/>

Waking up the next day, some of my friends got the same messages. Now, I had to pay attention to it. It wasn't just a coincidence. Switching to my VM, I downloaded the app and opened it with a decompiler, `jadx-gui`.

I took a look at the Manifest.xml to find the entry point.

![itsme/Screen_Shot_2020-05-13_at_12.34.52_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_12.34.52_PM.png)

There were a total of 7 activities. 6 of them were just onboarding info about the app.

<img src="/assets/img/itsme/Screen_Shot_2020-05-13_at_12.52.02_PM.png" alt="drawing" width="350"/>

Then there's the main activity which only ask for a verification code.

<img src="/assets/img/itsme/Screen_Shot_2020-05-13_at_12.54.53_PM.png" alt="drawing" width="350"/>

Right away, I saw the person can't even spell `install`. Bad start. 

I found two ways to get the verification code. First, if you clicked on the left button, you get redirect to a website, `movsup.org`. The site asks for a username and your phone platform. Then it tells you to rate the app on the play store to get the code. I couldn't get it that way because I was in an emulator and I was not logged in to the play store.

Site                       |  Site verifcation
:-------------------------:|:-------------------------:
<img src="/assets/img/itsme/Screen_Shot_2020-05-13_at_1.02.27_PM.png" alt="drawing" width="350"/>  |  <img src="/assets/img/itsme/Screen_Shot_2020-05-13_at_1.02.51_PM.png" alt="drawing" width="350"/>

The other solution was to  get the code with `jadx-gui`. Entering the main activity, there's a validate function. 

![itsme/Screen_Shot_2020-05-13_at_1.19.06_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_1.19.06_PM.png)

We can see the the access code and two urls.

- hxxp://bit.ly/APKdownd
- hxxps://t.co/PXu5l8lpzv?amp=1

Entering the code in the app would work, but because I wasn't logged in, it failed on me. Time to investigate the links.

The `[t.co](http://t.co)` link is a redirect to `movsup.org`. The `[bit.ly](http://bit.ly)` is shorten link for a public google drive with a download button for another apk. 

Installing the apk on my emulator, I didn't see any indication of it being installed. There was no app icon or anything else to prove to me it was there. Looking back to jadx-gui, I looked at the manifest again and the package name made everything clear.

The whole application might have just been a joke.

![itsme/Screen_Shot_2020-05-13_at_1.37.48_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_1.37.48_PM.png)

There is much more activity in this application. In the main activity, nothing much is going on except that it is loading an ad

![itsme/Screen_Shot_2020-05-13_at_1.49.52_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_1.49.52_PM.png)

The rest of the app are doing the same.

![itsme/Screen_Shot_2020-05-13_at_1.53.40_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_1.53.40_PM.png)

![itsme/Screen_Shot_2020-05-13_at_1.54.10_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_1.54.10_PM.png)

People who downloaded the app from the play store noticed the constant ad being presented.

<img src="/assets/img/itsme/Screenshot_20200513-124930_Google_Play_Store.jpg" alt="drawing" width="350"/>

How was the application able to spread? Well, I am not sure how the app was able to send the invite message. People on twitter were saying that it read your contacts and send the message.

![itsme/Screenshot_20200513-140720_Twitter.jpg](/assets/img/itsme/Screenshot_20200513-140720_Twitter.jpg)

I didn't see any indication of that. The permission declared in both apps had nothing to do with contacts. The first app(`itsme`) had just needed internet connection and wake lock

![itsme/Screen_Shot_2020-05-13_at_2.15.58_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_2.15.58_PM.png)

The second app(`tapeviral`) had more and those permissions were to read the application's badges

![itsme/Screen_Shot_2020-05-13_at_2.17.00_PM.png](/assets/img/itsme/Screen_Shot_2020-05-13_at_2.17.00_PM.png)

The whole application is just a troll from what I could find. Nothing but trying to make money by spamming users with useless ads. Best thing to do is just delete the app and not click links from phone numbers you don't know.

## File hash

| App Name      | MD5                              |
|---------------|----------------------------------|
| itsme.apk     | e62513f35edd11e1aae3dfc54ddc133c |
| tapeviral.apk | 144b09aec1d7909c80520178bc0e37df |