---
title: RRule Trigger Tutorial
visible_title: "RRule Trigger Tutorial"
active_sub_menu_id: site_mnu_docs_tutorials
---
# RRuleTrigger Tutorial

## Introduction {#RRuleTriggersTutorial-Introduction}

<tt>RRuleTrigger</tt> fires a job on a schedule described by an
<a href="https://datatracker.ietf.org/doc/html/rfc5545#section-3.3.10">RFC&nbsp;5545 <tt>RRULE</tt></a>
(the iCalendar "recurrence rule"). It is the RRULE analogue of <tt>CronTrigger</tt>: where a cron
expression is a compact, positional string, an <tt>RRULE</tt> is a set of named parts such as
<tt>FREQ=WEEKLY;BYDAY=MO,WE,FR;BYHOUR=10;BYMINUTE=0</tt>.

If you already receive schedules in iCalendar form (e.g. from a calendaring system), or you prefer
the self-describing RRULE syntax, <tt>RRuleTrigger</tt> lets you use them directly.

Unlike a cron expression, an <tt>RRULE</tt> is evaluated relative to a start instant &#45; the RFC&nbsp;5545
<tt>DTSTART</tt>. In Quartz that anchor is supplied by the trigger's own start-time. As with cron, the
start-time acts as a lower bound: the start instant only fires if it actually matches the rule.

## Format {#RRuleTriggersTutorial-Format}

An RRULE is a string made up of <tt>PART=VALUE</tt> pairs separated by semicolons. <tt>FREQ</tt> is the
only mandatory part. Any time component (second, minute, hour) that the rule does not constrain defaults
to the corresponding component of the trigger's start-time.

<table cellpadding="3" cellspacing="1">
    <tbody>
        <tr>
            <th>Part</th>
            <th>Mandatory</th>
            <th>Allowed Values</th>
        </tr>
        <tr>
            <td><tt>FREQ</tt></td>
            <td>YES</td>
            <td><tt>SECONDLY</tt>, <tt>MINUTELY</tt>, <tt>HOURLY</tt>, <tt>DAILY</tt>, <tt>WEEKLY</tt>, <tt>MONTHLY</tt>, <tt>YEARLY</tt></td>
        </tr>
        <tr>
            <td><tt>INTERVAL</tt></td>
            <td>NO</td>
            <td>A positive integer (default 1). Fire every <tt>INTERVAL</tt> units of the <tt>FREQ</tt>.</td>
        </tr>
        <tr>
            <td><tt>COUNT</tt></td>
            <td>NO</td>
            <td>A positive integer &#45; the number of occurrences to fire. Mutually exclusive with <tt>UNTIL</tt>.</td>
        </tr>
        <tr>
            <td><tt>UNTIL</tt></td>
            <td>NO</td>
            <td>An (inclusive) end date-time, e.g. <tt>20261231T120000Z</tt>. Mutually exclusive with <tt>COUNT</tt>.</td>
        </tr>
        <tr>
            <td><tt>WKST</tt></td>
            <td>NO</td>
            <td>The day the week starts on: <tt>SU MO TU WE TH FR SA</tt> (default <tt>MO</tt>).</td>
        </tr>
        <tr>
            <td><tt>BYMONTH</tt></td>
            <td>NO</td>
            <td>1-12</td>
        </tr>
        <tr>
            <td><tt>BYWEEKNO</tt></td>
            <td>NO</td>
            <td>1-53 or -53..-1 (only with <tt>FREQ=YEARLY</tt>)</td>
        </tr>
        <tr>
            <td><tt>BYYEARDAY</tt></td>
            <td>NO</td>
            <td>1-366 or -366..-1</td>
        </tr>
        <tr>
            <td><tt>BYMONTHDAY</tt></td>
            <td>NO</td>
            <td>1-31 or -31..-1 (negative counts back from the end of the month)</td>
        </tr>
        <tr>
            <td><tt>BYDAY</tt></td>
            <td>NO</td>
            <td><tt>SU MO TU WE TH FR SA</tt>, optionally prefixed by an ordinal such as <tt>2MO</tt> ("2nd Monday")
                or <tt>-1FR</tt> ("last Friday"). Ordinals are only valid with <tt>FREQ=MONTHLY</tt> or <tt>YEARLY</tt>.</td>
        </tr>
        <tr>
            <td><tt>BYHOUR</tt></td>
            <td>NO</td>
            <td>0-23</td>
        </tr>
        <tr>
            <td><tt>BYMINUTE</tt></td>
            <td>NO</td>
            <td>0-59</td>
        </tr>
        <tr>
            <td><tt>BYSECOND</tt></td>
            <td>NO</td>
            <td>0-60</td>
        </tr>
        <tr>
            <td><tt>BYSETPOS</tt></td>
            <td>NO</td>
            <td>1-366 or -366..-1 &#45; selects the nth occurrence out of the set produced within each interval
                (e.g. combine with <tt>BYDAY</tt> to pick the first weekday of the month).</td>
        </tr>
    </tbody>
</table>

Each <tt>BY&#42;</tt> part may be a comma-separated list, for example <tt>BYDAY=MO,WE,FR</tt> or
<tt>BYMONTHDAY=1,15</tt>.

## Building the Trigger {#RRuleTriggersTutorial-Building}

Use <tt>RRuleScheduleBuilder</tt> (typically via a static import) to attach an RRULE schedule to a trigger:

<pre>
import static org.quartz.RRuleScheduleBuilder.rruleSchedule;
import static org.quartz.TriggerBuilder.newTrigger;

// Fire at 10:00am every Monday, Wednesday and Friday
RRuleTrigger trigger = newTrigger()
    .withIdentity("trigger1", "group1")
    .withSchedule(rruleSchedule("FREQ=WEEKLY;BYDAY=MO,WE,FR;BYHOUR=10;BYMINUTE=0;BYSECOND=0"))
    .startAt(startTime) // acts as the RFC 5545 DTSTART anchor
    .build();
</pre>

<tt>RRuleScheduleBuilder</tt> also provides a few convenience factory methods:
<tt>dailyAtHourAndMinute(int, int)</tt>, <tt>weeklyOnDayAndHourAndMinute(int, int, int)</tt> and
<tt>monthlyOnDayAndHourAndMinute(int, int, int)</tt>.

## Examples {#RRuleTriggersTutorial-Examples}

<table cellpadding="3" cellspacing="1">
    <tbody>
        <tr>
            <td width="360">**Expression**</td>
            <td>**Meaning**</td>
        </tr>
        <tr>
            <td><tt>FREQ=DAILY;BYHOUR=12;BYMINUTE=0;BYSECOND=0</tt></td>
            <td>Fire at 12pm (noon) every day</td>
        </tr>
        <tr>
            <td><tt>FREQ=DAILY;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am every day</td>
        </tr>
        <tr>
            <td><tt>FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am every Monday, Tuesday, Wednesday, Thursday and Friday</td>
        </tr>
        <tr>
            <td><tt>FREQ=WEEKLY;BYDAY=WE;BYHOUR=14;BYMINUTE=10,44;BYSECOND=0</tt></td>
            <td>Fire at 2:10pm and at 2:44pm every Wednesday</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYMONTHDAY=15;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am on the 15th day of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYMONTHDAY=-1;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am on the last day of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYMONTHDAY=-2;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am on the 2nd-to-last day of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYDAY=-1FR;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am on the last Friday of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYDAY=3FR;BYHOUR=10;BYMINUTE=15;BYSECOND=0</tt></td>
            <td>Fire at 10:15am on the third Friday of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYMONTHDAY=1,15;BYHOUR=10;BYMINUTE=0;BYSECOND=0</tt></td>
            <td>Fire at 10:00am on the 1st and 15th of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=MONTHLY;BYDAY=MO,TU,WE,TH,FR;BYSETPOS=1;BYHOUR=9;BYMINUTE=0;BYSECOND=0</tt></td>
            <td>Fire at 9:00am on the first weekday of every month</td>
        </tr>
        <tr>
            <td><tt>FREQ=DAILY;INTERVAL=5;BYHOUR=12;BYMINUTE=0;BYSECOND=0</tt></td>
            <td>Fire at noon every 5 days</td>
        </tr>
        <tr>
            <td><tt>FREQ=YEARLY;BYMONTH=11;BYMONTHDAY=11;BYHOUR=11;BYMINUTE=11;BYSECOND=0</tt></td>
            <td>Fire every November 11th at 11:11am</td>
        </tr>
        <tr>
            <td><tt>FREQ=DAILY;BYHOUR=12;BYMINUTE=0;BYSECOND=0;COUNT=10</tt></td>
            <td>Fire at noon every day, but only 10 times</td>
        </tr>
        <tr>
            <td><tt>FREQ=DAILY;BYHOUR=12;BYMINUTE=0;BYSECOND=0;UNTIL=20261231T120000Z</tt></td>
            <td>Fire at noon every day, up to and including 2026-12-31</td>
        </tr>
        <tr>
            <td><tt>FREQ=SECONDLY;INTERVAL=30;BYDAY=SA,SU</tt></td>
            <td>Fire every 30 seconds, but only on Saturdays and Sundays</td>
        </tr>
    </tbody>
</table>

## Notes {#RRuleTriggersTutorial-Notes}

+ The trigger's **start-time is the RFC&nbsp;5545 <tt>DTSTART</tt>** anchor. It is a lower bound &#45; the
    start instant itself only fires if it matches the rule &#45; and it is what <tt>INTERVAL</tt> and
    <tt>COUNT</tt> are measured from.

+ Any time component the rule does not specify (<tt>BYSECOND</tt>, <tt>BYMINUTE</tt>, <tt>BYHOUR</tt>)
    defaults to the corresponding component of the start-time.

+ <tt>COUNT</tt> counts the number of occurrences that actually fire; <tt>UNTIL</tt> is an inclusive upper
    bound. The two are mutually exclusive.

+ As with cron, be careful when firing between the hours of the morning when "daylight savings" changes
    occur in your locale, as the time shift can cause a skip or a repeat.

+ A runnable example is provided in the Quartz distribution at
    <tt>examples/src/main/java/org/quartz/examples/example16/RRuleTriggerExample.java</tt>
    (<tt>../gradlew :examples:runExample16</tt>).

+ For the full specification of the rule parts and their interactions, see
    <a href="https://datatracker.ietf.org/doc/html/rfc5545#section-3.3.10">RFC&nbsp;5545 &sect;3.3.10</a>.
