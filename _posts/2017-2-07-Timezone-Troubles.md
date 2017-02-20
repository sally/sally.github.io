---
layout: post
title: Timezone Troubles and UTC Offsets
date: 2017-02-07
---

Publications are a core part of our app. They are intended to be administrators' mediums to reach their colleagues about announcements, events, emergencies, etc., so you can imagine that getting the dates and times correct on these publications is key.

That might be easy to think about if a company is very small and has all its operations in one timezone, but what about those that have functions all across the US, or even international sites? Specifically, we want to display the correct timestamps relative to the reader's timezone (rather than the publisher's timezone).

It's easy to say, "oh, we'll just put HH:MM PST/MST/CST/EST depending on the publisher's time," but then we're leaving the employees to do the conversions themselves - which isn't very conducive to a positive user experience.

![](http://i.imgur.com/byDfdbJ.gif)

*"So if I live in Phoenix, and it's currently daylight saving's time, which Arizona doesn't observe... Then I subtract 2 hours my time to get to... Wait, that's not right."*

Our team went to tackle the problem at its root - what is the behavior of date and time when a Publication object is created in the database?

```ruby
> p = Publication.new
=> #<Publication id: nil, created_at: nil, updated_at: nil>
> p.save
   (0.2ms)  BEGIN
  SQL (0.6ms)  INSERT INTO "publication" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", 2017-02-19 17:04:46 UTC], ["updated_at", 2017-02-19 17:04:46 UTC]]
   (8.5ms)  COMMIT
=> true
> p.created_at
=> Tue, 07 Feb 2017 17:04:46 UTC +00:00
```

It looks like they are stored as ActiveSupport::TimeWithZone objects, with UTC as the timezone. Perfect - now all we needed was a way to get the user's UTC offset, so that we could use it to compute the correct time of publication relative to the user.

We decided that the best way to obtain this UTC offset is from the front-end. JavaScript has a neat way of obtaining the UTC offset from the client:

```javascript
> var d = new Date()
> var n = d.getTimezoneOffset();
> n
480
```

Now, this 480 might seem like a mysterious number, but it is the UTC offset in minutes for my timezone, PST. Multiplying this by 60 gives us the UTC offset in seconds, which is 28,800.

This number might be slightly misleading, because PST is actually 28,800 seconds *behind* UTC, not ahead, so the human mind might expect -28,800 instead (since this is the number you subtract from UTC to get PST).

This is fine, we can just multiply by -60 instead in the previous step.

Great. At this point, we have asked our front-end engineers to assign a new entry variable called `utc_offset` which is passed into the system upon login of a user.

Now, we can manipulate this `utc_offset` variable in order to serve the correct dates/times. In a helper module that helps us deal with date/time formatting and converting, a key line of code in a function called `convert_utc_based_on_offset` is:

```ruby
offset_date = date_object + offset.to_i.seconds
```

where the `offset` input variable is by default set to 0, so that in case we fail to obtain `utc_offset` from the current account, we simply just show UTC times for all dates/times.

We arrived at this line of code by doing a test run in the console of the first publication:

```ruby
> created_at = Post.first.created_at
Tue, 07 Feb 2017 17:04:46 UTC +00:00
> with_offset = created_at + (-28800.seconds)
Tue, 07 Feb 2017 09:04:46 UTC +00:00
```

While the timezone on the object still isn't correct, we only care about extracting the correct date and time:

```ruby
> with_offset.strftime('%D %r')
"02/07/17 09:04:46 AM"
```

Now the time that is displayed to a user regarding publications is independent of the publisher's timezone.

Another cool thing is that if a user needs to travel to another timezone, the app will respond accordingly (since `utc_offset` is gotten from the client) and show the correct times relative to the correct timezone. Tada!
