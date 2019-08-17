---
template: post
title: How to build an email reply feature
slug: how-to-build-an-email-reply-feature
draft: true
date: 2019-08-17T10:15:46.844Z
description: >-
  How to build an email reply feature by which the end users of your app can
  reply to an email notification to post a message in your app
category: engineering
tags:
  - engineering
---
## Who this post is for ✅

Engineers or product managers who aren't afraid of a sprinkle of technical references.

## What is this feature (TODO use gifs of examples) 📩

![](/media/screenshot-2019-08-17-at-12.22.18.png)

## Why it may create value for your users ⏳

People use your app to accomplish a goal, not to gaze confusingly at your snazzy new interface where half the buttons are not where they used to be. Their goal is not navigating your apps experimental navigation, nor trying to remember which email and password or social login they used for your app's login with your _must contain at least 12 characters, !$_- password policy_.

Their goal I describe in this post is to respond to a new message notification in their email inbox, and the fastest and simplest way for them to do that is to reply to the email. 

> _If your app has message threads that are transactional in nature, and you care about transaction success rates, you probably want to provide this shortcut for users. See limitations at the end of this article for a look into the exclusions for this rule._

## How does it work (high level) 🔀

1. Someone posts to a message thread, and another user, let's call him _Jonny_ is subscribed to receive email notifications of new messages to this thread.
2. Jonny receives an email notification from your app's email servers containing the contents of the message and who sent it, to which thread. This email also indicates that Jonny can reply to the email to respond into the thread.
3. Jonny hits reply, and the original email's 'reply-to' email address is automatically used to fill the 'To' field of his email. This 'reply-to' email address contains at least information to uniquely identify the messaging thread, usually via a thread id. Jonny writes his message in the email body section and presses send
4. The email server which your domain's DNS indicates emails should be sent to receives the new email, it looks at the 'To' field, 'Body' field and other fields and send along the new message information to your apps API in order to add a new message to your database.
5. The message is added to the database and can be seen in your app's interface. Email notifications to the other participants may also be sent

## How would one build it  🏗

I'm assuming you have the infrastructure to send notification emails to users in which you can specify the _Reply-To_ header. If you don't have this part, I recommend using a third party service such as _Sendgrid_, _Mandrill_ or _Customer.io_ and following their guides to set up sending emails. We use Sendgrid and their API allows us to send emails including the \`replyTo\` email header.

**Problem:** How do you set up a program that can receive emails and parse their contents?

When I was trying to figure how I could solve this problem, I asked Andris Reinman, an open source contributor to email tools such as Wildduck I found on Github who kindly helped me out and replied with the following: 

> The most simple solution would be to set up a custom SMTP server application to act as the MX. If you can write Node.js, then I’d suggest you to check out this example of a SMTP server script: https://github.com/nodemailer/smtp-server/blob/master/examples/server.js
>
> What you’d have to do with that example would be to check if the recipient seems valid in the onRcptTo handler and then in onData handler send the message to wherever you want to, for example upload to an URL. The stream object is a standard readable stream and it contains the entire rfc822 message.

Luckily I could code in Node.js and his advice prevented me from trying to set up a full email server such as Mailin or Wildduck, both very complex to setup and maintain projects. 



**Mini problem:** 

****

**Problem:** How to ensure the sender is authorised to post into the message thread and which user they are in that thread?



## Limitations & Considerations 🚧

Limited formatting

Error handling (Send confirmation email? Send failure email?)

Interactive media

Time or order sensitive messages

Security (and forwarding of emails)

Change of email address

Revoking access

If you want people to come to site instead to convert them on other goals

## 

## A little bit about me and why I wrote this post 🤔

During the day I'm the CTO and cofounder at Deedmob, a startup in Amsterdam building software to make volunteering better. Whilst on paper I may be a CTO, I spend most of my time doing product management and I only spend comparatively little time managing engineering and writing code. We built this feature at Deedmob about a year ago to help our customers respond to messages faster, and at the time I couldn't find any guide online, nor did I know much about how email server work. I spent some time meandering around how open source email servers work and how I could implement this feature without going down the rabbit hole of email protocol and server complexity before I asked for help from an expert I found on github, who kindly helped me narrow my search. In the spirit of paying it forward I've written this post in the hopes that it helps someone else in a similar position to me at that time.
