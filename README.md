---
title: How to host a Sapper.js SSR app on Firebase.
published: true
description: Sapper is an up and coming JavaScript framework for creating Server-Side applications. This Guide takes you through the steps on how to tweak your project to run it on Firebase with Cloud functions and Firebase hosting!
tags: Sapper SSR, Sapper Firebase, Sapper, Firebase
cover_image: https://thepracticaldev.s3.amazonaws.com/i/2au4fu10q5c51dqenp1e.jpg
---

I've been spending about two days scouring the net on trying to find the optimal way to integrate Sapper with Firebase. It's not as easy as it sounds.

[Link to example site](https://sapper-firebase.web.app)

## What is Sapper?

Sapper is a framework for building extremely high-performance web apps. It is a compiler actually. All your code gets built and optimized ready for production.

It's built on Svelte.js. Sapper has many cool features and I recommend you check the docs out [Sapper](https://sapper.svelte.dev/). The Framework allows for SSR capabilities, but can also generate a static site with `npm run export`.

## What is Firebase

Firebase is Google's platform for "Building apps fast, without managing infrastructure". Basically, in a few commands you can have your static site hosted, hooked up to a database and have authentication readily available.

Other than static sites, Firebase also explores the realm of "serverless" by way of "Functions". These are basically small pieces of logic that you can call either from your app, or when some sort of update in terms of authentication or database occurs.

Functions are useful for:

- Getting sensitive logic out of the client
- Performing some action on a piece of data (eg. sanitizing) before it gets inserted into the database.
- **In our case, helping our code serve up a Server Side Rendered application**

## Benefits of server-side

- SEO: Crawlers can better index your pre-generated markup.
- PERFORMANCE: You don't need to serve up bloated assets and you can do some server side caching.

## Why the combination of Firebase x Sapper?

Firebase development is really fast and easy, so is Sapper's. I thought: "Why not best of both worlds?". I can have all the niceties of Firebase authentication, database and CDN hosting - along with the speed of development, size of app and general awesomeness of Sapper.

# Let's get started

First, we'll need to install the necessary dependencies. (I'm assuming that you already have Node.js version >= 10.15.3 and the accommodating NPM version on your system).

## Installing Sapper

Sapper uses a tool called degit, which we can use via `npx`:

```bash
$ npx degit sveltejs/sapper-template#rollup sapper_firebase

$ cd sapper_firebase && npm install # install dependencies
```

## Adding firebase to the project

- Note:
  You will need the Firebase CLI tool for this:
  `npm i -g firebase-tools@latest`

**Before the next step:**

You need to go to [Firebase](https://console.firebase.google.com) and log in to a console. Here you have to create a new Firebase project. When the project is created, on the home page click the **</>** button to add a new web-app. Name it whatever you like.

Now we can proceed.

```bash
# in /sapper_firebase
$ firebase login # make sure to login to your console account

$ firebase init functions hosting
```

This will return a screen where you can select new project:

```bash
Select a default Firebase project for this directory:
❯ sapper-firebase (sapper-firebase) # select your project
```

Follow the steps:

```bash

? What do you want to use as your public directory? static
? Configure as a single-page app (rewrite all urls to /index.html)? N
? Choose your language: Javascript.
? Do you want to use ESLint to catch probable bugs and enforce style? N
? Do you want to install dependencies with npm now? Y
```

Then let your functions dependencies install.

## Changing our firebase.json

The goal we want to achieve by changing the firebase.json is to:

1. Serve our project's static files through Firebase hosting.
2. Redirect each request to the Function that we're going to create (I'll explain this in detail).

Here's the updated firebase.json:

```json
{
  "hosting": {
    "public": "static",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "function": "ssr"
      }
    ]
  }
}
```

> The rewrite is checking for requests under any route, and basically letting the function named 'ssr' handle it.

So let's create the 'ssr' function:

`/sapper_firebase/functions/index.js`

```javascript
const functions = require("firebase-functions");

exports.ssr = functions.https.onRequest((_req, res) =>
  res.send("hello from function")
);
```

This is only temporary so we can test our function. We can do so by running:

```bash
$ firebase serve
```

if you get this message:

`⚠ Your requested "node" version "8" doesn't match your global version "10"`

You can go to `functions/package.json` and set:

```json
"engines": {
    "node": "10"
}
```

Other than that, after running `firebase serve` and going to http://localhost:5000, you should see this:

![](https://thepracticaldev.s3.amazonaws.com/i/mack7ekvjl8y4vl7zakf.png)

## Testing the function

We're seeing that because Firebase created an `index.html` file in our `static` floder. We can safely delete it and we can go to http://localhost:5000/ and should now see:

![](https://thepracticaldev.s3.amazonaws.com/i/blvrq5jrqsggbarimnph.png)

## Hoorah!

Now that we've proven our function to work and redirect requests, we need to mess with Sapper a bit, to get our function to run our server. Let's start off by editing the `src/server.js` file, our goals are:

1. We want to export the sapper middleware that is generated on build.
2. We want to only run the local server on `npm run dev`.

`src/server.js`

```javascript
import sirv from "sirv";
import polka from "polka";
import compression from "compression";
import * as sapper from "@sapper/server";

const { PORT, NODE_ENV } = process.env;
const dev = NODE_ENV === "development";

if (dev) {
  polka() // You can also use Express
    .use(
      compression({ threshold: 0 }),
      sirv("static", { dev }),
      sapper.middleware()
    )
    .listen(PORT, err => {
      if (err) console.log("error", err);
    });
}

export { sapper };
```

We can test if `npm run dev` works by letting it build and visit http://localhost:3000. It should show a picture of Borat (Standard Sapper humor).

But this is still only our Sapper server, we want to run it from our Firebase function. First we need to install express:

```bash
$ cd functions
$ npm install express sirv compression polka
```

> Note: 'sirv' 'compression' and 'polka' are required, because of the built directory that depends on it. We'll only use express in our function though. But if we exclude the others, our function will fail on deploy.

After installing express and the other dependencies, we first want to tweak our workflow to build a copy of the project into functions. We can do this by editing the npm scripts:

```json
...
"scripts": {
    "dev": "sapper dev",
    "build": "sapper build --legacy && cp -Rp ./__sapper__/build ./functions/__sapper__",
    "prebuild": "rm -rf functions/__sapper__/build",
    "export": "sapper export --legacy",
    "start": "npm run build && firebase serve",
    "predeploy": "npm run build",
    "deploy": "firebase deploy",
    "cy:run": "cypress run",
    "cy:open": "cypress open",
    "test": "run-p --race dev cy:run"
},
...
```

This will copy all the necesarry files to the functions folder before hosting or serving them locally.

Try it with:

```bash
$ npm start # now using firebase
```

You should see the message from earlier!

## Serving our Sapper app via Functions and express

We can plug an express app into our function, use our imported Sapper middleware on express and serve our SSR app seemlessly. The static folder is also being served via the very fast Firebase CDN.

`functions/index.js`

```javascript
const functions = require("firebase-functions");
const express = require("express");

// We have to import the built version of the server middleware.
const { sapper } = require("./__sapper__/build/server/server");

const app = express().use(sapper.middleware());

exports.ssr = functions.https.onRequest(app);
```

## Hosting your Sapper Firebase SSR app locally

All you have to do now is:

```bash
$ npm start
```

**AAANND...**

# HOORAAAH

![](https://thepracticaldev.s3.amazonaws.com/i/rfz8m42n5xiqad5k1bga.png)

If you see this image on http://localhost:5000 your app is being served by a local firebase functions emulator!

To confirm that it is SSR, just reload a page and check the page source, all the markup should be prerenderd on initial load. Also check out your terminal, you should see all kinds of requests as you navigate your app!

## Hosting your app

Because of our neat NPM scripts hosting/deploying is as easy as:

```bash
$ npm run deploy
```

This will take a while, to get your files and functions up. [But here's my version online SSRing like a boss](https://sapper-firebase.web.app).

## Thank you

This is one of my first posts that I actually feel could help someone out, since it took me hours to figure out (Maybe I'm just slow). But I typed it out since I can't make videos on my [Youtube Channel](https://www.youtube.com/channel/UCd4UmlBFIhj-yJrzn6foxgw), because of time issues.

But if you did enjoy this, or have questions, chat to me on twitter [@eckhardtdreyer](https://twitter.com/eckhardtdreyer). Until next time.
