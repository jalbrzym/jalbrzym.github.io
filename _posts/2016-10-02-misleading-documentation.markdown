---
layout: post
title:  "Misleading documentation - Exchange meeting requests"
date:   2016-10-02 23:30:00 +0200
categories: 
---

### Hello Exchange!

A few weeks ago I had the opportunity to work with on-premise Exchange Server 2013. Microsoft provides a powerful interface to manage Exchange mailboxes called [EWS (Exchange Web Services)](https://msdn.microsoft.com/en-us/library/office/dd877012(v=exchg.150).aspx). Unfortunately, communication between EWS and Exchange Server is based on heavy-weight SOAP messages. Instead of creating requests and parsing responses manually, I decided to use [EWS Managed API](https://github.com/OfficeDev/ews-managed-api/blob/master/README.md). This is an open-source library, which wraps SOAP communication and provides API for .NET developers in more object-oriented manner. 

### Schedule a meeting

My task was to save information about meeting entered by organizer and (after successfully validation and data persistence) to send meeting request to all attendees. The first part was quite simple - show a form, bind data and save into database.

The second part of the task was more interesting. Before I started implementation I opened a EWS Managed API documentation and I found [how to create and send a meeting request](https://msdn.microsoft.com/en-us/library/office/dn495611(v=exchg.150).aspx#Anchor_2). The sample code looks so easy. It consists in three simple steps:

* create an `Appointment` object representing a meeting
* set meeting data using propper `Appointment` object's properties (start date, location, attendees, etc.)
* call `Save` method of an `Appointment` object  

<pre><code class="cs">Appointment meeting = new Appointment(service);

meeting.Subject = "Team building exercise";
meeting.Body = "Let's learn to really work as a team and then have lunch!";
meeting.Start = DateTime.Now.AddDays(2);            
meeting.End = meeting.Start.AddHours(4);
meeting.Location = "Conference Room 12";
meeting.RequiredAttendees.Add("Jan.Kowalski@exchange.local");
meeting.OptionalAttendees.Add("Jan.Nowak@exchange.local");
meeting.ReminderMinutesBeforeStart = 60;

meeting.Save(SendInvitationsMode.SendToAllAndSaveCopy);
</code></pre>

Everything seemed to work fine. Meeting request was sent correctly and all participants received it. Unexpectedly, when I tried to accept or decline meeting request, I've got a following error message: 

 
`The action couldn't be completed. A conflicting change was made to the item on the server`.

![My helpful screenshot]({{ site.url }}/assets/posts/2016-10-02-misleading-documentation/exchange-error.png)

### Problem investigation

Firstly, I asked Google and Stack Overflow about this error. They said nothing - it didn't bode well. 

The next step was to check, if the client application makes appropriate calls to the Exchange Server. I used Fiddler to trace requests performed by EWS Managed API. There was nothing suspicious. I saw well-formed requests with valid SOAP messages in their bodies.

"If the client works good, there must be a server-side issue", I said to myself. I made sure, that no one beside me was using the same Exchange Server. Then I created a meeting request manually - it worked fine. I verified Exchange Server settings and mailboxes permissions, but everything were set as it should be. I asked sys-admins to install the latest Cumulative Update for Exchange Server. Unfortunately, the problem remained unresolved. 

I wasted a couple of hours and I went back to square one. 

### Time is my enemy

I looked at the documentation and I compared a body of the sample request from documentation and what I got using the same sample code. 

<pre><code class="xml">&lt;t:CalendarItem&gt;
    &lt;t:Subject&gt;Team building exercise&lt;/t:Subject&gt;
    &lt;t:Body BodyType="HTML"&gt;Let's learn to really work as a team and then have lunch!&lt;/t:Body&gt;
    &lt;t:ReminderMinutesBeforeStart&gt;60&lt;/t:ReminderMinutesBeforeStart&gt;
    &lt;t:Start&gt;2013-09-21T16:00:00.000Z&lt;/t:Start&gt;
    &lt;t:End&gt;2013-09-21T20:00:00.000Z&lt;/t:End&gt;
    &lt;t:Location&gt;Conference Room 12&lt;/t:Location&gt;
    &lt;t:RequiredAttendees&gt;
        ...
    &lt;/t:RequiredAttendees&gt;
    &lt;t:OptionalAttendees&gt;
        ...
    &lt;/t:OptionalAttendees&gt;
    &lt;t:MeetingTimeZone TimeZoneName="Pacific Standard Time" /&gt;
&lt;/t:CalendarItem&gt;
</code></pre>

They were nearly identical except for the different time of start and end of the meeting. It's obvious that they were different, because in the sample code the start time is set on current time (`DateTime.Now`). One thing was suspicious - the start and end time were set at full hour.

I set the start and end time manually on the exact point in time and sent it. 

<pre><code class="cs">app.Start = new DateTime(year: 2016, month: 9, day: 30, hour: 19, minute: 0, second: 0);
app.End = new DateTime(year: 2016, month: 9, day: 30, hour: 20, minute: 0, second: 0);
</code></pre>

All attendees received the meeting request. When I tried to accept or decline, finally Exchange set an attendee status correctly and notified a meeting organizer about it. Great!

To make investigation be completed, I performed an experiment. I set manually the start time using the value of miliseconds greater than 0.

<pre><code class="cs">app.Start = new DateTime(year: 2016, month: 9, day: 30, hour: 19, minute: 0, second: 0, millisecond: 1);
</code></pre>

Result? After I tried to accept the meeting request, I saw the same error again!

The author of documentation had 1 out of 1000 probability for getting current time of 0 miliseconds. Coincidence? Is writer of documentation a lucky guy? ( ͡° ͜ʖ ͡°)

### Lessons learned

* If you write a documentation, please check twice that all of sample code snippets work correctly and get **expected**, **repeatable** results. Put a disclaimer whenever on some conditions examples may fail.
* It is important to be vigilant if you work with a documentation of a new or unfamiliar technology. Write **integration tests** which cover basic use cases and edge cases. Such tests can prove correctness of a manual. Furthermore, one day your external API provider can make a change in a webservice functionality or you can update a library to the newest version. Those tests will help you quickly detect unexpected breaking changes in third-party technologies used by your application.
* An error message should be **helpful in troubleshooting**. Show precise and well-formatted description of encountered problem. Furthermore, you can suggest steps to solve a problem. Avoid displaying meaningless (ex. "*An error has occured*") or misleading error messages, which could only increase frustration of your users.  
* I encourage you to **share solutions** of such problems on the internet. You have nothing to loose, but you can save a lot of hours of other developers which will encounter similar problem.
* Exchange Server, for some reason, doesn't like overly punctual people. Bear in mind that Exchange accept time accurate to the second. You have to **round milliseconds to the nearest second**, unless Exchange may behave strange.

