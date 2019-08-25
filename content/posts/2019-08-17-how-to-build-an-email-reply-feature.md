---
template: post
title: How to build an email reply feature
slug: how-to-build-an-email-reply-feature
draft: false
date: 2019-08-17T10:15:46.844Z
description: >-
  How to build a feature that makes it possible to reply to an email
  notification directly to post a message in your app
category: engineering
tags:
  - engineering
---
## Who this post is for ✅

Engineers and parts may be useful to product managers who are also interested in building a reply-to email based feature for their app.

## What is this feature 📩

Many popular services in which you can have a conversation (such as support software) allow you to respond to an email notification directly instead of logging into their interface to reply to a message. 

![](/media/screenshot-2019-08-17-at-16.17.05.png)

![](/media/screenshot-2019-08-17-at-16.17.11.png)

_An example of a github notification email you can reply to_

## Why it may create value for your users ⏳

The goal of your user which I describe in this post is to respond to a new message notification in their email inbox. The fastest and simplest way for them to achieve their goal is to reply to the email.

> _If your app has message threads that are transactional in nature, and you care about transaction success rates, you probably want to provide this shortcut for users. See limitations at the end of this article for a look into the exclusions for this rule._

An engineer following this guide should be able to be implement the technical parts of this feature in less than a day barring any extensive security auditing needs.

## How does it work (high level) 🔀

1. _Johnny_ is subscribed to receive email notifications of new messages to a message thread in your app.
2. Johnny receives an email notification from your app's email servers containing (1) the contents of a new message (2) who sent it, (3) in which thread it is. The email notes that Johnny can reply to the email to respond into the thread.
3. Johnny hits reply, and the original email's 'reply-to' email address is automatically used to fill the 'To' field of his email. This 'reply-to' email address contains at least information to uniquely identify the messaging thread, usually via a thread id. Johnny writes his message in the email body section and presses send
4. The email server which your domain's [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) points to receives the new email, among other things it looks at the 'To' and 'Body' fields and sends along the new message to your app's [API](https://en.wikipedia.org/wiki/Application_programming_interface) in order to add a new message to your database.
5. The message is added to your database and can be seen in your app's interface. Email notifications to the other participants may also be sent at this time.

![](/media/screenshot-2019-08-17-at-12.22.18.png "Reply to zendesk support threads")
_Note the reply-to address in this example from typeform_

## How would one build it  🏗

_I recommend skipping to [Limitations & Considerations](#limitations) if you're not interested in the technical parts._

I'm assuming you have the infrastructure to send notification emails to users in which you can specify the _Reply-To_ header. If you don't have this part, I recommend using a third party service such as _Sendgrid_, _Mandrill_ or _Customer.io_ and following their guides to set up sending emails.

**Problem:** How do you set up a program that can receive emails and parse their contents?

When I was trying to figure how I could solve this problem, I asked Andris Reinman, an open source contributor to email tools such as Wildduck I found on Github who kindly helped me out and replied with the following: 

> The most simple solution would be to set up a custom SMTP server application to act as the MX. If you can write Node.js, then I’d suggest you to check out this example of a SMTP server script: https://github.com/nodemailer/smtp-server/blob/master/examples/server.js
>
> What you’d have to do with that example would be to check if the recipient seems valid in the onRcptTo handler and then in onData handler send the message to wherever you want to, for example upload to an URL. The stream object is a standard readable stream and it contains the entire rfc822 message.

Luckily I could code in Node.js and his advice prevented me from trying to set up a full email server such as Mailin or Wildduck, both very complex to setup and maintain. 

Here's that code simplified to the bits that we need

```js
/* eslint no-console: 0 */

'use strict';

// Replace '../lib/smtp-server' with 'smtp-server' when running this script outside this directory

const SMTPServer = require('../lib/smtp-server').SMTPServer;

const SERVER_PORT = 2525;

const SERVER_HOST = false;

// Connect to this example server by running

//   telnet localhost 2525

// or

//   nc -c localhost 2525

// Authenticate with this command (username is 'testuser' and password is 'testpass')

//   AUTH PLAIN dGVzdHVzZXIAdGVzdHVzZXIAdGVzdHBhc3M=

// Setup server

const server = new SMTPServer({

    // log to console

    logger: true,

    // not required but nice-to-have

    banner: 'Welcome to My Awesome SMTP Server',

    // disable STARTTLS to allow authentication in clear text mode

    disabledCommands: ['AUTH', 'STARTTLS'],

    // By default only PLAIN and LOGIN are enabled

    authMethods: ['PLAIN', 'LOGIN', 'CRAM-MD5'],

    // Accept messages up to 10 MB

    size: 10 * 1024 * 1024,

    // allow overriding connection properties. Only makes sense behind proxy

    useXClient: true,

    hidePIPELINING: true,

    // use logging of proxied client data. Only makes sense behind proxy

    useXForward: true,

    // Handle message stream

    onData(stream, session, callback) {

        stream.pipe(process.stdout);

        stream.on('end', () => {

            let err;

            if (stream.sizeExceeded) {

                err = new Error('Error: message exceeds fixed maximum message size 10 MB');

                err.responseCode = 552;

                return callback(err);

            }

            callback(null, 'Message queued as abcdef'); // accept the message once the stream is ended

        });
    }

});

server.on('error', err => {
    console.log('Error occurred');
    console.log(err);
});

// start listening

server.listen(SERVER_PORT, SERVER_HOST);
```

The way this node server works is it runs an SMTP server that listens to the 2525 port that will call your \`onData\` callback function when a new email is received.

The \`onData(stream, session, callback)\` callback function gives you the email contents as a  _stream_ , specifically a _SMTPServerDataStream._ I don't know the internals here, and you can look into them deeper if you'd like. For our purposes it suffices to know that we can get the \`to\` address via

```js
const to = session.envelope.rcptTo[0].address;
```
and the \`from\` address via

```js
    const from = session.envelope.mailFrom.address;
```

The stream gives us the raw email contents which includes a bunch of headers, previous messages in the thread and email signatures. We want the contents that the person actually typed for this email, and so we need to parse the raw email contents and then only pick the bits that we need_._

To parse the raw email contents I found a library called \`mailparser\` which is simple enough (I'm using babel btw)

```js
const { simpleParser } = require('mailparser');

// ...

onData(stream, session, callback) {

simpleParser(stream)

      .then(mail => {

        // use mail.text 

...
```

Then we want to remove the bits of \`mail.text\` that are the email signature or email history. We can use the npm library \`mailstrip\` for this task.

```js

const { simpleParser } = require('mailparser');const mailstrip = require('mailstrip');
// ...

onData(stream, session, callback) {
simpleParser(stream)
      .then(mail => {const parsedAndStrippedText = mailstrip(mail.text)
// ...
```

We now have the message contents, and which email address sent it, and to which email address it was sent. So we have all the information we need to send a new message request to our API. In full this looks like:

```js
const SMTPServer = require('smtp-server').SMTPServer;
const mailstrip = require('mailstrip');
const { simpleParser } = require('mailparser');
import request from 'request';

const SERVER_PORT = 2525;

const SERVER_HOST = false;

// Setup server

const server = new SMTPServer({

  // log to console

  logger: true,

  // not required but nice-to-have

  banner: 'Welcome to the Deedmob email-reply-service server',

  // disable STARTTLS to allow authentication in clear text mode

  disabledCommands: ['AUTH', 'STARTTLS'],

  // By default only PLAIN and LOGIN are enabled

  authMethods: ['PLAIN', 'LOGIN', 'CRAM-MD5'],

  // Accept messages up to 10 MB

  size: 10 * 1024 * 1024,

  // allow overriding connection properties. Only makes sense behind proxy

  useXClient: true,

  hidePIPELINING: true,

  // use logging of proxied client data. Only makes sense behind proxy

  useXForward: true,

  // Validate RCPT TO envelope address. Example allows all addresses that do not start with 'deny'

  // If this method is not set, all addresses are allowed

  // onRcptTo(address, session, callback) {

  //   if (!(/^user-/i.test(address.address) || /^org-/i.test(address.address))) {

  //     return callback(new Error('Not accepted'));

  //   }

  //

  //   callback();

  // },

  // Handle message stream

  onData(stream, session, callback) {

    const to = session.envelope.rcptTo[0].address;

    const from = session.envelope.mailFrom.address;

    simpleParser(stream)

      .then(mail => {
        console.log(session);

        request(

          {

            method: 'POST',

            url: `https://www.mywebsite.com/api/conversations/email-reply`,

            json: true,

            body: {

              to: to,

              from: from,

              message: mailstrip(mail.text),

            },

            headers: {

              'X-Requested-With': 'XMLHttpRequest',

            },

          },

          (error, response, body) => {

            // HANDLE ERRORS

            if (error) console.error(error);

            else callback(null, 'Message sent'); // accept the message once the stream is ended

          }

        );

      })

      .catch(err => {

        console.log(err);

      });

  },

});

server.on('error', err => {

  console.log('Error occurred');

  console.log(err);

});

// start listening

server.listen(SERVER_PORT, SERVER_HOST);
```

Note I'm using babel so I also have an \`serverEntry.js\` file which just includes

```js
require('babel-register');

require('./server');
```

which I run with \`node serverEntry.js\`.

**So how do I deploy this?**

Well we need to be able to expose the 2525 port, so I whipped together a \`Dockerfile\`

```docker
# specify the node base image with your desired version node:<version>

FROM node:8

# Create app directory

# WORKDIR /usr/src/app

# Install app dependencies

# A wildcard is used to ensure both package.json AND package-lock.json are copied

# where available (npm@5+)

COPY package*.json ./

RUN npm install

# If you are building your code for production

# RUN npm install --only=production

# Bundle app source

COPY . .

EXPOSE 2525

CMD [ "npm", "start" ]
```

which does this for me. You can deploy this wherever you can deploy Docker images, such as to a DigitalOcean droplet or anywhere else. Our containers is deployed in our kubernetes cluster on Google Cloud.

I'm assuming you know or can learn the commands to deploy a Docker image and won't cover that part in this guide.

You then need to point an MX (mail exchanger) record from your domain to the external IP address of the server that hosts your server in order to route emails through to the server. We used a subdomain email address for this feature as our root domain already had email addresses set up for our team. We used the subdomain 'msg', and thus our replyTo email address ends in \`@msg.deedmob.com\` and our MX record name is \`msg\` to correctly route these emails.

[You can access all the demo code here](https://github.com/davidfurlong/email-reply-server-example)

**Problem:** How do we make sure the sender is who they say they are and that they are authorised to post into the message thread and which user they are in that thread?

There are two ways to handle this problem that I know of.

The first way to do this, is to encode information in the reply-to address before the @ symbol of the origin notification email. This information should convey the thread id, the user id of the notified person and a secret authentication token for the user. 

There are different configurations of this method which have different security properties:

(1) Use a stored secret authentication token for each (thread, user) pair. 

(2) Combine the unique identifiers of users and threads with a secret and then perform a one-way, collision resistant hash on the combined string.

(3) Use a stored secret authentication token for each (last_message, user) pair which makes it possible

Pros and cons of (1), (2) and (3):

(1) 
*Pros:* possible to revoke tokens
*Cons:* [Replay attacks](https://en.wikipedia.org/wiki/Replay_attack), need to store additional information

(2)
*Pros:* no need to store additional information
*Cons:* Depending on the hash function chosen and the defense against an attacker generating accounts, threads and brute force attempts it may be have properties which make it too possible to guess the secret or send unauthorized messages to other threads. Replay attacks are also possible. If your one secret is compromised the attacker can post as any user in any thread, and that if you change the secret to remove this access, all sent emails 'replyTo' header messages will be out of date and no longer be able to authenticate the corresponding users. Revoking tokens is not possible.

(3) 
*Pros:* Replay attacks are not possible, Revoking tokens is possible.
*Cons:* More information needs to be stored

In this post the code of option (2) is shown:

```js
const NONCE = Buffer.from('7'.repeat(tweetnacl.secretbox.nonceLength));
const emailReplySecret = Buffer.from(config.emailReplySecret);
export function generateReplyAddress(user: User, participant: ConversationParticipant) {
  const format = `C${participant.get('conversation_id')}U${user.get('id')}`;
  // Encode secret using hex (encoding must be the same)

  const secret = Buffer.from(
    tweetnacl.secretbox(Buffer.from(format, 'utf8'), NONCE, emailReplySecret)
  ).toString('hex');

  return `${secret}@msg.deedmob.com`;
}
```

In order to generate reply addresses we create a string with delimiters ('C', 'U' although the C is not strictly necessary) below  

and to decode

```js
export function decodeReplyAddress(address: string) {

  const [encryptedSecret, emailAddress] = address.split('@');

  const decodedSecret = tweetnacl.secretbox.open(

    // Decode secret using hex

    Buffer.from(encryptedSecret, 'hex'),

    NONCE,

    emailReplySecret

  );

  if (!decodedSecret) return false;

  const secret = Buffer.from(decodedSecret).toString('utf8');

  const [conversation_id, poster_user_id, organization_id] = (

    secret.match(REPLY_ADDRESS_FORMAT) || ([] as string[])

  ).slice(1); // match[0] is the full string

  return {

    conversation_id: Number(conversation_id),

    poster_user_id: Number(poster_user_id),

    ...(Number(organization_id) && { organization_id: Number(organization_id) }),

  };

}
```

The second way to build this feature is to use the \`To\` address to authenticate the user. This means we still need to add the thread id to \`replyTo\` to the email address in the email header, but we can do it in plaintext and don't need to keep it secret. The upside of this is that if the user accidentally forwards an email notification that the holder of the email can't successfully authenticate and impersonate that user. A downside is that if the user changes their email address in the app (if they are able to), then old emails will either fail to authenticate, or you need to continue to store all their old email addresses for the purposes of authentication. This option also allows for a shorter email address usually, as brute force attacks on the \`To\` email address are no longer possible. However they may be reasons why this doesn't work well when users have email forwarding rules set up, email aliases, or change their email address without changing it in app. In order to set it up this way we also need to set up additional authentication of emails that the sending email address actually did send that email, which we won't go in to here because I quite frankly don't want to go down that rabbit hole. Zendesk appears to do it this way, and perhaps thats is because their support software doesn't allow users to change the email address for a support query. 

![](/media/screenshot-2019-08-17-at-12.22.18.png)

Okay that was longer than I was expecting it to be. Thanks for staying with me. There's some issues you're likely to come across whilst building this feature, and the next section is dedicated to addressing these.

<a name="limitations"></a>
## Limitations & Considerations 🚧

**Limited formatting:** Users can only respond in the tools available within an email body field. In our code we assume to only care about plain text, and we strip out all html. But if your in-app interface allows attaching images perhaps you could also parse embedded or attached images within the email and send them through. People may expect to be able to do everything they can do in normal emails, but if your parser or app can't handle these attachments then we might end of losing some of the information intended to be conveyed by the user without a good way of communicating the constraints to them.

**Error handling:** If something goes wrong with the email reply, since we don't have control over the user's email client interface, we can't just show them an error message. The best we can do is send them an email in response that the message was unsuccessful, and why. Alternatively we could send them a confirmation email when it is successful, with the contents. However if it fails, will users really notice? Perhaps both is best.

**Spam:** Your mail server will undoubtedly receive lots of spam emails. We were getting tens of thousands a month. This may affect your server's capacity and resource usage, and its best to validate that the \`to\` address conforms to your structured format as soon as possible and definitely before sending a request through to your app's API.

**Security:** Mentioned previously were some security considerations regarding authentication of identity. Replay attacks are not attempted to be mitigated here, and we assume brute force attacks to be prevented by rate-limiting and DDOS protection of our API (Assumes our secret has sufficiently many possible values). There are ways to improve upon the security of this system and my goal herein is not to cover them nor to present a complete security assessment of the options. Your needs may require different levels of security. Using the \`replyTo\` header instead of the \`from\` address may have security implications if the encryption exists for one but not the other. _When I asked my friend he said most email encryption is fundamentally insecure_, and I'm no expert on the matter, so if you need proper security do your own research.

**Errors in stripping email contents:** I encountered some issues when stripping email contents of their signatures and histories, and sometimes these are not stripped away correctly. There may be a way to improve upon \`mailstrip\` above, but for our cases it only failed to strip out signatures rarely.

**Why don't we just use the 'From' for what is now the 'Reply-to' address**: The reply-to email address is hard to read (and often has more than 20 characters for security purposes - and up to 64 characters before the @) and is therefore sometimes perceived to be untrustworthy by users. It is easier for recipients to categorize and for email clients to increase trustworthiness assessments of individual email addresses than it is for all emails sent from a domain.

**Counter-productivity:** You may want people to come to your site to send the message despite it being more work for them, because you make money off ads and want to show them, or you want them to do other things on the site at the same time. In that case it may not be goal-aligned to provide this convenience feature to your users.

## A little bit about me and why I wrote this post 🤔

During the day I'm the CTO and cofounder at Deedmob, a startup in Amsterdam building software to make volunteering better. Whilst on paper I may be a CTO, I spend most of my time doing product management currently. We built this feature at Deedmob about a year ago to help our customers respond to messages faster. I couldn't find an online guide and with very little understanding of how email servers and protocols worked, I asked for help from an expert I found on github, who kindly helped me narrow my search. In the spirit of paying it forward I've written this post in the hopes that it helps someone else in a similar position.

_If this post helped you, you have questions or have any other interesting use cases of this feature let me know in the comments :)_

__

_// Writing time: 5.5 hours_
