---
layout: post
title: Do not use Karma & Jasmine for unit testing Firestore Security Rules in your Angular app!
comments: true
tags: [firebase, firestore, angular, jest, karma, jasmine]
---
I'm writing an Angular 14 app with Google Firebase Firestore as a database. I assumed that developing the Firestore Security Rules with Firebase web v9 by writing Karma/Jasmine unit tests should be not too hard. After all, Karma is Angular 14's default test runner, no? Turns out that using Karma to develop or test Firestore Security Rules is a bad idea. You should use Jest (or maybe Mocha) instead.<span class="more"></span>

# Why using Karma is a bad idea when writing unit tests for Firestore
Angular 14 comes with [Karma](https://karma-runner.github.io/) & [Jasmine](https://jasmine.github.io/) as its built-in testing framework. To be more precise: Karma is the test runner framework, and Jasmine the actual behaviour-driven unit testing framework. Of course you can switch to a different setup like Jest or Mocha, but if Karma is already the default framework it's just natural to take this as a starting point for writing unit tests.   
The [Karma documentation](https://karma-runner.github.io/6.4/intro/how-it-works.html) tells us a crucial detail:

> Karma is essentially a tool which spans a web server [...]
> After starting up, Karma loads plugins and the configuration file, then starts its local web server which listens for connections. Any browser already waiting on websockets from the server will reconnect immediately. As part of loading the plugins, test reporters register for 'browser' events so they are ready for test results.
>Then karma launches zero, one, or more browsers [...]

If your goal is to write and unit test Firestore Security Rules locally using Firebase emulators than these are bad news. Why? In short: Because your unit test will end up running in a browser (!) process trying to connect to a server process, and as such is not allowed to access the local file system. And that's most likely what you want to do when trying to develop and unit test Firestore Security Rules. To understand this we need to highlight a few things about Firestore Security Rules and the Firebase emulators.

Some time ago Google introduced [Firebase emulators](https://firebase.google.com/docs/rules/emulator-setup) to develop both your frontend SPA application against the Firestore backend locally. There are different reasons why this is convenient for every developer, among them being no need to make changes to an existing production database, no costs involved for running queries on Google's servers, and speedier development without the need for an internet connection.   
Using Firebase tools we can put all our Firestore security rules in a local <code>firestore.rules</code> file, test the security rules locally, and only deploy that file to the server once it's ready.    
According to the documentation how to unit test Firestore Security Rules you can initialise a test environment like so:

```javascript
Let testEnv = await initializeTestEnvironment({
    projectId: "demo-project-1234",
    firestore: {
        rules: fs.readFileSync("firestore.rules", "utf8")
    },
});
```

The problem is: This does not work in a browser process. The browser's security disallows you from accessing the local file system, hence you cannot use <code>fs</code> to read out a file.

You could of course put all the security rules in a local string variable and assign that in the parameter object given to <code>initializeTestEnvironment</code> instead of reading it from the file system. But if your security rules grow in complexity, this is a very poor workaround - not least because you want do actually deploy the entire <code>firestore.rules</code> file to the server using the corresponding Firebase CLI commands:
```
firebase deploy --only firestore:rules
```
Thus, you really do not want to keep your security rules in a simple string variable within your unit test.   
Equally dissatisfactory is to write and test your security rules in the Firestore web UI, as it's slow and cumbersome if your rules are anything else than trivial.

And that's why I recommend switching to Jest instead. Jest works differently than Karma behind the scenes. I'll describe the development process in more details in another article.