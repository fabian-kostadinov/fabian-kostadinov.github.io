---
layout: post
title: How to test Firestore security rules web v9 with Jest for your Angular app
comments: true
tags: [firebase, firestore, angular, jest, karma, jasmine]
---
Previously, I wrote why [using Karma as a test runner to unit test your Firestore security rules for your Angular app is a bad idea]({% post_url 2022-12-08-do-not-use-karma-and-jasmine-to-write-unit-tests-for-firestore-security-rules-in-angular-14 %}). Let's see then how you can use Jest in combination with [Firebase Firestore emulators](https://firebase.google.com/docs/rules/emulator-setup). I'll be using [Angular v14](https://angular.io/), [Firebase web v9](https://firebase.google.com/), the node package [@firebase/rules-unit-testing v2.0.5](https://www.npmjs.com/package/@firebase/rules-unit-testing) and [Jest v29.3](https://jestjs.io/).<span class="more"></span>

# The setup
Jest will be used as our unit testing framework and as a test runner. In our unit tests I will use the node package @firebase/rules-unit-testing to connect to a Firestore emulator process running on localhost:8080. The Firebase emulator suite contains various useful tools to develop and test your Angular app locally, i.e. without the need to integrate with a production database running in Google cloud. Not only do you not have to read from and write to your production database instance thus messing around with your productive data, but you can also save money from not having to run requests against a paid Google cloud service. Simply develop your security rules locally and then deploy them to your production environment running in the Google cloud.   
For this tutorial, I assume that you have already:
1. created an Angular v14 app,
2. and installed the Firebase CLI tools including the Firebase emulators.
The [command to initialize your Firestore](https://firebase.google.com/docs/rules/manage-deploy#use_the) to your Angular app is <code>firebase init firestore</code>, by the way.

# Step 1: Setting up the environment
Let's first install Jest inside our Angular app folder. The good news are that you don't even have to remove Karma/Jasmine, both can be installed in parallel with each other without interfering.
```
cd my-angular-app
npm install --save-dev jest
```
Optionally, you might also want to install the @types/jest node package if you intend to use Jest types globally in your unit tests. (I don't use them myself, so I don't install them.)
```
npm install --save-dev @types/jest
```
Second, we'll need the @firebase/rules-unit-test Node package. We won't need this for production, hence I install it as a development dependency only.
```
npm install --save-dev @firebase/rules-unit-testing
```
That was easy.

I'm not a big fan of making Jest available globally. Rather, I want to start it with a corresponding script specified in my package.json file. So, in package.json I add a new script:
```json
{
    ...
    "scripts": {
        ...,
        "test:firestore": "jest --detectOpenHandles src/firebase-test-cases/fiestore.rules.test.js",
        ...,
    },
    ...
}
```
This allows me to switch to my Angular app folder and run the unit tests contained in <code>src/firebase-test-cases/firestore.rules.test.js</code> with the command:
```
npm run test:firestore
```
Neat, ay! Note the parameter --detectOpenHandles. This parameter makes sure that any unresolved promises are properly closed when the unit test shuts down. This is to make sure we don't unintendedly leave open some database or network connections after our test cases have completed.

If you're using Visual Studio Code and would like to run Jest in debug mode, then you can also create a new runtime environment configuration. Open the file <code>.vscode/launch.json</code> and add another configuration. This is how it looks in my setup using VS Code v1.74.0:
```json
{
    "version": "0.2.0",
    "configuration": [
        ...,
        {
            "type": "node",
            "request": "launch",
            "name": "Jest debug current file",
            "program": "${workspaceFolder}/node_modules/.bin/jest",
            "args": ["${fileBasenameNoExtttension}", "--detectOpenHandles"],
            "console": "integratedTerminal",
            "disableOptimisticBPs": true,
            "windows": {
                "program": "${workspaceFolder}/node_modules/jest/bin/jest"
            }
        },
        ...
    ]
}
```
This configuration allows me to run a Jest unit test from within a file I have opened in VS Code and then clicking the Run and Debug configuration called _Jest debug current file_.

# Step 2: Firing up the Firestore local emulator
The Firebase Firestore local emulator includes a very powerful Firestore database running locally on your dev machine. Its APIs are accessible by default through localhost:8080. It does not only allow creation of new collections as well as reading and writing of documents, but it is also able to emulate the authentication mechanics plus corresponding authorisation and validation rules via Firestore security rules. Hence, you can unit test all of those elements locally. Isn't that cool? Additionally, it also has a web UI accessible by default at http://localhost:4000 that allows you to have a look at the content of your emulated Firestore database, which is very convenient for development purposes. Note that the emulator requires an installation of Java, which should be available on most computers these days.   
The initialization of Firebase tools for your Angular app has created several files and folders in the root folder of our Angular app:
* firebase.json: Contains general Firebase hosting information
* firestore.indexes.json: Information regarding Firestore DB indexes
* firestore.rules: This is the default file containing Firestore security rules. This is what we are going to unit test.
* (.firebase/): A hidden folder created by Firebase tools. Not meant to be manipulated by us.
* (firestore-debug.log): Not surprisingly a log file.

If you want to make use of the web UI and inspect your data, make sure that you specify a <code>--project=&lt;project_id&gt;</code> parameter when starting up the emulators like so. You will later on need to specify the same projectId inside your test case:
```
firebase emulators:start --project=test-my-angular-app
```
If you open the file firebase.json you can see a section towards the end specifying dependencies of firestore.indexes.json and firestore.rules. If you haven't changed the defaults then the entry looks like so:
```json
...
    "firestore": {
        "rules": "firestore.rules",
        "indexes": "firestore.indexes.json"
    }
...
```
The emulator will thus be reading out the firebase.json file first, and then read out the firestore.rules files. It will then start a Java process which contains the actual emulated services. To shut down the emulators, simply go to your console and kill the process you started, e.g. by pressing Ctrl+C [or Cmd+C buttons on a Mac] and also closing the command window running the Java process.

# Step 3: Writing the skeleton of our unit test
As mentioned before I am writing my unit test in plain Javascript in a file called <code>src/firebase-test-cases/firestore.rules.test.js</code>. First, I import several packages that I'll need in the unit test:
```javascript
const {
    assertFails,
    assertSucceeds,
    initializeTestEnvironment,
} = require('@firebase/rules-unit-testing');
const {
    getDoc,
    setDoc,
    addDoc,
    deleteDoc,
    collection
} = require('@firebase/firestore');
const fs = require('fs');

// Our instance of RulesTestEnvironment
describe("Test Firestore security rules", () => {
    // TODO
});
```
Have a look at the <code>initializeTestEnvironment</code> function. This function bootstraps our test environment object. It is asynchronous and returns a <code>Promise&lt;RulesTestEnvironment&gt;</code> object. As an argument it takes a [TestEnvironmentConfig object](https://firebase.google.com/docs/reference/emulator-suite/rules-unit-testing/rules-unit-testing.md#initializetestenvironment) that specifies the various parameters of the Firestore local emulator environment. Note the _projectId_ parameter. This takes the same value that we specified when we started the firebase emulator with the command <code>firebase emulators:start --project=test-my-angular-app</code>. If you set a distinct value, then you will end up wondering why your local Firestore emulator web UI never seems to show any data, yet your test case successfully writes to and reads from a Firestore database that you cannot seem to see. Let's initialize the test environment object and add some general cleanup code to be executed in between tests and at the end of all unit tests.
```javascript
let testEnv; // Our instance of RulesTestEnvironment
beforeAll(async () => {
    testEnv = await initializeTestEnvironment({
        projctId: "test-my-angular-app",
        rules: fs.readFileSync("firestore.rules", {"encoding": "utf-8"}),
        host: "localhost",
        port: "8080"
    }).then(resultTestEnv => {
        // The resultTestEnv variable now contains a RulesTestEnvironment object. We must
        // return it from the then clause to resolve the promise and assign it to the
        // testEnv variable.
        return resultTestEnv;
    }).catch(error => {
        console.error(error);
    });

    // TODO: Initialize Firestore with some test data
});

beforeEach(async () => {
    if (testEnv != undefined) {
        // To clean up emulator state and data in between tests call:
        // await testEnv.cleanup();
    }
});

afterAll(async () => {
    if (testEnv != undefined) {
        // Remove all data from our running emulated firestore instance
        // await testEnv.clearFirestore();
    }
});
```
There are several points worth noting. See how we are using async/await in <code>beforeAll</code>, <code>beforeEach</code> and <code>afterAll</code> functions? Each of those functions takes a callback as an argument. You must make sure to wait until those asynchronous callbacks are resolved, otherwise these functions will simply continue immediately rather than wait until the cleanup actions have completed. According to the [docs](https://firebase.google.com/docs/reference/emulator-suite/rules-unit-testing/rules-unit-testing.rulestestenvironment) <code>RulesTestEnvironment.cleanup()</code> and <code>RulesTestEnvironment.clearFirestore()</code> return a <code>Promise&lt;void&gt;</code> object, so we must <code>await</code> their completion too.   
Another important point is how we read out the firestore.rules file using <code>fs.readFileSync</code>. Instead of reading out the file we could also simply provide the rules in a string variable. This is fine for simple security rules, but your security rules can become more complicated easily when writing validation rules for your data, so keeping those rules in a separate file is definitely a better approach.

# Step 4: Initializing some data to our emulated Firestore database
Great, we can now create a test environment with which to work. But how do we add some initial data to our database, so that our unit tests have some initial data to work with? Problem is that we are still in the setup phase, so we don't want to perform some potentially complicated authentication first. How to solve that? The solution is yet another function: <code>RulesTestEnvironment.withSecurityRulesDisabled(callback)</code>. This function executes a callback function that returns a <code>Promise&lt;void&gt;</code> and to which it provides a _context_ argument. The context argument can be used to obtain a reference to the local Firestore object by calling <code>context.firestore()</code> - but with security rules entirely disabled. We effectively circumvent the Firestore authentication here in our emulated Firestore environment! Sounds complicated, so let's look at an example. A very frequent need is to have a _users_ collection with some documents containing information on our users such as full name, address, maybe a URL to a profile picture and other user info.
```javascript
// Within the beforeAll function after having initalized testEnv variable:
await testEnv.withSecurityRulesDisabled(async (ctx) => {
    // Create a new object to persist
    const newDoc = { userId: "1234567", username: "Bob Firefighter", isAdmin: false };

    // Obtain the reference to the Firestore collection
    let dbRef = collection(ctx.firestore(), "users");

    // Write the new document to the database
    let docRef = await addDoc(dbRef, newDoc)
        .then(resultDocRef => {
            console.log(resultDocRef.id);
            return resultDocRef;
        })
        .catch(error => {
            console.error(error);
        });
})
.then(() => {
    console.log("Initialization of test data successfully completed");
})
.catch(error => {
    console.error("Initialization of test data failed with error: ", error);
});
```

# Step 5: Observe your data in the Firestore emulator web UI
Once you've started the Firebase emulators, you will get access to a web UI running at http://localhost:4000. Just enter the URL into your browser and navigate to the Firestore tab. There you can inspect which data are currently loaded into your database. Of course, if you wipe out your data immediately in a _beforeEach_ or _afterAll_ function in your test case then you will never see any data appearing as it's immediately cleaned out after you finished your test case.

# Step 6: Write some test cases
We are ready to create some actual test cases. We have two options here. We can either rely on [Jest's own asynchronous expect functions](https://jestjs.io/docs/asynchronous), or we rely on [rules-unit-test's assertSucceeds or assertFails functions to test permission granted or denied situations](https://firebase.google.com/docs/reference/emulator-suite/rules-unit-testing/rules-unit-testing). We did not really go into how to write the security rules and validation rules so far, but let's imagine there is one [validation rule deployed on the server applying a regular expression that checks whether an email provided is valid](https://firebase.google.com/docs/reference/rules/rules.String#matches). Typically, immediately after a user is registered in Firestore, a new request is sent to the server to create a new user object with some additional user information.
```javascript
test("username must be of type string", async () => {
    // Here we attempt to send a new user doc to the server that has an invalid
    // username type of number rather than string
    const newDoc = { userId: "7777777", email: "invalid@email", isAdmin: false };
    const ctx = testEnv.authenticatedContext({userId: "1234567"});
    const dbRef = collection(ctx.firestore(), "users");

    await assertFails(addDoc(dbRef, newDoc));
});
```

# Step 7: Deploy your locally tested Firestore security rules to the production server
Finally, you want to [deploy those security rules back to the production server](https://firebase.google.com/docs/rules/manage-deploy) hosted on Google Firebase by calling:
```
firebase deploy --only firestore:rules
```

----

References
* Angular: [https://angular.io/]()
* Jest: [https://jestjs.io/]()
* API reference of @firebase/rules-unit-test node package: [https://firebase.google.com/docs/reference/emulator-suite/rules-unit-testing/rules-unit-testing]() 
* Firebase CLI docs: [https://firebase.google.com/docs/cli]()
* Sample projects howo to test Firestore security rules: [https://github.com/firebase/quickstart-testing/tree/master/unit-test-security-rules]()
* Javascript API reference of @firebase/firestore node package: [https://firebase.google.com/docs/reference/js/firestore_]()
* Using Firebase Firestore local emulator suite to test Firestore security rules: [https://firebase.google.com/docs/rules/emulator-setup]()
* RulesTestEnvironment API reference: [https://firebase.google.com/docs/reference/emulator-suite/rules-unit-testing/rules-unit-testing.rulestestenvironment]()
* Cloud Firestore Security Rules API reference: [https://firebase.google.com/docs/reference/rules/index-all]()
* Another useful tutorial that is still using Firebase web v8 rather than v9: [https://medium.com/firebase-developers/develop-your-firestore-with-tdd-unit-testing-security-rules-afefb0d772c4]()