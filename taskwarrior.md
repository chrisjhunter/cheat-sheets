# Taskwarrior & Timewarrior Cheat Sheet

Taskwarrior (`task`) for to-dos, Timewarrior (`timew`) for time tracking —
commonly wired together via a Taskwarrior `on-modify` hook so starting a
task starts the clock.

## Taskwarrior basics
```bash
task add "print team sheet"                   # add a task
task add project:home priority:H "fix fence"  # with attributes
task list                                     # all pending tasks
task all                                      # everything, incl. completed/deleted
task next                                     # prioritized shortlist (Taskwarrior's "next" report)
task 2 start                                  # start working on task 2
task 2 stop
task 35 done                                  # mark complete
task 2 modify priority:H
task 2 anno "waiting on parts"                # add an annotation/note
task 2 delete
task undo                                     # undo last change
```

## Filtering & searching
```bash
task +home list                # tasks with tag "home"
task list +home                # same, tag filter
task project:home list         # filter by project
task list project:home
task list priority:H           # filter by priority
task +home |grep -i kub        # combine with grep for text search
task list |grep -i mens
task status:completed list     # completed tasks
task due:today list            # due today
task due.before:tomorrow list  # overdue + due today
```

## Reports & views
```bash
task                  # default report (usually "next")
task list project     # grouped/columned by project
task summary          # project progress summary
task burndown.daily   # ASCII burndown chart
task calendar         # calendar view with due dates
task history.monthly  # completed/added history
```

## Contexts & config
```bash
task context define work project:Work  # define a saved filter context
task context work                      # activate it (subsequent commands scoped to it)
task context none                      # clear active context
task config                            # show all config
task show report.next.columns          # inspect one config value
```

## Timewarrior basics
```bash
timew start                          # start tracking, no tag
timew start "client work"            # start with a tag
timew stop
timew track 9am - 5pm "client work"  # log a completed block after the fact
timew continue                       # resume last tracked interval
timew cancel                         # discard the active (unsaved) interval
```

## Reviewing time
```bash
timew show           # current status
timew day            # today's intervals
timew yesterday
timew week
timew lastweek
timew month
timew lastmonth
timew summary :week  # aggregated summary for the week
timew summary :ids   # include interval IDs (for modify/move)
```

## Editing entries
```bash
timew modify start @1 2023-11-28T16:00:00    # fix the start time of interval @1
timew modify end @1 5pm
timew move @1 2023-11-28T4pm 2023-11-28T5pm  # shift an interval
timew move @1 :yesterday                     # move to a relative day
timew delete @1
timew join @1 @2                             # merge two intervals
timew split @1                               # split an interval in two
```

## Tags & annotations
```bash
timew tag @1 "billable"
timew untag @1 "billable"
timew annotate @1 "fixed the deploy script"
timew tags  # list all tags used
```

## Linking task + time (common on-modify hook pattern)
```bash
# ~/.task/hooks/on-modify.timewarrior (from the official Taskwarrior extension repo)
# starts/stops timew automatically when a task's status changes to/from "started"
```
```bash
timew report :project  # if tags mirror task project names, this doubles as project time
```

## Useful one-liners
```bash
task rc.verbose=nothing export | jq '.[] | select(.status=="pending") | .description'  # scriptable export
timew export :week | jq '[.[] | .duration] | length'                                   # count of tracked intervals this week
task +home export | jq length                                                          # count of tasks matching a filter
timew summary :month :ids                                                              # month summary with interval IDs for editing
```
