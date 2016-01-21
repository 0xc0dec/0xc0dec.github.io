---
layout: page
title:  TeamCity agent startup&#58; no protocol tc/RPC2 error
date:   2011-12-15 00:00:00
---

If you happen to install TeamCity agent and see this error in `output.log` file,
make sure that `serverUrl` property in `buildAgent.properties` file is correct and has `http:// prefix`!
Today I spent a lot of time striving to fix the problem, until I decided to change `serverUrl=tc` to `serverUrl=http://tc` (`tc` is our TeamCity server address).