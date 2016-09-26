---
layout: post
title:  "Misleading Exchange documentation"
date:   2016-09-21 20:30:00 +0200
categories: 
---

### Hello Exchange!

A few weeks ago I had the opportunity to work with on-premise Exchange Server 2013. Microsoft provides a powerful interface to manage Exchange mailboxes called EWS (Exchange Web Services). Unfortunately, communication between EWS and Exchange Server is based on heavy-weight SOAP messages. Instead of creating requests and parsing responses manually, I decided to use EWS Managed API v2.0. This is an open-source library, which wraps SOAP communication and provides API for .NET developers in more object-oriented manner. 

### Schedule a meeting

Everything seemed to work fine. Meeting request was sent correctly and all participants received it. Unexpectedly, when I tried to accept or decline meeting request, I've got an error message: 


`The action couldn't be completed. A conflicting change was made to the item on the server`.

### Problem investigation

<pre><code class="xml">&lt;soap:Body&gt;
    &lt;m:CreateItem SendMeetingInvitations="SendToNone"&gt;
        &lt;m:Items&gt;
        &lt;t:CalendarItem&gt;
            &lt;t:Subject&gt;Tennis lesson&lt;/t:Subject&gt;
            &lt;t:Body BodyType="HTML"&gt;Focus on backhand this week.&lt;/t:Body&gt;
            &lt;t:ReminderDueBy&gt;2013-09-19T14:37:10.732-07:00&lt;/t:ReminderDueBy&gt;
            &lt;t:Start&gt;2013-09-21T19:00:00.000Z&lt;/t:Start&gt;
            &lt;t:End&gt;2013-09-21T20:00:00.000Z&lt;/t:End&gt;
            &lt;t:Location&gt;Tennis club&lt;/t:Location&gt;
            &lt;t:MeetingTimeZone TimeZoneName="Pacific Standard Time" /&gt;
        &lt;/t:CalendarItem&gt;
        &lt;/m:Items&gt;
    &lt;/m:CreateItem&gt;
&lt;/soap:Body&gt;
</code></pre>


The author of documentation had 1 out of 1000 probability for getting current time of 0 miliseconds. Coincidence? Is writer of documentation a lucky guy? 

### Lessons learned

* If you write a documentation, please check twice that all of sample code snippets work correctly and get **expected**, **repeatable** results. Put a disclaimer whenever on some conditions examples may fail.
* It is important to be vigilant if you work with a documentation of a new or unfamiliar technology. Write integration tests which cover basic use cases and edge cases. Such tests can prove correctness of a documentation. Furthermore, 
* I encourage you to share solutions of such problems on the internet. You have nothing to loose, but you can save a lot of hours of other developers which encounter similar problem.
* Exchange Server, for some reason, doesn't like overly punctual people. Bear in mind that Exchange accept time accurate to the second. You have to round milliseconds to nearest second, unless Exchange may behave strange.





