---
title: "How to get started with Monitoring on the Blue Team: A Step-By-Step Guide"
datePublished: Tue Aug 29 2023 23:10:24 GMT+0000 (Coordinated Universal Time)
cuid: cllwx8o0w000609le3lhwcdus
slug: how-to-get-started-with-monitoring-on-the-blue-team-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1693209605064/102e7beb-2ef2-4889-b023-59e1c0ed9dbc.png
tags: hacking, soc, cybersecurity-1, blueteam, siem

---

First up - What is the Blue Team anyway? Good Question - the Blue Team is typically not just a single team.

It is a combination of multiple security disciplines combined for ease of speech. Similar to how `Red Team` sometimes means pentester as well. Wrong, but socially accepted. But you might still have the question - Which Blue Team disciplines exist?

## Blue Team Disciplines

Blue Team tasks typically consist of the following disciplines:

1. **Monitoring** 👀
    
2. Analytics 🧮
    
3. Hardening 🪨
    
4. Incident Response 👩‍🚒
    
5. Threat Hunting ⚡
    

In this post, we will cover the Monitoring Discipline one by one with example tasks and things you need to know about it. So without further ado here is the overview of Monitoring:

## Summary

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693214217107/b2667ac8-af89-41f8-9560-5b93b566777f.png align="center")

Monitoring is a combination of Logging, Detection, Visualization and Inventory Management:

* SOC / SIEM
    
    * Alerting
        
    * Event Triage
        
    * Dashboards
        
* Inventory (Machines, Cloud Assets, People, Services)
    
* Networking
    
    * Internal Firewalls + Web Application Firewalls (WAFs)
        
    * Network Access Control (NAC)
        
    * Virtual Private Networks (VPN)
        
    * DNS Sinkholes
        
* Packet & Protocol Tracking (Network Monitoring)
    

That is A LOT of words you might not have heard before - but fear not, I will share all the necessary details.

A job that is often associated with monitoring is called SOC Analyst - or Security Operations Center Analyst.

### In simple Terms - Monitoring explained for aspiring Blue Teamers

Monitoring means collecting Logs and looking at graphical representation of them.

Uh... what?  
Why do computer nerds collect parts of trees? 🪵

...and why ON EARTH do they look at paintings of them? 🎨🖌️👩‍🎨

Well... almost

Logs are Log files or even streams.

Aha. Why would I need them?

Imagine you have a web application and someone tries to hack it.

How do you know?

Better: How would you know?

Well... I would know if I get hacked... RIGHT!? Kinda.. but also - no.

That is way too late.

Remember the 5 stages of ethical hacking?

1. Reconnaissance
    
2. Enumeration & Scanning
    
3. Initial Exploitation - Gaining Access
    
4. Lateral Movement & Privilege Escalation - Maintaining Access
    
5. Cleanup
    

Imagine you get hacked, if we take our previous assumption that we would only find out once the attacker has access - that is after Step 3.

What if we wanted to catch them at Step 1 or 2?!

HELLOOOO LOGS. 🪵🪓

Now how do you find logs, if you cannot go to the hardware store or the forest to collect them?!

And what do they look like? Those are the right questions!

AND we will answer them today.

## SOC - Security Operations Center

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693214087920/33dc9739-2add-4bb2-b1f8-3d5dc272a5f2.png align="center")

The Security Operations Center is a team of individuals who watch logs to identify threats and compromises. This team typically operates in 24h / 7 days a week mode for 365 days of the year. Each SOC analyst works in shifts, similar to nurses in hospitals.

It is hard to find good SOC personnel, so companies often outsource the SOC to MSSPs - Managed Security Service Providers. These are companies that specialize in SOC and sometimes even Incident Response.

The tool every SOC uses is called SIEM - Security Information and Event Management System.

### SIEM - Security Information and Event Management System

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693215314397/25be8d57-709e-4826-bbc4-9809b5d9d916.png align="center")

A SIEM is basically a software tool that allows you to collect and analyze log files.

Log files are text files that contain logging information - these can be operational data for computers (e.g. user logins & created files) or services (Web Application, Authentication Service, Domain Name Service - DNS).

These log files are collected, normalized (transformed) and used to build dashboards. Dashboards consist of graphs/charts and alerts.

### Alerting

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693311917870/64e98abb-e70e-44ee-a65a-4c225070624e.png align="center")

Alerts are based on detection rules, which specify actions or filenames that are considered malicious. The process of checking the local files is called File Integrity monitoring - you usually need to mention which folders / disks should be checked and the more folders, the more data is stored.

This is also an issue because a lot of data means SOC analysts can be hit by alert fatigue. This "disease" happens when you see so many (false positive) alerts and become desensitized to future ones - a dangerous situation because the next one could be real.

Less is more, focus on high-impact alerts first and foremost.

For those of you wondering - *OK great, but how can I get started with Security Information and Event Management Systems?*

Check out this post: [https://maikroservice.com/how-to-setup-wazuh-as-your-siem-with-debian-proxmox](https://maikroservice.com/how-to-setup-wazuh-as-your-siem-with-debian-proxmox).

It will guide you through the setup and requirements of a SIEM for your homelab, which gives you invaluable experience for the job hunt. I cannot stress this enough - if you do not have job experience, build it yourself with a homelab and simulated attack scenarios.

After the alert a SOC analyst needs to decide how bad the situation is and what to do next - this process is called Triage.

### Triage

Triage is at least the 2nd most important task of a (future) SOC analyst.

As a Junior SOC Analyst, you would watch the dashboards and alerts and if something happens you gather information and create a ticket.

You have a tiered model - level 1 is junior analysts, level 2 is intermediate analysts and level 3 is team lead / seasoned professionals - The ticket flow is similar to a pyramid. At the bottom, the level 1 analysts look at the majority of alerts and forward the ones they deem interesting.

For each (in an ideal world) alert, a ticket is created and valuable information is added.

These tickets are forwarded to the next higher level and then categorized (triaged), level 1 analysts can also triage the tickets already, depending on the setup.

*Ok, but how do analysts know which data point is important?*

They use dashboards to visualize the events.

### Dashboards

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693311810547/f31fc36d-29d9-4790-94e1-846788f0d915.png align="center")

Dashboarding (I hope this is a word) is the process of visualizing data using graphs and tables. Typically, this is done with the SIEM software but outside of security dashboards are also used for data analytics and reporting (e.g. sales, or marketing data).

These dashboards can be used to have a quick look at the status quo or dive down into data points, e.g. analyze events on a specific machine after an alert was raised.

What you need to understand the alerts properly is a deep understanding of how logs work - up next.

### Logs in Detail

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693229798080/14a42823-a13b-4205-90cc-d73e68ed64f0.png align="center")

Logs are flat files - sometimes `JSON`, sometimes `CSV`, sometimes `TXT` files.

*Sometimes what now?*

`JSON - JavaScript Object Notation`

`CSV - Comma Separated Values`

`TXT - Text`

These are some of the different formats that Log files can come in.

*Can you give me an example of what those look like?*

Sure - glad you asked:

```bash
# JSON (pretty)
{
  "UUID": [
  {
    timestamp: "2022-11-02T20:57:38+00:00", 
    user: "admin", 
    action: "login", 
    page: "admin.php"
  }
  ]
}
```

```bash
# CSV
timestamp,user,action,page
"2022-11-02T20:57:38+00:00","admin","login","admin.php"
```

```bash
# and TXT
timestamp,user,action,page
"2022-11-02T20:57:38+00:00", "admin","login","admin.php"
```

*Wait CSV and TXT look exactly alike?!*

Yes, that could very well be - sometimes the delimiter (the character between the individual entries) changes from `,` to `;`

but `TXT` is often "just" `CSV` with a different file extension.

*Now what/who generates these things and who defines the format?*

Log files are produced by almost all modern applications and operating systems.

Which is kind of an answer to - *How do we find logs?*

Applications produce them - *NICE!*

But let's dive deeper - so these are application logs - and a full-fledged web application produces different logs compared to for example Microsoft Word or Excel.

*Why?*

Word & Excel are single-user local applications (they can also be used in browsers now, I know.)

While Web Apps are distributed.

That means that many users use the same application, plus the servers that host the application are typically spread around the world 🗺️

*What do you think would be different between the logging of these two types of applications?!*

Let's make it even more complicated.  
What if your Operating System also produces Log Files.

*What do you think is in there?*  
*Would that look different for Windows, Linux and MacOS?*  
*Where do I even find these logs?*  
*Are they on by default?*  
*Do people spy on me w/ logs?*

Let us slowly dive into the different types of logs -

`Application Logs` are different from `System Logs`.

There might be more types but those two are enough for now.

`Application Logs` show user interactions with the app  
\+ they show which service talked to which IP/other service.

*Why?!*

> User Input is the most dangerous thing on the internet

Worse than Sharks, Snakes, Spam Emails and Phishing combined.

It is how companies get hacked, how everyone gets hacked.

So we save it to logs.

*We want to know if a user tries to hack us, right?!*

NOT SO FAST!

Welcome `GDPR` and other regulations.

You cannot just log everything a user does, you need their consent - and even with that, you shall only save `necessary` and `minimal amounts of data`.

so you don't log everything... ok.

*Do you log anonymized interactions?!*

That is better. But now you don't know *WHO* tries to hack you.

And for now, that is not that important, for Monitoring + Security you care about the User Input first + maybe the IP

Two things to think about -  
*What happens to your account if someone hacks your password?*

*What happens if a user uses a VPN?*

**Think about those and leave a comment below with your answers.**

Back to Logging: As Blue Teamers, we want to know which input breaks our application.

So that we can (*tell the developers to*) fix it!

`Application Logs = User Input + Services talking`

*What about System Logs?*

Your Operating System creates logs on your interactions as well.

*Why?*

Product Improvements, Troubleshooting, Support of IT staff at your company and many other reasons.

*Got it, seems legitimate.*

*What is in there though?*

To answer this question - let us first find the logs!  
The hunt begins.

`Linux & MacOS` store System logs in `/var/log`

While Windows 1. mostly calls them Events and 2. stores them at: `C:\Windows\system32\winevt\Logs` by default (`Windows 7/8/10/11`).

*How can I read them?*

MacOS -&gt; `console.app`  
Linux -&gt; text reader (e.g. `less`)  
Windows -&gt; Windows Event Viewer (`eventvwr.exe`)

Nice and last question - *How can I make sure that all of them are aggregated in one place for System Monitoring in a Business Network / Home Network?!*

That is exactly what a SIEM is for, but you need to make sure to know how many devices belong to your company.

How? Inventory Management!

### Inventory (Machines, Cloud Assets, People, Services)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693223285606/d772e30d-07d4-4fdb-a56e-a513cfff9a9e.png align="center")

Knowing how many total laptops, servers, mobile phones and cloud storage buckets you have in your company is ... hard, very hard actually. Because employees are quickly onboarded / offboarded, can also autonomously buy hardware and use cloud accounts this often leads to confusion.

The larger the organization the worse this gets. BUT it is essential to keep track of the hard and software in your company.

Only when you know it exists can you monitor the device/service. No monitoring, no visibility means this device can be hacked and you would not know about it.

![](https://pbs.twimg.com/media/FglzkveXwAA3BEs.jpg align="center")

To make matters even more interesting, in 2023 infrastructure is usually distributed across multiple data centers with content delivery networks and load balancers in between. If those words do not mean anything to you now, that is totally fine.

The idea behind my mentioning all of this was to show the complexity of a "simple" website in 2023. It can quickly get out of hand when engineering takes the reigns and moves forward.

Keeping inventory is at the interface of analytics, whenever you see a new IP/device in your logs you might want to check where it comes from.

This leads us to the next topic - networking.

### Networking

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693225541349/4497289e-0bf5-44cb-a57e-727510206f6e.png align="center")

Each device in your business environment will most likely connect to other devices in the same department.

When you are a remote/global company this department might be filled with people from South America to Japanese employees all working on the same projects.

That is a lot of ground to cover and you need both the internet and internal networks to enable collaboration.

These internal networks need to be protected and one of the classics is to use a firewall.

### Firewalls

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693345574599/47f36f4b-67f6-4cb7-bdf0-0c67a245aafb.png align="center")

Speaking about firewalls means that we have a network or service that needs protection. These networks can be internal and hold confidential documents or they can be part of our Web Application infrastructure for example. The latter also needs protection but needs a different kind of firewall - a Web Application Firewall (WAF).

*Why can you not use one for the other?!*

Well, one might be a hardware firewall and the other one might be software, there might be different threats associated with each environment etc. - but you could. You could probably figure out how to use one for the other when money is tight or hidden costs (time investment for figuring out which vendor to go with) are deemed too high.

Generally, firewalls protect you against unwanted access e.g. by blocking traffic from specific ports.

*Okay, but what is a port?*

Good question, which is interlinked with the next chapter but I wanted to show you that there will be a lot of concepts that you might or might not understand immediately.

What is important though is that you focus on learning concepts rather than specific technologies/hardware/tools. In the end, the technology will most likely change over the course of your career but the concepts will stay the same.

Oh, and a port is a specific software address that applications/machines can connect to on a computer. It's similar to a loading dock in a factory or stop in a bus terminal, you need to be in the correct place to be picked up and take the bus / get the delivery.

How would you allow/deny access to the networks? Besides a Firewall you can use Network Access Control (NAC).

### Network Access Control (NAC)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693345526105/8ee43d8b-a960-4686-bf11-c41eb97911c7.png align="center")

Network Access Control measures allow / deny people to connect to your network of choice based on certain conditions.

*What are those?!*

You can decide, but some examples include:

* Tiered networks (e.g. separate guest and employee networks)
    
* Bring your own device policies (someone wants to use their private phone for work, which means they have e.g. to enroll in device management, keep the phone operating system up to date and can only install certain apps)
    
* Compliance policies (e.g. anyone who wants to connect to your network via cable or wifi needs to have a registered device which is actively managed by the device management)
    

Another access control measurement would be to have a virtual private network (VPN) for your employees.

### Virtual Private Networks (VPNs)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693345594120/99eb29f1-d1c8-416f-b83e-7e7bf71639cf.png align="center")

VPNs as their name suggests are virtual networks that need some kind of authentication or configuration file to get into. There are two types of VPNs - mesh-style networks and server-based networks.

Mesh-style networks can be represented by wireguard, while server-based networks can be initiated using OpenVPN.

For a more in-depth walkthrough check out this Twitter thread: [https://twitter.com/maikroservice/status/1673811963070017538](https://twitter.com/maikroservice/status/1673811963070017538)

If you want to be extra sure that no malicious traffic enters/exits your company networks you can also use a DNS sinkhole.

### DNS Sinkholes

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693345682298/bbda2b6e-a95f-40da-9bb8-5418f327d132.png align="center")

DNS Sinkholes are basically blackholes for domain name server (DNS) traffic. DNS translates human-readable domain names (e.g. `maikroservice.com`) to IP addresses (`76.76.21.21`).  
If one of the devices in your network tries to access a known malicious url, your DNS server would respond with a "wrong" IP that does not lead to the hacker domain.

This can be combined with an allowlist - which defines the allowed domains anyone in the company can access.

The combination of a DNS sinkhole and an allowlist provides a layered approach to security. Malicious or unauthorized domains are diverted away from their intended destinations, while trusted domains on the allowlist are granted unblocked access.

If you cannot use an allowlist you need to check the network traffic closely - also called network monitoring.

### Packet & Protocol Tracking (Network Monitoring)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693345855214/61ecab8e-1270-4c9c-8739-767e1efd3ded.png align="center")

When you join a new company as a consultant the first thing you tackle is to get a lay of the land - you try to figure out how bad the current situation is and which future action will have the highest impact.

You would also try to establish a baseline - what exactly is considered normal in this environment, what would an attack on a high-value target look like and can we detect it easily?

The toolset you would bring to this section consists of Wireshark (packet analyzing software), an Intrusion Detection and Prevention System (IDS/IPS) and probably DNS/Proxy Server logs combined with Linux command line tools (e.g. grep, awk, sed and others).  
If you want to learn more about those command line tools, check out this video: [https://www.youtube.com/watch?v=zNa6G7KOGXc](https://www.youtube.com/watch?v=zNa6G7KOGXc).

Sometimes the logs you are looking for are not in the correct format and you need support, there might be help available.

### Logging as a Service

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1693345944539/9145186e-b174-4d27-91a3-57ab29fe96a6.png align="center")

If you are in a situation where your logs look like a pile of lego stacked on top of each other - you might need help.

Some organizations might be unaware of the amount of Log Aggregation / Transformation necessary to make sense of logs and the underlying traffic/behavior patterns.

You can work with external partners to identify potential high-value log sources, transform and combine different ones and build dashboards/alerts on top of the logs in your SIEM.