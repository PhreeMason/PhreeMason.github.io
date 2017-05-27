---
layout: post
title:  My Calendar
date:   2017-05-27 13:16:25 +0000
---


This app was supposed to take only 3, maybe 4 days to make. However, I learned pretty early that rails is much harder when you aren’t within safe cozy blanket of a learn lab/code along. Like always, it took me quite some time to figure what I wanted to make. The idea for a calendar app came to me while I was browsing programmer forums and came across a layout template for a beautiful calendar design which can be seen [here](https://responsivedesign.is/patterns/calendar/). 

The template only showed an outline for the month August 2014, with the days hard coded. It was up to me to build out the months and days for the rest of the calendar. I also need a reason for the calendar app, so I decided to make it a site to keep track of tasks and dates, similar to the calendar app on most phones. 

At first I had trouble getting my models to behave and interact the way I wanted them to. I wanted to make a calendar that was very accurate, one that keeps track of leap years and accurately displays each day under the correct weekday. The problem was I couldn’t think of a way to do that without reinventing the time model, which is not what I wanted to do. After failing to create my own time model I realized I only needed to display time, not create it.

This first thing I did was creat methods that properly display month days.


```
  def first_week_days
    week_days = []
    day_of_week  = DateTime.new(year, order).wday
    n = 7 - day_of_week
    (1..n).each { |e| week_days << e }
    week_days
  end

  def middle_week_days
    week_days = []
    start = first_week_days.last + 1
    finish = last_week_days.first - 1
    all_days = (start..finish).map { |e| e }
    all_days.each_slice(7) {|e| week_days << e }
    week_days
  end

  def last_week_days
    week_days = []
    day_of_week = DateTime.new(year, order).end_of_month.wday
    n = days - day_of_week
    (n..days).each { |e| week_days << e }
    week_days
  end

```
The beginning days and ending days for any month varies. Months can begin on any day of the week and end on any day of the week. However the days in the middle of the month are the same for all months. I used ruby’s datetime model to help identify the days at the end and beginning of a month. 
The first_week_days method uses the datetime model to find out which day of the week the first day of a month falls on and stored the value in day_of_week. This value is then subtracted from 7 so I can identify which days to allocate to the first month and which days to give to the previous month. The same process is repeated in the last_week_days method however this time it tracks the last day of the month instead of the first and counts backwards from the last day. This middle_week_days method simply gathers all the days between the first two methods and breaks them up into groups of seven in order to fill up all the middle weeks.

After displaying the days of a month, I need to show the tasks for that day within the calendar. I also needed to show users their upcoming tasks. To accomplish this I wrote a sql query to retrieve all tasks that did not already expire. 

```
  def self.upcoming_for_user(user)
    tasks = joins(:calendar).where(["calendar_id = ?", user.calendar.id]).where('start_time >= ?', Time.now).order("start_time")
  end
```

This method gets all tasks from the database for a given user with a start time that did not already pass. Although it's just one line, it took me about 20 minutes to write!

My favorite part of this project *also where I spent most of my time* was styling the elements using css. This was my first time styling a site solo and I enjoyed it and am proud of the final results. 

