---
title: taskwarrior
date: 2025-06-17
tags:
  - tools
categories: handbook
---

# taskwarrior

Command-line to-do list manager. [More information](https://taskwarrior.org/docs/).

## overview

Taskwarrior has a flexible command line syntax, but it may not be clear at first what the underlying structure means. Here is the general form of the syntax:

![](https://taskwarrior.org/images/syntax.png)

There are four parts to the syntax (`filter`, `command`, `modifications`, and `miscellaneous`), and each part is optional.

```tldr
  Add a new task which is due tomorrow:
      task add description due:tomorrow

  Update a task's priority:

      task task_id modify priority:H|M|L

  Complete a task:

      task task_id done

  Delete a task:

      task task_id delete

  List all open tasks:

      task list

  List open tasks due before the end of the week:

      task list due.before:eow

  Show a graphical burndown chart, by day:

      task burndown.daily

  List all reports:

      task reports
```

man:

```man
Documentation for Taskwarrior can be found using 'man task', 'man taskrc', 'man task-color', 'man task-sync' or at https://taskwarrior.org

The general form of commands is:
  task [<filter>] <command> [<mods>]


The <mods> consist of zero or more changes to apply to the selected tasks, such as:
  task <filter> <command> project:Home
  task <filter> <command> +weekend +garden due:tomorrow
  task <filter> <command> Description/annotation text
  task <filter> <command> /from/to/     <- replace first match
  task <filter> <command> /from/to/g    <- replace all matches

Tags are arbitrary words, any quantity:
  +tag       The + means add the tag
  -tag       The - means remove the tag

Built-in attributes are:
  description:    Task description text
  status:         Status of task - pending, completed, deleted, waiting
  project:        Project name
  priority:       Priority
  due:            Due date
  recur:          Recurrence frequency
  until:          Expiration date of a task
  limit:          Desired number of rows in report, or 'page'
  wait:           Date until task becomes pending
  entry:          Date task was created
  end:            Date task was completed/deleted
  start:          Date task was started
  scheduled:      Date task is scheduled to start
  modified:       Date task was last modified
  depends:        Other tasks that this task depends upon
Alternately algebraic expressions support:
  and  or  xor            Logical operators
  <  <=  =  !=  >=  >     Relational operators
  (  )                    Precedence

  task due.before:eom priority.not:L   list
  task '(due < eom and priority != L)'  list
```

### Filter

A filter is a means of addressing a subset of tasks. Because filters are optional, the simplest case is no filter. A command with no filter addresses all tasks.

Generally filter arguments appear before the command, so any arguments to the left of the command are considered filter arguments.

There is a special case, in which a command that does not support modifications or miscellaneous arguments, expects only filter arguments, and so they can appear before or after the command, without confusing Taskwarrior:

![](https://taskwarrior.org/images/filter.png)

### Command

Each time you run Taskwarrior, you are issuing a `command` either explicitly, or implicitly with the default command (the `default.command` configuration setting). The command you specify determines how the command line is understood by Taskwarrior. Here are some examples of that:

![](https://taskwarrior.org/images/syntaxes.png)

The first example, `task list` is a report with no filter, and the second, `task +home list` is with a filter. The third, `task 12 modify project:Garden` has both a filter and modifications. The last example, `task show editor` has a miscellaneous argument.

Taskwarrior looks for the first argument on the command line that looks like an exact command name, and failing that, looks for an abbreviated command name. It is better to use the full name of a command to avoid ambiguity.

It is the position of the `command` argument, and the type of command that determines how the arguments are understood.

task的列信息

```task columns
Command          Category  R/W ID GC Recur Context Filter Mods Misc Description
columns          config     RO                                 Misc All supported columns and formatting styles
config           config     RO                                 Misc Change settings in the task configuration
reports          config     RO                                      Lists all supported reports
show             config     RO                                 Misc Shows all configuration variables or subset
udas             config     RO                                      Shows all the defined UDA details
commands         metadata   RO                                      Generates a list of all commands, with behavior details
stats            metadata   RO    GC          Ctxt   Filt           Shows task database statistics
ids              metadata   RO ID GC Recur           Filt           Shows the IDs of matching tasks, as a range
count            metadata   RO    GC Recur    Ctxt   Filt           Counts matching tasks
projects         metadata   RO    GC Recur    Ctxt   Filt           Shows all project names used
tags             metadata   RO    GC          Ctxt   Filt           Shows a list of all tags used
uuids            metadata   RO    GC Recur           Filt           Shows the UUIDs of matching tasks, as a space-separated list
context          context    RO                                 Misc Set and define contexts (default filters / modifications)
```

获取信息

```internals
_aliases         internal   RO                                      Generates a list of all aliases, for autocompletion purposes
_columns         internal   RO                                      Displays only a list of supported columns
_commands        internal   RO                                      Generates a list of all commands, for autocompletion purposes
_config          internal   RO                                      Lists all supported configuration variables, for completion purposes
_context         internal   RO                                      Lists all supported contexts, for completion purposes
_get             internal   RO                                 Misc DOM Accessor
_ids             internal   RO ID GC Recur           Filt           Shows the IDs of matching tasks, in the form of a list
_projects        internal   RO    GC Recur           Filt           Shows only a list of all project names used
_show            internal   RO    GC                                Shows all configuration settings in a machine-readable format
_tags            internal   RO    GC Recur           Filt           Shows only a list of all tags used, for autocompletion purposes
_udas            internal   RO                                      Shows the defined UDAs for completion purposes
_unique          internal   RO ID GC                 Filt      Misc Generates lists of unique attribute values
_urgency         internal   RO    GC                 Filt           Displays the urgency measure of a task
_uuids           internal   RO    GC Recur           Filt           Shows the UUIDs of matching tasks, as a list
_version         internal   RO                                      Shows only the Taskwarrior version number
_zshattributes   internal   RO                                      Generates a list of all attributes, for zsh autocompletion purposes
_zshcommands     internal   RO                                      Generates a list of all commands, for zsh autocompletion purposes
_zshids          internal   RO ID GC Recur           Filt           Shows the IDs and descriptions of matching tasks
_zshuuids        internal   RO    GC Recur           Filt           Shows the UUIDs and descriptions of matching tasks
```

操作

```operation
add              operation  RW                Ctxt        Mods      Adds a new task
annotate         operation  RW                       Filt Mods      Adds an
append           operation  RW                       Filt Mods      Appends text to
delete           operation  RW                Ctxt   Filt Mods      Deletes the
denotate         operation  RW                Ctxt   Filt      Misc Deletes an
done             operation  RW                Ctxt   Filt Mods      Marks the
duplicate        operation  RW                Ctxt   Filt Mods      Duplicates the
edit             operation  RW       Recur    Ctxt   Filt           Launches an
log              operation  RW                Ctxt        Mods      Adds a new task
modify           operation  RW                       Filt Mods      Modifies the
prepend          operation  RW                Ctxt   Filt Mods      Prepends text to
purge            operation  RW    GC          Ctxt   Filt           Removes the
start            operation  RW                Ctxt   Filt Mods      Marks specified
stop             operation  RW                Ctxt   Filt Mods      Removes the
undo             operation  RW                                      Reverts the most
```

统计

```graphs
burndown.daily   graphs     RO    GC Recur    Ctxt   Filt           Shows a
burndown.monthly graphs     RO    GC Recur    Ctxt   Filt           Shows a
burndown.weekly  graphs     RO    GC Recur    Ctxt   Filt           Shows a
calendar         graphs     RO ID GC                           Misc Shows a
ghistory.annual  graphs     RO       Recur    Ctxt   Filt           Shows a
ghistory.daily   graphs     RO       Recur    Ctxt   Filt           Shows a
ghistory.monthly graphs     RO       Recur    Ctxt   Filt           Shows a
ghistory.weekly  graphs     RO       Recur    Ctxt   Filt           Shows a
history.annual   graphs     RO       Recur    Ctxt   Filt           Shows a report
history.daily    graphs     RO       Recur    Ctxt   Filt           Shows a report
history.monthly  graphs     RO       Recur    Ctxt   Filt           Shows a report
history.weekly   graphs     RO       Recur    Ctxt   Filt           Shows a report
summary          graphs     RO    GC          Ctxt   Filt           Shows a report
```

### Modifications

If a command accepts modifications, they generally appear after the command. Most commands that accept modifications also accept filters, and so the filter arguments appear before the command, while the modifications appear after. Here is an example:

![](https://taskwarrior.org/images/modification.png)

This command specifies a compound filter, consisting of more than one term. These terms are logically combined with an `and` operator by default, unless otherwise specified. In this case, tasks that have both the `home` tag, and a `status` value of `pending` are to be modified.

The modifications, appearing after the command, set the `priority` to `H` igh, and the `due` date to the end of the month (`eom`).

Because the filter is evaluated at runtime, we don’t know how many tasks will be modified. It could be none, one, many or all of the tasks. It could be determined with:

```
task +home status:pending count
```

The user writing this command would have an idea of how many tasks this will affect, but this is just an example, with no contextual data shown.

### Miscellaneous

Some commands accept neither a filter, nor modifications, but do accept miscellaneous arguments. An example is the `show` command, that queries configuration settings, and does not accept a filter:

![](https://taskwarrior.org/images/miscellaneous.png)

This is another special case, in which the command only accepts miscellaneous arguments, and so they can appear before or after the command.

### Overrides

Overrides are temporary values for configuration settings, and can be specified anywhere on the command line, because they are not considered to be either filter, modification or miscellaneous. In fact, the command itself doesn’t see the overrides, instead they are handled before the command runs.

![](https://taskwarrior.org/images/override.png)

There can be any number of overrides on the command line, and they have no effect on the syntax.

## reports

Taskwarrior has three kinds of reports. There are built-in reports that cannot be modified, such as `info` and `summary`. There are built-in reports which can be redefined completely or eliminated, such as `list`, `next`. And finally there are your own custom reports. To generate a list of _all_ the reports, use the `reports` command:

```reposts
Report           Description
---------------- --------------------------------------------------
active           Active tasks
all              All tasks
blocked          Blocked tasks
blocking         Blocking tasks
burndown.daily   Shows a graphical burndown chart, by day
burndown.monthly Shows a graphical burndown chart, by month
burndown.weekly  Shows a graphical burndown chart, by week
completed        Completed tasks
ghistory.annual  Shows a graphical report of task history, by year
ghistory.monthly Shows a graphical report of task history, by month
history.annual   Shows a report of task history, by year
history.monthly  Shows a report of task history, by month
information      Shows all data and metadata
list             Most details of tasks
long             All details of tasks
ls               Few details of tasks
minimal          Minimal details of tasks
newest           Newest tasks
next             Most urgent tasks
oldest           Oldest tasks
overdue          Overdue tasks
projects         Shows all project names used
ready            Most urgent actionable tasks
recurring        Recurring Tasks
summary          Shows a report of task status by project
tags             Shows a list of all tags used
unblocked        Unblocked tasks
waiting          Waiting (hidden) tasks

28 reports
```

### Built-In Static Reports

Typically, a report consists of a table of data, with one row of data corresponding to a single task, with the task attributes represented as columns. But there are other reports which do not conform to this structure. Those are the built-in static reports, and they are not modifiable, because they are quirky and require custom code. These reports are:

```
burndown.daily
burndown.monthly
burndown.weekly
calendar
colors
export
ghistory.annual
ghistory.monthly
history.annual
history.monthly
information
summary
timesheet
```

Each of these reports takes non-standard arguments, may or may not support filters, and generates distinctive output.

### Built-In Modifiable Reports

These reports are standard format, using standard arguments, and are also modifiable.

```
active
all
blocked
blocking
completed
list
long
ls
minimal
newest
next
oldest
overdue
ready
recurring
unblocked
waiting
```

Suppose you wanted to remove a column from one of these reports, for example removing `tags` from the `minimal` report. First look at the existing columns and labels of the `minimal` report:

`task show report.minimal.labels` shows

```
Config Variable       Value
--------------------- ---------------------------
report.minimal.labels ID,Project,Tags,Description
```

and `task show report.minimal.columns` shows

```
Config Variable        Value
---------------------- ---------------------------------------
report.minimal.columns id,project,tags.count,description.count
```

Having determined what the current values are, simply override with the new values:

```shell
$ task config report.minimal.labels  'ID,Project,Description'
...
$ task config report.minimal.columns 'id,project,description.count'
```

You can think of the built-in modifiable reports as a set of default custom reports.

### Custom Reports

Defining a custom report is straightforward. You have control over the data shown, the sort order and the column labels. A custom report is simply a set of configuration values, and those values include:

- description
- columns
- labels
- sort
- filter

Let us quickly create a custom report, which will be named `simple`. This report will display the task ID, project name and description. We will need to gather the five settings listed above.

The description is the easiest, and in this case the report will be described:

```coffeescript
Simple list of open tasks by project
```

This is just a descriptive label that will be used when the report is listed. Next we need to specify the columns in the report, and the order in which those are shown. Here the `_columns` helper command will show the columns available:

`task _columns` shows

```
depends
description
due
end
entry
foo
id
imask
mask
modified
parent
priority
project
recur
scheduled
start
status
tags
until
urgency
uuid
wait
```

That represents all the data that Taskwarrior stores, and therefore all the data that may be shown in a report. Our `simple` report will show id, project and description, which are all in the list. This means our column list is simply:

```
id,project,description
```

But there are also formats for each column, and the `columns` command shows them, with examples. Here are the formats for our three columns: `task columns id`

```
Columns Supported Formats Example
------- ----------------- ------------------------------------
id      number*           123
uuid    long*             f30cb9c3-3fc0-483f-bfb2-3bf134f00694
        short             f30cb9c3
```

This is easy, because there is only one `id` format. `task columns project`

```
Columns Supported Formats Example
------- ----------------- -------------
project full*             home.garden
        parent            home
        indented            home.garden
```

There are three formats for the `project` column, and the default, `full` is the one we want. `task columns description`

```
Columns     Supported Formats Example
----------- ----------------- -----------------------------------------------------------------------------
description combined*         Move your clothes down on to the lower peg
                                2014-02-08 Immediately before your lunch
                                2014-02-08 If you are playing in the match this afternoon
                                2014-02-08 Before you write your letter home
                                2014-02-08 If you're not getting your hair cut
            desc              Move your clothes down on to the lower peg
            oneline           Move your clothes down on to the lower peg 2014-02-08 Immediately before ...
                              this afternoon 2014-02-08 Before you write your letter home 2014-02-08 If ...
            truncated         Move your clothes do...
            count             Move your clothes down on to the lower peg [4]
```

There are five formats for description. This time we prefer the `count` format, so our columns list is now:

```
id,project,description.count
```

Labels are the column heading labels in the report. There are defaults, but we wish to specify these like this:

```
ID,Proj,Desc
```

Sorting is also straightforward, and we want the tasks sorted by project, and then by entry, which is the creation date for a task. This illustrates that a task attribute that is not visible can be used in the sort. The sort order is then:

```
project+/,entry+
```

The `+` means an ascending order, but we could have used `-` for descending. The `/` solidus indicates that `project` is a break column, which means a blank line is inserted between unique values, for a visual grouping effect. 2.4.0

Finally, we need a filter, otherwise our report will just display all tasks, which is rarely wanted. Here we wish to see only pending tasks, and that means the filter is:

```
status:pending
```

Now we have our report definition, so we just create the five configuration entries like this:

```shell
task config report.simple.description 'Simple list of open tasks by project'
task config report.simple.columns     'id,project,description.count'
task config report.simple.labels      'ID,Proj,Desc'
task config report.simple.sort        'project+/,entry+'
task config report.simple.filter      'status:pending'
```

Note the equivalent report directly from the config file would look like that:

```config
report.simple.description=Simple list of open tasks by project
report.simple.columns=id,project,description.count
report.simple.labels=ID,Proj,Desc
report.simple.sort=project+\/,entry+
report.simple.filter=status:pending
```

And it is finished. Run the report like this:

```shell
task simple

ID Proj Desc
-- ---- -----------------
 1 Home Wash the windows
 3 Home Vacuum the floors

 2      Food shopping
```

Custom reports also show up in the help output.

```shell
task help | grep simple
task  simple              Simple list of open tasks by project
```

I can inspect the configuration.

```shell
task show report.simple

Config Variable           Value
------------------------- ------------------------------------
report.simple.columns     id,project,description.count
report.simple.description Simple list of open tasks by project
report.simple.filter      status:pending
report.simple.labels      ID,Proj,Desc
report.simple.sort        project+/,entry+
```

Now the report is fully configured, it joins the others and is used in the same way.

## attributes

### built-in

```
Built-in attributes are:
  description:    Task description text
  status:         Status of task - pending, completed, deleted, waiting
  project:        Project name
  priority:       Priority
  due:            Due date
  recur:          Recurrence frequency
  until:          Expiration date of a task
  limit:          Desired number of rows in report, or 'page'
  wait:           Date until task becomes pending
  entry:          Date task was created
  end:            Date task was completed/deleted
  start:          Date task was started
  scheduled:      Date task is scheduled to start
  modified:       Date task was last modified
  depends:        Other tasks that this task depends upon
```

### modifiers

Attribute modifiers make filters more precise. Supported modifiers are:

```
  Modifiers         Example            Equivalent           Meaning
  ----------------  -----------------  -------------------  -------------------------
                    due:today          due = today          Fuzzy match
  not               due.not:today      due != today         Fuzzy non-match
  before, below     due.before:today   due < today          Exact date comparison
  after, above      due.after:today    due >= tomorrow      Exact date comparison
  none              project.none:      project == ''        Empty
  any               project.any:       project !== ''       Not empty
  is, equals        project.is:x       project == x         Exact match
  isnt              project.isnt:x     project !== x        Exact non-match
  has, contains     desc.has:Hello     desc ~ Hello         Pattern match
  hasnt,            desc.hasnt:Hello   desc !~ Hello        Pattern non-match
  startswith, left  desc.left:Hel      desc ~ '^Hel'        Beginning match
  endswith, right   desc.right:llo     desc ~ 'llo$'        End match
  word              desc.word:Hello    desc ~ '\bHello\b'   Boundaried word match
  noword            desc.noword:Hello  desc !~ '\bHello\b'  Boundaried word non-match

Alternately algebraic expressions support:
  and  or  xor            Logical operators
  <  <=  =  !=  >=  >     Relational operators
  (  )                    Precedence

  task due.before:eom priority.not:L   list
  task '(due < eom and priority != L)'  list
```

note that

```
  by         due.by:today       due <= today         Exact date comparison
```

### How Recurrence Works

A recurring task is a task with a due date that keeps coming back as a reminder. Here is an example:

```
task add Pay the rent due:1st recur:monthly until:2015-03-31
Created task 123.
```

This task has a due date, a monthly recurrence, and an optional until date coinciding with the end of the lease.

A recurring task is given a status of `recurring` which hides it from view, although you can see it in the `all` report. The recurring task you create is called the template task, from which recurring tasks instances are created. So the template remains hidden, and the recurring instances that spawn from it are the tasks that you will see and complete.

Here is a look at the template task:

```shell
task 123 info

Name        Value
ID          123
Description Pay the rent
Status      Recurring
Recurrence  monthly
Entered     2014-03-01 12:13:28 (42 secs)
Due         2014-04-01 00:00:00
Until       2015-03-31 00:00:00
UUID        64bcf8fd-74d5-40d2-9e57-1d6a5922cdfc
Urgency     2.4
```

Now if you run a report, such as `task list`, you will see the first instance of that recurring task generated. We can take a look at the instance:

```shell
task 124 info

Name        Value
ID          124
Description Pay the rent
Status      Pending
Recurrence  monthly
Parent task 64bcf8fd-74d5-40d2-9e57-1d6a5922cdfc
Mask Index  0
Entered     2014-03-01 12:17:03 (15 secs)
Due         2014-04-01 00:00:00
Until       2015-03-31 00:00:00
UUID        29d2df7a-1165-4559-b974-a727519e00f1
Urgency     2.4
```

Notice how the instance has a status `pending`, and a reference back to the template task (Parent task). In addition, you can see it inherited the recurrence and description, and if there was a project, priority and tags, those would also be inherited.

The recurring instance has an attribute named ‘Mask Index’, which is zero. This indicates that it is the first of the many recurring task instances, the counting being zero-based.

Now if we look back at the template task, we see changes:

```shell
task 123 info

Name          Value
ID            123
Description   Pay the rent
Status        Recurring
Recurrence    monthly
Mask          -
Entered       2014-03-01 12:13:28 (3 mins)
Due           2014-04-01 00:00:00
Until         2015-03-31 00:00:00
Last modified 2014-03-01 12:17:03 (24 secs)
UUID          64bcf8fd-74d5-40d2-9e57-1d6a5922cdfc
Urgency       2.4

Date                Modification
2014-03-01 12:17:03 Mask set to '-'.
                    Modified set to '2014-03-01 12:17:03'.
```

> [!note] template mask
> The template task now has a `mask` attribute, and some change history. That `mask` is a string of task statuses. It has a value of `-` which indicates that one instance was created, task 124, because it is one character long. The `-` means that instance is still pending. This is how the template task keeps track of what it does and does not need to generate. When the task instance changes status that `-` becomes `+` (completed) or `X` (deleted) or `W` (waiting).

Note that you never directly interact with task 123, the template task. It is hidden for a reason. Instead, you interact with the recurring task instances, and in most cases, changes are propagated to the template task and optionally other sibling tasks.

In this example one task instance is generated for the next due period. This is because the configuration setting `recurrence.limit` is set to 1, the default. If this number is increased to 2, then you would see the next 2 instances generated. Note that this only generates two steps into the future, without regard for whether those two instances are completed or not - don’t expect to complete the first task and see a new one pop up immediately.

## filter

```
The <filter> consists of zero or more restrictions on which tasks to select, such as:
  task                                      <command> <mods>
  task 28                                   <command> <mods>
  task +weekend                             <command> <mods>
  task project:Home due.before:today        <command> <mods>
  task ebeeab00-ccf8-464b-8b58-f7f2d606edfb <command> <mods>

By default, filter elements are combined with an implicit 'and' operator, but 'or' and 'xor' may also be used, provided parentheses are included:
  task '(/[Cc]at|[Dd]og/ or /[0-9]+/)'      <command> <mods>

A filter may target specific tasks using ID or UUID numbers.  To specify multiple tasks use one of these forms:
  task 1,2,3                                    delete
  task 1-3                                      info
  task 1,2-5,19                                 modify pri:H
  task 4-7 ebeeab00-ccf8-464b-8b58-f7f2d606edfb info
```

### Tags, Virtual Tags

Tags are arbitrary words, any quantity:

+tag The + means add the tag
-tag The - means remove the tag

The basic tag syntax is very powerful and simple to use. There are two ways to use this, shown here:

```shell
task +HOME list
task -WORK list
```

These two commands illustrate the complete tag interface. The first command is a filter that lists only tasks that have the `HOME` tag. The second command is a filter that lists only tasks that _do not_ have the `WORK` tag. The + and - syntax therefore means presence and absence of a tag. This is simple to use, and can be combined like this:

```shell
task +HOME -WORK list
```

This shows tasks that have the `HOME` tag, but do not have the `WORK` tag. This is a very simple and easy to use mechanism, but it does require that your tasks are properly tagged. In other words, it is based directly on task metadata.

A tag may be any single word that does not start with a digit, punctuation, or mathematical operator.

#### Supported Virtual Tags

Since version 2.2.0, Taskwarrior has supported virtual tags, and the list will continue to grow. Here is the full list of supported virtual tags:

- `BLOCKED` - Is the task dependent on another incomplete task?
- `UNBLOCKED` - The opposite of `BLOCKED`, for convenience. Note `+BLOCKED` == `-UNBLOCKED` and vice versa.
- `BLOCKING` - Does another task depend on this incomplete task?
- `DUE` - Is this task due within 7 days? Determined by `rc.due`
- `DUETODAY` - Is this task due sometime today?
- `TODAY` - Is this task due sometime today?
- `OVERDUE` - Is this task past its due date?
- `WEEK` - Is this task due this week? 2.3.0
- `MONTH` - Is this task due this month? 2.3.0
- `QUARTER` - Is this task due this quarter? 2.6.0
- `YEAR` - Is this task due this year? 2.3.0
- `ACTIVE` - Is the task active, i.e. does it have a start date?
- `SCHEDULED` - Is the task scheduled, i.e. does it have a scheduled date?
- `PARENT` - Is the task a hidden parent recurring task? 2.3.0
- `CHILD` - Is the task a recurring child task?
- `UNTIL` - Does the task expire, i.e. does it have an until date?
- `WAITING` - Is the task hidden, i.e. does it have a wait date?
- `ANNOTATED` - Does the task have any annotations?
- `READY` - Is the task pending, not blocked, and either not scheduled, or scheduled before now. 2.4.0
- `YESTERDAY` - Was the task due yesterday? 2.4.0
- `TOMORROW` - Is the task due tomorrow? 2.4.0
- `TAGGED` - Does the task have any tags?
- `PENDING` - Is the task in the pending state? 2.4.0
- `COMPLETED` - Is the task in the completed state? 2.4.0
- `DELETED` - Is the task in the deleted state? 2.4.0
- `UDA` - Does the task contain any UDA values? 2.5.0
- `ORPHAN` - Does the task contain any orphaned UDA values? 2.5.0
- `PRIORITY` - Does the task have a priority? 2.5.0
- `PROJECT` - Does the task have a project? 2.5.0
- `LATEST` - Is the task the most recently added task? 2.5.0

### filters usage

#### Complex Filters

Some Taskwarrior filters are simple in concept, but the syntax is not that straightforward. For example, to determine whether a task has a due date that falls on the current day, you need to use this filter:

```shell
task due.after:yesterday and due.before:tomorrow list
```

This filters tasks with a due date during the narrow time window of ’today’. Note that it is not sufficient to just specify the date, because due dates all have associated times (defaulting to 0:00:00), and if you want to match the date, you need to consider the time. So for example, this command _does not_ list tasks due today:

```shell
task due:today list
```

Instead, this filter matches tasks with a due date of today, and a time of 0:00. In order to see all tasks due today, you need to provide proper range bracketing.

#### Simplification

Here is where virtual tags can help, by providing a simple tag interface to more complex state conditions of the task. There is a virtual tag, named `TODAY` that can be used in filters, and it means that instead of this filter:

```sh
task due.after:yesterday and due.before:tomorrow list
```

We can now use this:

```shell
task +TODAY list
```

Which is a much simpler way of filtering tasks due today. Because this is a tag interface, we can also invert it:

```shell
task -TODAY list
```

This shows only tasks that are not due today.

Virtual tags are built in to Taskwarrior. They are evaluated at run time, which means they do not require direct metadata, and therefore do not occupy space in the database, but are determined according to the state of the task in the same way that the complex filter example above is determined.

Thus virtual tags combine the simplicity of the tag interface with more complex defined conditions, for convenience.

## Using Dates Effectively

A task does not require a due date, and can simply be a statement of need:

```shell
$ task add Send Alice a birthday card
```

However, this is exactly the kind of task can benefit from having a due date, and perhaps several other dates also.

There are several dates that can decorate a task, each with its own meaning and effects. You can choose to use some, all or none of these, but like all Taskwarrior features, they are there in case your needs require it, but you do not pay a performance or friction penalty by not using them.

### The due Date

Use a `due` date to specify the exact date by which a task must be completed. This corresponds to the last possible moment when the task can be considered on-time. Using our example, we can set the `due` date to be Alice’s birthday (line breaks added for clarity):

```shell
$ task add Send Alice a birthday card \
           due:2016-11-08
```

Now your task has an associated `due` date, to help you determine when you need to work on it. But what effect does this have on Taskwarrior? How can it be used to best advantage?

We call the `due` date of a task ‘metadata’. As such, it is just a piece of data associated with the task, and therefore it can become part of a filter:

```shell
$ task due:today list
...
```

This is one way to find out if any of your tasks are due today. You could also use:

```shell
$ task +TODAY list
...
```

That is an example of a virtual tag, `TODAY`, which is not a real tag, but is something you can query, and is equivalent to the previous example. Additionally, you can use `DUE` which filters tasks that have a due date today or within the next 7 days, or `WEEK` for all tasks due within the current calendar week, or `YESTERDAY`, `TOMORROW`, `MONTH` and `YEAR`.

Note that number of days in which a task is considered `DUE` can be configured using the `rc.due` setting.

You can find tasks that have any due date at all:

```shell
$ task due.any: list
...
```

Or no due date:

```shell
$ task due.none: list
...
```

There is also an `overdue` report that makes use of the `OVERDUE` virtual tag, to show you what is already late. If you run the `calendar` report, your due date will be highlighted on it.

What we see here is that Taskwarrior leverages the metadata to drive various features. Several reports will sort by `due` date, and as we see above, a task that has a due date now belongs on your schedule.

### The scheduled Date

A `scheduled` date is different from a `due` date, and represents the earliest opportunity to work on a task. Let’s continue with the same example above. You need to send a birthday card to Alice, but her birthday isn’t until November, so it’s not the kind of task that can be done in advance. Ideally this would be done a few days ahead of the `due` date:

```shell
$ task add Send Alice a birthday card \
           due:2016-11-08 \
           scheduled:2016-11-04
```

This means that you need to send Alice a birthday card, no later than 2016-11-08, and no earlier than 2016-11-04.

If a task has a `scheduled` date, then once that date passes, the task is considered ready, and there is a `ready` report and a `READY` virtual tag for this:

```shell
$ task ready
...
$ task +READY list
...
```

Tasks that have no `scheduled` date are considered always ready. Again, metadata drives the sophistication of your task list.

### The wait Date

Many people do not like to look at long task lists, finding them daunting, or just distracting. You can add a `wait` date to a task, which has the effect of hiding the task from you until that date. In our example, Alice’s birthday isn’t close yet, so we applied a `scheduled` date to indicate that we should not begin the task yet, as it is not ready. Now let’s add a `wait` date to the task:

```shell
$ task add Send Alice a birthday card \
           due:2016-11-08 \
           scheduled:2016-11-04 \
           wait:november
```

Here the task is given a `wait` date of 2016-11-01, via the useful shortcut ’november’, which means the task will not appear on lists until November. At that time, it will reappear, but it will still not be ready until 2016-11-04.

You can view all the hidden waiting tasks using the `waiting` report:

```shell
$ task waiting
...
```

There is a `WAITING` virtual tag to select these tasks, but note that you have to use the `all` report with it, otherwise you get conflicts with the other reports that specify a ‘pending’ status, because a waiting task is not pending.

### The until Date

Now suppose I miss Alice’s birthday completely. Shame on me. The task would be overdue, but this is the kind of task where I don’t want to complete it late, I’d rather just forget it, and wish Alice a belated happy birthday in person. I could simply delete or complete the task, but there is another option, which is to add an `until` date:

```perl
$ task add Send Alice a birthday card \
           due:2016-11-08 \
           scheduled:2016-11-04 \
           wait:november \
           until:2016-11-10
```

This means that on 2016-11-10, the task self-destructs, and is automatically deleted. This might be the right thing to do for a birthday card task, but is not suitable for a “Pay the rent” task. Beware!

There is a DOM-based shortcut you can use, to make the command above a little more formulaic:

```perl
$ task add Send Alice a birthday card \
           due:2016-11-08 \
           scheduled:due-4d \
           wait:due-7d \
           until:due+2d
```

This is evaluated only at task creation time, so if you change the due date, you also need to change the other dates. Note there is an `UNTIL` virtual tag to show you all tasks that are set to auto-expire.

### Other Dates

There are other dates associated with a task, but these are more for internal use, and are less useful for you.

Each task has an `entry` date which records when it was created. Each completed or deleted task has an `end` date, which records when it was completed or deleted. An active, or started task has a `start` date, but only while it is in the active state. Finally, every task has a `modification` date, which records when it was last modified. This is used as a hint when tasks are being synced.

In addition, you may find you have a use case for a different kind of date for your task lists. For example, you may adhere to an agile development process, and a task may be assigned to a sprint, and that sprint may be identified by its end date. You can add arbitrary dates like this to Taskwarrior by defining a [User Defined Attribute](https://taskwarrior.org/docs/udas/) (UDA) and then storing that metadata with your tasks. In this case, Taskwarrior will do nothing with your UDA but store it, and let you use it in reports and filters.

### Urgency

The presence and values of date metadata in your tasks affects the urgency calculations. For example, if a task is blocked by a dependency, the urgency is reduced. Similarly, tasks that are ready have an elevated urgency.

## DOM - Document Object Model

Taskwarrior has a Document Object Model, or DOM, which defines a way to reference all the data managed by taskwarrior. You may be familiar with the DOM implemented by web browsers that let you access details on a page programmatically. For example:

```javascript
document.getElementById("myAnchor").href;
```

Taskwarrior allows the same kind of data access in a similar form, for example:

```sh
task _get 1.description
```

This references the description text of task 1. There is a [`_get` helper command](https://taskwarrior.org/docs/commands/_get/) that queries data using a DOM reference. Let’s see it in action, by first creating a detailed task.

```shell
task add Buy milk due:tomorrow +store project:Home pri:H
task 1 info

Name          Value
------------- ------------------------------------------
ID            1
Description   Buy milk
Status        Pending
Project       Home
Priority      H
Entered       2014-09-28 21:53:59 (4 seconds)
Due           2014-09-29 00:00:00
Last modified 2014-09-28 21:53:59 (4 seconds)
Tags          store
UUID          c0ab2bf6-b4f5-45c2-8420-18ab4f1ba7e7
Urgency       16.56
                           project     1  *    1 =     1
                          priority     1  *    6 =     6
                              tags   0.8  *    1 =   0.8
                               due  0.73  *   12 =  8.76
```

All the attributes of that task are available via DOM references. Here are some examples:

```shell
$ task _get 1.description
Buy milk

$ task _get 1.uuid
c0ab2bf6-b4f5-45c2-8420-18ab4f1ba7e7

$ task _get c0ab2bf6-b4f5-45c2-8420-18ab4f1ba7e7.id
    1

$ task _get 1.due.year
2014

$ task _get 1.due.julian
272
```

### Supported References

system.version: The version of taskwarrior, for example:

```shell
$ task _get system.version
2.4.0
```

system.os: The operating system, for example:

```shell
$ task _get system.os
FreeBSD
```

rc.<name>: Any configuration value default, with any overrides from the `.taskrc` applied, then with any command line overrides applied last. For example:

```shell
$ task _get rc.data.location
~/.task
```

<attribute>: Any task attribute. Note that no task ID or UUID is specified, so this variant is only useful on the command line like this:

```shell
$ task add Pay rent due:eom wait:'due - 3days'
```

Note that ‘due’ is a DOM reference from earlier on the command line.

<id>.<attribute>: Any attribute of the specified task ID. For example:

```shell
$ task add Fix the leak depends:3 scheduled:3.due
```

This makes the new task dependent on task 3, and scheduled on the due date of task 3. Note that ‘3.due’ is a DOM reference of a specific task.

<uuid>.<attribute>: Any attribute of the specified task UUID, as above.

Any attribute that is of type `date` can be directly accessed as a date, or it can be accessed by the elements of that date. For example:

- `<date>.year` - 2.4.0 The year, for example:
  ```shell
  $ task _get 1.due.year
  2014
  ```
- `<date>.month` - 2.4.0 The month, for example:
  ```shell
  $ task _get 1.due.month
  9
  ```
- `<date>.day` - 2.4.0 The day of the month, for example:
  ```shell
  $ task _get 1.due.day
  29
  ```
- `<date>.week` - 2.4.0 The week number of the date, for example:
  ```shell
  $ task _get 1.due.week
  40
  ```
- `<date>.weekday` - 2.4.0 The numbered weekday of the date, where 0 is Sunday and 6 is Saturday. For example:
  ```shell
  $ task _get 1.due.weekday
  1
  ```
- `<date>.julian` - 2.4.0 The Julian day of the date, which is the day number of the date in the year. For example, January 1st is 1, February 10th is 41. For example:
  ```shell
  $ task _get 1.due.julian
  272
  ```
- `<date>.hour` - 2.4.0 The hour of the day, for example:
  ```shell
  $ task _get 1.due.hour
  0
  ```
- `<date>.minute` - 2.4.0 The minute of the hour, for example:
  ```shell
  $ task _get 1.due.minute
  0
  ```
- `<date>.second` - 2.4.0 The seconds of the minute, for example:
  ```shell
  $ task _get 1.due.second
  0
  ```

Tags can be accessed as a single item as an `<attribute>`, of the individual tags can be accessed:

- `tags.<literal>` - 2.4.0 Direct access, per-tag, for example:
  ```shell
  $ task _get 1.tag.home
  home
  ```
  If the tag is present, it is shown, otherwise the result is blank, and Taskwarrior exits with a non-zero status.
  ```shell
  $ task _get 1.tag.DUE
  DUE
  $ task _get 1.tag.OVERDUE
  ```
  Workѕ for virtual tags too, in the same manner.

Annotations are compound data structures, with two elements, which are `description` and `entry`. Annotations are accessed by an ordinal.

- `annotations.<N>.description` - 2.4.0 The description of the Nth annotation, for example:
  ```shell
  $ task _get 1.annotations.0.description
  ```
- `annotations.<N>.entry` - 2.4.0 The creation timestamp of the Nth annotation, for example:
  ```shell
  $ task _get 1.annotations.0.entry
  ```
  Note that because `entry` is of type `date`, the individual elements can be addressed, as above, for example:
  ```shell
  $ task _get 1.annotations.0.entry.year
  2014
  ```

UDA values can be accessed in the same manner.
