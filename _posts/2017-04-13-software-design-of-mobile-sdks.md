---
layout: post
title:  "Software Design of Mobile SDKs"
date:   2017-04-13
categories: blog
tags: sdk
---

I’m Pat, and I write crap code. 

    Note: I am no longer 'Pat', but I still write crap code.

I know this, because I’ve been writing code for almost 25 years and every time I look at past-Pat’s software I think, “Damn it, past-Pat, why are you so bad at things?”

I’d like to share with you some of the things I’ve learned writing the Teak mobile SDKs as well as work from years back including Qualcomm, GarageGames, and other contract work.

# Design Philosophy

I like to pretend that, for every line of code a developer has to copy/paste to integrate my SDK, the developer punches me in the face. Therefore, my goal is to minimize the face-punching.

Chances are, their producer also wants to punch me in the face for each line of code as well. So there’s a strong incentive to keep things simple.

Some things can’t be made simple, and that’s fine. Complicated things can be complicated, but simple things need to be simple.

You need to identify the things that will work for 90% of your users, and make them as simple as possible. You are not just writing code for today; you are writing code for hundreds of people’s tomorrows. That’s a lot of support overhead, so do the work to make it simple. Once it’s in the wild, change is hard.

Keep that face-punch count low.

# I’m bad, you’re bad, we’re all bad

Developers are jerks. I know this because I’m a developer.

If we can pass `NULL`, we’ll pass `NULL`. If we can pass empty string, we’ll pass empty string. If we can pass an uninitialized struct, we’ll pass an uninitialized struct.

You need to validate and sanitize the input your SDK takes in. Seems pretty basic, yeah? Well we’re bad at things, aren’t we. Make no assumptions about what gets passed in to your SDK. Check for `NULL`, check for out of range enums, do the work.

Now remember that the developers of the OS are also bad. Don’t assume that the calls you make to get OS information are going to behave as described. Validate what you get back from OS calls.

If input is bad, print out a useful error to the console, and then put your SDK into an invalid state (more on this later), do not cause a crash if you can avoid it.

# Lifecycle Driven Development

While writing the Teak SDKs I stumbled across a strategy that I wish I would have known about many years ago. I decided to call it “lifecycle driven development.”

The mobile application lifecycle is going to likely impact the stability of your SDK more than anything else.

Don’t write any of the functionality code for your SDK until you have completed writing all of the code you need to interact with the application lifecycle.

I’m serious, don’t write any of the functionality code in your SDK until you have implemented *and tested and debugged* the interactions with the application lifecycle. It will save you a lot of headaches.

What does this really mean? First some important implementation notes, then I’ll tell you.

# Some Implementation Notes

While I want to keep this article focused on philosophy over code, I also want to make sure you get punched in the face a minimum number of times. So with that in mind, I strongly suggest that you do not mimic the lifecycle and force the integrator of your SDK to call MySdk.onResume, MySdk.onPause, and what not.

Instead, you should be using [method swizzling on iOS](http://nshipster.com/method-swizzling/) to hook in to the [UIApplication Lifecycle](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/). On Android you should be using [ActivityLifecycleCallbacks](https://developer.android.com/reference/android/app/Application.ActivityLifecycleCallbacks.html), and have the integrator call a method on your SDK during `Activity.attachBaseContext`.

# Finite State Machines are Great

So you know what application lifecycle events your SDK cares about, what now? Finite state machine.

No, it’s not overcomplicating things, you bad developer. If that question didn’t pop into your head, maybe you’re a better developer than I am. The first time I thought, “This is a good use case for a FSM,” my next thought was, “Isn’t that overcomplicating it?”

Your SDK does not have all of the information it needs right from the application start/resume, to be fully functional. It just doesn’t.

> Even if the only thing your SDK needs is the IDFA, you can’t assume you have it, or can get it right away.

Even on iOS, when getting the IDFA is a synchronous call, even when it is “always available”...check those docs, sometimes it will return nil, and when it does, Apple says to just try again later.

Your SDK should use a finite state machine. It should output to the debug log when it makes state transitions. It should validate the data it needs to progress states, and reject transitions when it doesn’t have the data it needs.

Do the work. I promise you it will make bugs so much easier to find.

# Invalid State

I said previously that you should not cause crashes, and your SDK should instead be put into an invalid state. This should be a state in your FSM that any state can transition into, and can never get transitioned out of.

Outside operations should fail, and print out debug notes, but you should not crash.

# Some More Design Philosophy

As an SDK author, you job is not only to write a good, solid, easy-to-work-with SDK; it’s also to (in as much as is possible) prove that your SDK isn’t breaking the customer’s application.

Your customers are not just people writing code, they are people with spreadsheets and timelines. They have features that need to ship, and schedules that fall behind.

If you assume that QA is going to test new features in isolation, or that good development practices are being employed, you are making a mistake. Remember, we’re all bad at things.

Your SDK will get blamed for bugs it can’t possibly break.

You will get crash reports that never came up in testing.

You will get frustrated, but if you put in the work to design your SDK well, and validate inputs, you will be able to find the bugs in your code and be responsive to issues when they arise.

Take a deep breath, and remember that I’m bad at coding, you’re bad at coding, and we’re all just trying to ship.

# You Aren’t Writing an SDK Yet
So you’ve got your lifecycle interactions wired up to a finite state machine, and it is all working. Are you ready for functionality? Restrain your enthusiasm.

> You aren’t writing an SDK yet; you are now writing a domain specific language that you’re going to use to write your SDK.

You want to now implement things like “When the SDK next enters this state, run the following block.” This is going to help you, not only avoid a bunch of boiler-plate code, but it’s going to help you organize your thoughts and intentions around the functionality your SDK provides.

Take the time to think about this. As an SDK author, one of the most important things you can do is think about how the guts of your system will work. What can break it, and how to prevent those things.

Take a walk, have a think. It’s important.

# How About Now?

Nope, still not ready.

The last thing you need to do is think about the data contexts that will exist in your SDK.

In Teak, I split these up into Application Context, Device Context, Debug Context, and Session.

The Application Context contains application-specific configuration data, once each value is assigned it won’t change during the lifecycle of the application.

The Device Context contains all of the information about the device that we’re concerned with. None of it is specific to the application, and once assigned the values won’t change during the lifecycle of the application.

The Debug Context contains few values, and they are persisted to disk. These values may change during the lifecycle of the application, and are used for things like the ability to enable debug logging in a production build via the result of a remote call to our servers. This allows our customers to turn on verbose logging in a production build for their specific device so that they can diagnose issues if needed.

Finally the Session contains all of the information about a play session. It tracks things like player information, and can be expired based on things like time past in the background, or signing in with a new user.

I keep these things in separate contexts based on how the data is used by our SDK, and how the data will change over the lifecycle of the application.

Again, these are things that may seem like overkill but put in the hours to think about how data is going to flow in your SDK, and do the work.

# Begin

Now, you may begin the functionality.

I hope that you find, as I did, that the functionality of your SDK just...falls out of your fingers while you code.

I’ll append some short tips and suggestions to this.

Keep your face-punch count low, and good luck!

# Tips and Suggestions
* The first thing your SDK does should be to print out a version number to the debug log. You should be able to resolve that version to a specific version control commit. I use ‘git describe tags always’.
* Detect debug/production and adjust debug output accordingly. Provide a manual way to override this.
* You can do cool things with Android Studio, as it will detect URLs and make them clickable in the debug log. I use this so that developers can click on a link, and it will open a browser tab that creates a new issue in GitHub and I auto-insert a JSON blob of data that will help me diagnose the issue.
* After initialization, in debug mode, our SDK writes out a JSON blob containing information to help me debug issues to the debug log. I ask that integrators copy/paste that blob when reporting issues.
* Try and write things that delight your customers. Since our SDK includes push notifications, when everything is configured properly we output a URL in debug mode that the developer can click or copy/paste into a browser and the server will send a push notification to that device right away. It’s a little thing, but it helps me test and debug, and it’s a delightful thing for our customers.

*Thanks to Robert D. Blanchet Jr.*
