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
In this post we will go over how to setup a support viewer in a web application using Om Next and [Untangled](http://untangled-web.github.io/untangled/). This feature allows developers to step through the history of user actions, run these actions against development builds, and even run tests against these actions.

<!-- TODO Show video of stepping through app state here -->

### Om Next History

One of the nice features about Om Next is the use of a single global app state, which contains the information necessary to render the UI. A [reconciler](https://medium.com/@kovasb/om-next-the-reconciler-af26f02a6fb4#.o2k30oj4t) exists to handle updates in both directions.

Updates to the app state are applied via transactions, and each successful transaction returns a new app state with a UUID. Om Next automatically records the last 100 states of the application, which makes it trivial to observe the history of our actions. Once we setup a way to save this collection of app states, we can observe user interactions.

### Saving User Actions 

To persist changes to the server in an Om Next application, we must implement a client side and server side mutation. The reconciler will optimistically update the client and will send a POST to the server to run the server mutation.

The client mutation is straightforward. We deref the global app state and call `untangled.client.core/history` to get the last 100 app states. Then we send the history to the server along with an id and a comment describing the session.

<script src="https://gist.github.com/kenbier/56d514a4c71021677c6ecaa3913757b9.js"></script>

The server mutation is also fairly straightforward, despite looking more involved. 
We store the history of app states in a key-value store, using the `support-id` as the key. The support-id is also added to the user entity in datomic. The `:user/sessions` attribute acts as a foreign key to our key value store.

<!-- TODO Seperate defusecase post. Just link to it.-->
<!-- `defusecase` is a wrapper around Om's server dispatches, which adds some middleware and runs authorization policies before executing the body. Untangled also makes available [components](https://github.com/stuartsierra/component) in the `env` of our server mutation, such as the database or blob-store.  -->

<script src="https://gist.github.com/kenbier/f2667be66a7a79bc0a5ab845d81002a2.js"></script>

<!-- TODO section on setting up support viewer for code reloading using CORS -->
<!-- TODO mention how to read support request -->

<!-- TODO mention how we trigger support request in airbrake? -->

<!-- TODO reference to another post on predictive testing -->
