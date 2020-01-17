---
layout: post
title: Two Tips for Debugging Apps in IIS
date: '2015-11-14 23:33:47'
tags:
- iis
---

If you develop web sites or services hosted in IIS, here are two tips to make debugging on your development machine easier.

#### Disable Health Monitoring

IIS application pools have health monitoring built in to ensure processes are responding to requests in a timely manner. If a process stops responding after a period of time, IIS will kill it. This is really inconvenient when you're debugging web apps in your dev environment, and IIS ends your session. You can disable this feature to allow a debugger to be attached indefinitely. Of course, you should never do this in non-development environments.

Just open up the app pool's advanced settings and find _Ping Enabled_. Set it to false.

<img style="width: 90%" src="http://media.joebuschmann.com/ping_enabled.png" alt="IIS App Pool Settings - Ping Enabled">

#### Set Identity to ApplicationPoolIdentity

If you set the identity of an application pool to _ApplicationPoolIdentity_, the IIS worker processes in Task Manager will appear with the application pool's name in the User Name column instead of a Windows user. In addition, if you also create an app pool for each web application, you can quickly identify which w3wp.exe process points to which web application.

<img style="width: 90%" src="http://media.joebuschmann.com/application_pool_identity.png" alt="IIS App Pool Settings - Application Pool Identity">

For instance, if I'm debugging excessive memory use, I can see exactly which application is causing the issue. In this case, _Berkley Concierge Se_ is the culprit.

<img style="width: 90%" src="http://media.joebuschmann.com/task_manager.png" alt="Task Manager">
