---
layout: post
title:  "Misleading Exchange documentation"
date:   2016-09-21 20:30:00 +0200
categories: 
---

### Hello Exchange!

A few weeks ago I had the opportunity to work with on-premise Exchange Server 2013. Microsoft provides a powerful interface to manage Exchange mailboxes called EWS (Exchange Web Services). Unfortunatelly, communication between EWS and Exchange Server is based on heavy-weight SOAP messages. Instead of creating requests and parsing responses manually, I decided to use EWS Managed API v2.0. This an open-source library, which wraps SOAP communication and provides API for developers in more object-oriented manner. 

