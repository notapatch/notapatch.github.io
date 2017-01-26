---
layout: post
title: "Economy class domain email using Mailgun"
categories: economy DNS
---

Domain emails use the domain name after the @ symbol. Using one can make you appear every bit as professional as a header of 'This site uses cookies'. In my case I need the professional sounding domain email of patch@notapatch.io. 

There are plenty of ways to get a [paid domain email](https://www.google.co.uk/search?q=domain+email&ie=UTF-8) but if you are a small blog getting a couple of emails that can be an overkill. You can keep your personal email address and route your domain emails to and from it using a third party. Here Mailgun. 

#### Requirements
Avoid getting lost halfway through by having:
1. Understanding on what DNS records are - I recommend first 5 chapters of [DNS and BIND](http://shop.oreilly.com/product/9780596100575.do)  
2. Access to edit your domain DNS - [Domain name registrars](https://en.wikipedia.org/wiki/Domain_name_registrar) are a good place to locate your DNS records.  

Configuring DNS shouldn't be hard - for a blog the domain is only a few lines of code but I have found it takes over two readings and practical work before you can get comfortable with the terms. Beginners will have problems as most explanations expect you to understand DNS terms.

## Steps

1. Add your domain
2. Now Follow These Steps To Verify Your Domain
3. Domain emails to your personal email account
4. Domain emails from your personal email account
<br>
<br>

### 1. Add your domain

Domain emails are sent from any email server but they can be received at only one server. *If you set up an email server to receive emails on your domain, Mailgun cannot accept emails on that domain.* [See Mailgun FAQ](https://documentation.mailgun.com/faqs.html#how-do-i-pick-a-domain-name-for-my-mailgun-account).

#### How can I use Mailgun on my Domain?

| Any Domain Email?               | Send   | Receive |
|:-------------------------------:|:------:|:-------:|
| I do not have email set up      | ✔      | ✔       |
| I already email from the domain | ✔      | ✘       |
{:.table .u-margin-bottom-large}

If you are unsure, you need to check your DNS for the MX records. In the following example, we are using Yahoo mail and cannot now use Mailgun to receive emails on this domain.

    ; DNS Records
    ... 
    ...
    @ 10800 IN MX 1 MX1.BIZ.MAIL.YAHOO.COM
    @ 10800 IN MX 5 MX5.BIZ.MAIL.YAHOO.COM 

Sub-domains are required if you use more than one email service. I will not be covering sub-domains as I assume I am talking to a business user who accidentally [turned right](http://forum.wordreference.com/threads/few-of-us-can-afford-to-turn-left-on-an-aircraft.2330293/). 

Now we understand the choice add our domain name and ignore the recommendation to use a sub-domain:

Domain Name: <b>notapatch.io</b>

{% include helpers/image.html name="add-your-domain.png" caption="Here be dragons" %}

### 2. Now Follow These Steps To Verify Your Domain

This is the most difficult part to describe as how you configure DNS will depend on your DNS provider - they should look like this.

#### Example Mailgun related Resource Records

| Mailgun Ref     | Name          | Record Type  | Value                              |
|-----------------|---------------|---------------------------------------------------|
| SPF             | @             | TXT          | "v-=spf1 include:mailgun.org -all" |
| DKIM            | k1<wbr>._domainkey | TXT          | "k=rsa; p=MM0DQ..."                |
| MX Records      | @             | MX           | mxa.mailgun.org.                   |
| MX Records      | @             | MX           | mxb.mailgun.org.                   |
| Tracking opens  | email         | CNAME        | mailgun.org.                       |
{:.table .u-margin-bottom-large}

After time has passed, say 20 minutes, check the Mailgun domain by clicking 'Check DNS Records Now' and 95% of the time you will see a big orange unverified message and you will have to try again. To speed up the verification process read - [Has mailgun seen the changed DNS?](#has-mailgun-seen-the-changed-dns).
{% include helpers/image.html name="unverified.png" caption="Get used to seeing this" %}


#### Common Mistakes
1. Dots - values end with a dot (referring to root server) and names do not
2. Quotes - SPF and DKIM values are in quotes.
3. DKIM's name varies [something]._domainkey

#### <a name="has-mailgun-seen-the-changed-dns">Has Mailgun seen the changed DNS?</a>

Mailgun takes up to 48 hours to see DNS updates propagated. While I've found MX records are easy to get working - as you cut and paste them. If I have a problem, it will be with the site specific records SPK, DKIM and CNAME.

To avoid the wait, each time I change the DNS I alter the MX resource records - sometimes MAX sometimes MXB - I know propagation has occurred when the MX records have changed.

##### 1. Get MX records working

    @ 10800 IN MX 10 mxa.mailgun.org.
    @ 10800 IN MX 10 mxb.mailgun.org.
    ...
    More resource records
    ...


{% include helpers/image.html name="mx-green.png" caption="mxa and mxb" %}


##### 2. Change the MX record

Change the awkward records (SPK, DKIM and CNAME) and then change the MX records before saving the new DNS - the update has propagated when mxb has a green tick replaced with an orange warning icon.

    @ 10800 IN MX 10 mxa.mailgun.org.
    ...
    More resource records
    ...

{% include helpers/image.html name="mxb-orange.png" caption="DNS propagated: mxb has changed" %}


##### 3. Keeping changing the MX records around

The next time you use a different set of MX records than the last. Say only mxb this time.

    @ 10800 IN MX 10 mxb.mailgun.org.
    ...
    More resource records
    ...


With good fortune you will, eventually, see you have verified the domain with Mailgun.

{% include helpers/image.html name="verified.png" caption="And finally" %}


### 3. Domain emails to your personal email account

With the domain validated the next step is to route domain emails to your personal email account.

| Expression Type   | Match Recipient           |
| Recipient         | .*@notapatch.io           |
| Actions           | ✔ forward                 |
| Specified Actions | me@my-personal-email.com  |
| Description       | notapatch => me           |
{:.table .u-margin-bottom-large}

{% include helpers/image.html name="create-route.png" caption="Creating a route matching all domain emails" %}

When sending test emails *do not* use your personal email account to send a test email. The email system may shortcut emails sent from themselves.  



### 4. Domain emails from your personal email account

To email from your personal account, first add an SMTP login, and then setup your personal account to use it.

#### Adding an SMTP Login
{% include helpers/image.html name="smtp-credentials-for-notapatch.png" caption="SMTP authentication" %}

Notice SMTP Settings - now:
* Server smtp.mailgun.org
* Ports 25, 587 and 465
* Use full email address - in my case patch@notapatch.io

For the personal account I will demo using Gmail.

In Gmail [select gear => settings => accounts](https://mail.google.com/mail/u/0/#settings/accounts) - then select Add another email address


{% include helpers/image.html name="add-another-email-you-own.png" caption="New SMTP account being activated" %}


{% include helpers/image.html name="send-email-through-your-smtp-server.png" caption="Final step" %}

After completing registering the new email account and refreshing the Gmail tab - I have the choice of sending from patch@notapatch.io. When sending a message from patch@notapatch.io Mailgun's notapatch.io domain increases outgoing emails. 

{% include helpers/image.html name="outgoing.png" caption="Outgoing email greater than 0 ... and relax" %}

### Summary

Mailgun makes it possible to send and receive a domain email with no running cost - a welcome email solution for the small blog owner.
