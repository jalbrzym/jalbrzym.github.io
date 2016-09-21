---
layout: post
title:  "Misleading Exchange documentation"
date:   2016-09-21 20:30:00 +0200
categories: 
---

### Hello Exchange!

A few weeks ago I had the opportunity to work with on-premise Exchange Server 2013. Microsoft provides a powerful interface to manage Exchange mailboxes called EWS (Exchange Web Services). Unfortunatelly, communication between EWS and Exchange Server is based on heavy-weight SOAP messages. Instead of creating requests and parsing responses manually, I decided to use EWS Managed API v2.0. This is an open-source library, which wraps SOAP communication and provides API for developers in more object-oriented manner. 

### Schedule a meeting




### Lessons learned

* If you're writing documentation, please check twice if all of sample code snippets work correctly and get expected, repeatable results. 
* It is important to be vigilant with documentation of technology that is new for you.
* I encourage you to share solutions of such problems in the internet. You have nothing to loose, but you can save a lot of hours of other developers which encounter similar problem.
* Exchange Server, for some reason, doesn't like overly punctual people. Bear in mind that Exchange accept time accurate to the second. You have to round milliseconds to nearest second, unless Exchange may behave strange.





