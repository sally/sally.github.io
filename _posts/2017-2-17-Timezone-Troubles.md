---
layout: post
title: Timezone Troubles
date: 2017-02-07
---

Publications are a core part of our app. They are intended to be administrators' mediums to reach their colleagues about announcements, events, emergencies, etc., so you can imagine that getting the dates and times correct on these publications is key.

More specifically, we want to display the correct timestamps relative to the reader's timezone (rather than the publisher's timezone). It's easy to say, "oh, we'll just put HH:MM PST/MST/CST/EST depending on the publisher's time," but then we're leaving the employees to do the conversions themselves - which isn't very conducive to a positive user experience.

-> ![](http://i.imgur.com/byDfdbJ.gif) <-

-> *So if I live in Phoenix, and it's currently daylight saving's time, which Arizona doesn't observe... Then I subtract 2 hours my time to get to... Wait, that's not right.* <-
