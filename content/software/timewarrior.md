---
title: timewarrior
date: 2025-06-18
categories: handbook
tags:
  - tools
---

# timewarrior

## help

A time tracking tool used to measure the duration of activities. [More information](https://timewarrior.net/docs).

```

  Start tracking an activity:

      timew start

  Tag the current activity:

      timew tag activity_tag

  Start tracking and tag a new activity:

      timew start activity_tag

  Stop the current activity:

      timew stop

  Track an activity in the past:

      timew track start_time - end_time activity_tag

  View tracked items of the day:

      timew summary

  View report for the last day, week, current month, etc.:

      timew summary :today|yesterday|week|lastweek|month|lastmonth|year|lastyear

```

```
Usage: timew [--version]
       timew annotate @<id> [@<id> ...] <annotation>
       timew cancel
       timew config [<name> [<value> | '']]
       timew continue [@<id>] [<date>|<interval>]
       timew day [<interval>] [<tag> ...]
       timew delete @<id> [@<id> ...]
       timew diagnostics
       timew export [<interval>] [<tag> ...]
       timew extensions
       timew gaps [<interval>] [<tag> ...]
       timew get <DOM> [<DOM> ...]
       timew help [<command> | dates | dom | durations | hints | ranges]
       timew join @<id> @<id>
       timew lengthen @<id> [@<id> ...] <duration>
       timew modify (start|end) @<id> <date>
       timew modify range @<id> <interval>
       timew month [<interval>] [<tag> ...]
       timew move @<id> <date>
       timew [report] <report> [<interval>] [<tag> ...]
       timew retag @<id> [@<id> ...] <tag> [<tag> ...]
       timew shorten @<id> [@<id> ...] <duration>
       timew show
       timew split @<id> [@<id> ...]
       timew start [<date>] [<tag> ...]
       timew stop [<tag> ...]
       timew summary [<interval>] [<tag> ...]
       timew tag @<id> [@<id> ...] <tag> [<tag> ...]
       timew tags [<interval>] [<tag> ...]
       timew track <interval> [<tag> ...]
       timew undo
       timew untag @<id> [@<id> ...] <tag> [<tag> ...]
       timew week [<interval>] [<tag> ...]

Additional help:
       timew help <command>
       timew help dates
       timew help dom
       timew help durations
       timew help hints
       timew help ranges

Interval:
       [from] <date>
       [from] <date> to/- <date>
       [from] <date> for <duration>
       <duration> before/after <date>
       <duration> ago
       [for] <duration>

Tag:
       Word
       'Single Quoted Words'
       "Double Quoted Words"
       Escaped\ Spaces

Configuration overrides:
       rc.<name>=<value>

```

## timew-dom - Timewarrior DOM

Supported DOM references are:

```
dom.tag.count             Count of all tags
dom.tag.1                 Nth tag used
```

```
dom.active                '1' if there is active tracking, otherwise '0'
dom.active.tag.count      Count of active tags
dom.active.tag.1          Active Nth tag
dom.active.start          Active start timestamp (ISO Extended local date)
dom.active.duration       Active elapsed (ISO Period)
dom.active.json           Active interval as JSON
```

```
dom.tracked.count         Count of tracked intervals
dom.tracked.1.tag.count   Count of active tags
dom.tracked.1.tag.1       Tracked Nth, Nth tag
dom.tracked.1.start       Tracked Nth, start time
dom.tracked.1.end         Tracked Nth, end time, blank if closed
dom.tracked.1.duration    Tracked Nth, elapsed
dom.tracked.1.json        Tracked Nth, interval as JSON
```

```
dom.rc.<name>             Configuration setting
```
