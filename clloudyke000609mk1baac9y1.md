---
title: "How does Kerberos work - an introduction for beginner."
datePublished: Thu Aug 24 2023 07:28:23 GMT+0000 (Coordinated Universal Time)
cuid: clloudyke000609mk1baac9y1
slug: how-does-kerberos-work-an-introduction-for-beginner
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692862399990/29fe892c-fc07-41df-b686-c273d95c2a74.png
tags: windows, hacking, active-directory, cybersecurity, kerberos

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692860980254/c8d50f28-e5dc-46a2-bc2e-6fad22053861.png align="center")

## Introduction

Kerberos is an authentication protocol that superseded NTLM with the release of Windows 2000 (technically…)

Technically?!

Well… It's complicated 😅

Long story short - NTLM is still alive and kicking and… it might still be the fall-back solution in case Kerberos does not work. 😵

With that in mind let's talk about the moving parts in Kerberos and look at some wireshark pcaps.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861021297/2f80cd88-1ee9-41fc-b007-d3d29106cd4b.png align="center")

This is Kerberos Authentication in a nutshell 🥜

AS-REQ → AS-REP  
TGS-REQ → TGS-REP

Done.

well…

That's not really explaining it though, is it?!

Ok ok, you are right. Let's try to figure out what those words??? letters! mean..

## Kerberos Tickets 101

Kerberos comes from the Greek Cerberus - the three-headed guard dog of the underworld.

We don’t judge and love all puppies 🐶

This particular one has an affinity for tickets.

F\*CK balls and toys, it only likes tickets.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861086925/c8bfce47-a410-42c7-b04c-4a3ca9369325.png align="center")

When thinking about tickets, the first thing that comes to our heads is:

theme park!!! 🎉

In order to go into a theme park you need a ticket - the ACCESS ticket

But sometimes, you also need separate tickets to take rides inside the theme park - the ride tickets.

Usually, there is a booth at the entrance of the park where you pay 💰 and show your ID to prove you are of legal age to enter alone 🐣.

This booth is called the KDC.

## KDC - Key Distribution Center

This is the magical 🪄 place that hands out tickets.

The ticket booth consists of two small tents - one checks IDs + gives access tickets (left - AS)

\+ The other checks take access tickets + hand out ride tickets (right - TGS).

In the middle is the entrance to the theme park.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861122033/ee68c444-bdcf-4e6f-a2a1-2ea885660aeb.png align="center")

AND in front of it all, sits our little puppy wagging its tail.

Wait but why do we need a ticket from the booth if Kerberos had one under its paw?!

Can’t we just steal it?!  
And why does it have a different color (golden)?!

Well… I don't know if you are a dog whisperer but for me, this is a big NO NO

I would much rather trade something to get the ticket from mr puppy without hurting myself.

But what does Kerberos like?!

TICKETS!!!

It’s like the cookie monster but instead of cookies 🍪 Kerberos has an unhealthy obsession with tickets 🎫

No steaks, no squeakers, just tickets.

...to get the ticket we need another ticket…

Pheww…

## Ticket Granting Tickets

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861163525/aae08de0-5353-4fcd-9f04-0567f629e75c.png align="center")

Concerning the colors - We have seen a few different tickets already:

Ticket Granting Ticket (TGT) - access ticket (red)  
Ticket Granting Service Ticket - single ride ticket (silver)  
Golden Ticket - VIP pass for all rides (golden)

Now how do we combine all of this into the beautiful mess that we saw earlier?  
Remember our Wireshark flow?

AS-REQ → AS-REP  
TGS-REQ → TGS-REP

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861190918/4914e1e0-e89c-4175-8bcc-3ad7013c3111.png align="center")

ok slowly:

AS - Authentication Service - This was the left tent in the KDC picture  
REQ - Request  
REP - Response

OHHHHH, that makes sense 🤯  
But WAIT what about the KRB Error?!

YES, good catch 🫴🏉

Look at the Length of the AS-REQ(uest) - it is 300 Bytes and we receive a KRB5KD\_ERR\_PREAUTH\_REQUIRED

This means that the account that we try to authenticate with does not have the “Does not require preauthentication” flag set.

ok sure… WHAT?!!?

There is a (non-default) setting called “Does not require preauthentication” and if that one is set, this account does NOT require preauthentication.

In essence, this means you can request an access ticket (TGT),

WITHOUT the credentials of the user you are requesting it for.

EXCUSE ME?!

## AS-REP Roasting 101

Yeah… there is a technique called AS-REP (GET IT?!?!?!) roasting, which requests an access ticket for users without Kerberos preauth.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Keep in mind - while a TGT gets issued by the DC during AS-REP Roasting, this is not a working TGT that can be used to request ride tickets (TGS) - Thx <a target="_blank" rel="noopener noreferrer nofollow" href="https://twitter.com/exploitph" style="pointer-events: none">@exploitph</a> &amp; @<a target="_blank" rel="noopener noreferrer nofollow" href="https://twitter.com/filip_dragovic" style="pointer-events: none">filip_dragovic</a></div>
</div>

Back to the Ticket - Why do we want to request a ticket that we know will not be correctly working?

Part of the response is encrypted with the password of the user…

OH DANG…

Yes… They can then be used for offline cracking/brute force guessing of the user password. 😈😅

Hacker use a tool called `impacket-getNPUsers.py` to get a crackable hash like the one below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692878635372/cca2a983-2c79-4ab4-8bf6-6298496f00a5.png align="center")

Long story short - there is a reason why this “option” is not default!

Be careful BEFORE turning this on.

Back to the Error - We send the ID (credentials) of a user to get a personalized access ticket

→ There is a difference of 80 bytes (300 → 380) between the 1st & the 2nd AS-REQ

If all goes well, the KDC responds w/ an AS-REP (Authentication Service Response) with our access ticket 🎟️

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861268460/1b196d5f-81f0-4276-a942-eafcbc8af07c.png align="center")

Can we see that?!

Sure.

## Ticket Granting Service Tickets

Looking at the AS-REP in Wireshark shows us that there is a section called “Kerberos”

This one holds the AS-REP section → a part called “ticket”

Another one called cname - the account that requested the ticket (maikroservice), and crealm aka domain (snackempire.home)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861281906/cde1d9a6-589c-44b8-bfc5-e97105a49eaf.png align="center")

The ticket is encrypted and we can see it at the bottom highlighted in blue 💙🙆‍♂️

Wonderful, now we have a Ticket Granting Ticket or as I like to call it “access ticket” 🎟️

For all the defenders and curious people out there:

The request for an access ticket generates an event with ID 4768 on the domain controller.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861295965/ab733c59-8ae8-4646-9157-6e5f1abcc40a.png align="center")

Our access ticket gives us access 🤓 to the 2nd tent - the Ticket Granting Service (TGS) 🏕️🔑🎪 → ⛺️🎫

We now present said access ticket to the clerk in the 2nd ticket tent and they will ask one question:

Which ride do you want a ticket for?!

Ooofff….

Which rides are there?!

Great question!  
I have prepared an overview.

These are often called silver- or service tickets but we can think of them as “rollercoaster tickets” or whichever ride you prefer.

Concerning some of the possible services we can ask them for - 👀 ⬇️

Ooofff... That's a lot.

Ok, slowly!

## Ticket Granting Service Tickets Types (Silver Tickets)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861328018/9f3d1a7e-772e-4df4-8e88-9fb8413650c3.png align="center")

`HOST` - winrm/PowerShell remoting → remote access

`CIFS` - file server/psexec → remote access if the share is writeable

`HTTP` - winrm/PowerShell remoting → remote access

`LDAP` - remote server administration → remote control

`KRBTGT` - GOLDEN TICKET 🎫 → VIP ALL ACCESS PASS 😎🎉

Ok but what's the process like?!

In Wireshark - in the AS-REP our access ticket is `2da8de0621…`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861281906/cde1d9a6-589c-44b8-bfc5-e97105a49eaf.png align="center")

When we now request a ride ticket we send our access ticket in the TGS-REQuest (see below), this is important because it VERIFIES that we are a real user in the domain.

You can see that in our TGS-REQ we send our ticket (cipher) `2da8de0621…` which allows the Key Distribution Center to verify that we are who we say we are.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861366066/5991ccd5-0370-45c7-9466-eb14f4184ef6.png align="center")

Conceptually, we send our access ticket (TGT) with the name of the requested ride (service) and receive a `personalized` service/silver/ride ticket back.

This triggers a Windows Security Event #4769 which might be helpful when hunting for hackers 😉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692861375244/8d60fa85-f222-42ca-9248-9c8828f9e882.png align="center")

With the ride ticket in hand, we can now access the theme park and specific rides 🎢 or request a new ride ticket with our access ticket 🎟️

In the next post we walk through RDP and what happens on a packet level.

Stay tuned.