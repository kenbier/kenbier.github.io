---
layout: post
title: "Record and Replay User Actions"
description: ""
date: 2016-11-05
tags: []
comments: true
share: true
---

<!-- * Show mutations to setup support request and where we call mutations -->
<!-- * Show support viewer code + CORS dev support -->
<!-- * Reference to predictive testing, cover in another post. -->
In this post we will go over how to setup a support viewer in a web application using Om Next and [Untangled](http://untangled-web.github.io/untangled/). This feature allows developers to step through the history of user actions, replay these actions in development builds, and even run tests against these actions.

<!-- TODO Show video of stepping through app state here -->

## Om Next History

One of the nice features about Om Next is the use of a single global app state, which contains the information necessary to render the UI. A [reconciler](https://medium.com/@kovasb/om-next-the-reconciler-af26f02a6fb4#.o2k30oj4t) exists to handle updates in both directions.

Updates to the app state are applied via transactions, and each successful transaction returns a new app state with a UUID. Om Next automatically records the last 100 states of the application, which makes it trivial to observe the history of our actions. Once we setup a way to save this collection of app states, we can observe user interactions.

## Saving User Actions 

To persist changes to the server in an Om Next application, we must implement a client side and a server side mutation. The reconciler will optimistically update the client and will send a POST to the server to run the server mutation.

The client mutation is straightforward. The global app state is derrefed and passed to `untangled.client.core/history` to get the last 100 app states. Then the history is sent to the server with an id and a comment describing the session.

<script src="https://gist.github.com/kenbier/56d514a4c71021677c6ecaa3913757b9.js"></script>

The server mutation is also fairly straightforward. We store the history of app states in a key-value store, using the `support-id` as the key. The support-id is also added to the user entity in datomic. The `:user/sessions` attribute acts as a foreign key to our key value store.

<!-- TODO Seperate defusecase post. Just link to it.-->
<!-- `defusecase` is a wrapper around Om's server dispatches, which adds some middleware and runs authorization policies before executing the body. Untangled also makes available [components](https://github.com/stuartsierra/component) in the `env` of our server mutation, such as the database or blob-store.  -->

<script src="https://gist.github.com/kenbier/f2667be66a7a79bc0a5ab845d81002a2.js"></script>

## Setting up the Support Viewer

Now that we have our user sessions persisted, we can setup the support viewer that untangled provides. First we implement a read function on the server to return a support request, given an id. If the user has admin rights, the user session is fetched from the key-value store.

<script src="https://gist.github.com/kenbier/85075d90da411084a2c8d1d53cc8013e.js"></script>

<!-- Picture of table of user errors -->
<!-- Explanation of URL scheme -->
<!-- Each URL looks something like `https://report.adstage.io/support?id=<some_UUID>&token=<some_token>`. The hostname passed to `net/make-untangled-network` can be changed to localhost for demos or troubleshooting the viewer build. -->

Next we setup the support viewer [component](https://github.com/untangled-web/untangled-client/blob/master/src/untangled/support_viewer.cljs). We decided to implement the `start-untangled-support-viewer` method ourselves to add some features.

The support viewer is going to mount two applications, the Om Next application and a support application which controls the app state your UI renders with. To mount the Om Next app, we provide the initial state from our main build, plus some additional information such as `:ui/support-viewer?` and `:ui/no-router`. These extra values turn off things like airbrake error reporting, the router, and web tracking. 

The viewer app is pointed to our production endpoint, allowing the viewer to load user sessions that are stored in the production database via the `:started-callback`. This callbacks runs the `:support-request` mutation we implemented above, passing the `:id` URL parameter to the mutation.

One last thing to note is the `request-transform`, which wraps all calls from the two mounted apps with authorization tokens grabbed from the `token` URL param. Without it the viewer requests to our production server would be denied.


<!-- TODO splitup URL piece into own section, running user session in dev built. Mention how 
grabs token and id from URL params. 
Have way to list sessions in prod.
So go to production, click on session, app loads. Change URL to localhost, same params,
--!>

<script src="https://gist.github.com/kenbier/a4b045f0ef62235b9239ed279e8ee10e.js"></script>

<!-- TODO mention how we trigger support request in airbrake? -->
<!-- TODO reference to another post on predictive -->
