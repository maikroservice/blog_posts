---
title: "Step-by-Step Guide to Setting Up Snort as Your HomeLab IDS with wazuh (SIEM) Integration in 2023"
datePublished: Mon Oct 16 2023 20:50:26 GMT+0000 (Coordinated Universal Time)
cuid: clntddkgr000109l747ak1bsu
slug: step-by-step-guide-to-setting-up-snort-as-your-homelab-ids-with-wazuh-siem-integration-in-2023
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697489421689/33197e07-f5a9-4014-865b-0baba52f8f89.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1697489406887/2ed83a9c-09b2-40af-ac4e-89f582811bf6.png
tags: hacking, cybersecurity-1, homelab

---

IDS/IPS systems - Intrusion Detection / Prevention Systems - are part of any well-established organizational network. If security is a priority you need an IDS/IPS sooner rather than later.

Commonly used open source solutions for IDS are snort and suricata, today we will look at snort and setting it up from scratch in your home lab.

Cool, but how does and IDS work actually?

## Overview - What is an IDS/IPS?

The Intrusion Detection System analyzes all traffic either via port mirroring - essentially, cloning all packets and analyzing them - or acting like a gateway, which is similar to airport security, everyone needs to pass before they go in/out.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697454786266/4d83b112-3c02-4f8b-8e6c-d5f064818572.png align="center")

The software then compares the data inside the packets to predefined rules - these can range from regular expressions to more sophisticated integrations - essentially they all do the same though.

They compare data to known malicious indicators.

## The plan

Today we will do the following things:

1. install snort on debian virtual machine
    
2. configure snort
    
3. install wazuh agent
    
4. forward snort logs to wazuh
    
5. *optional* - download and install rules from emerging threats
    

Are you ready? Ok, without further ado - here we go.

## 1 - Install snort on debian

I assume in this post that you have a virtual machine with debian on it, should you want support in how to set this up check out this post:

[https://maikroservice.com/setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide](https://maikroservice.com/setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide)

Start the Virtual Machine - Once the VM is booted the installation process for snort is a single command inside your terminal:

```bash
sudo apt-get install snort -y
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697455838361/d88b207d-2892-4221-a027-7f4678c9fd5b.png align="center")

Once the prerequesites are installed snort will ask you to configure the starting setup properly - the first thing you have to do is enter your local network that needs scanning.

You need to use the CIDR notation, if you are unsure what that is ask your favorite search engine or GPT buddy for support with `subnetting` - or leave me a comment and I will try to answer as many as possible!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697455945812/0a52ce18-fbd2-4f92-8d1f-1f2e325c8294.png align="center")

Once that is done make sure to check if snort was installed properly by running `snort -v` - if you receive `bash: snort: command not found` make sure you are in a privileged context (e.g. `sudo snort -v` or `su - && snort -v`).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697456423913/2accd354-1d44-40cb-842e-85323edc6ff4.png align="center")

Great, that was the installation process of snort - you did very well 🤘.

Now we need some configuration to make sure snort works properly.

## 2 - snort configuration

By default snort (2.9.xx) has the following interesting locations - `/var/log/snort` and `/etc/snort`.

The former holds all the logs that snort produces, the second one holds the configuration file - `snort.conf` we are interested in.

Open the `/etc/snort/snort.conf` file with your favorite text editor e.g. `nano /etc/snort/snort.conf`.

Now scroll down to `Step #6` and find the lines below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697457187171/886eb78f-b4ab-4375-af59-7e020a57fe05.png align="center")

You need to change the following line(s) and save after:

```bash
output alert_fast: snort.alert.fast

# change it to
output alert_fast: snort.alert

# optional but suggested, remove the # before 
# the #output alert_syslog: LOT_AUTH LOG_ALERT line
```

Great, we adapted the configuration file, but umm...

*Why?*  
*What does that change do?*

Great questions - The line `alert_syslog: LOG_AUTH LOG_ALERT` tells snort to log stuff related to `authentication` and `alerts` via syslog.

*Cool but umm... What's syslog?*

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Syslog is a standard for system logging which you can find under <code>/var/log/syslog</code> on ubuntu/debian derivates and in <code>/var/log/messages</code> for CentOS derivates like amazon linux.</div>
</div>

*Why is this important?*

Your Security Information and Event Management System (SIEM) has integrations for syslog so events that are inside the syslog will by default be connected to your SIEM already - win &lt;&gt; win situation!

*Ok cool, but what about the other change we made to the configuration file?*

That one took me a while to find/figure out - By default snort stores log data in pcap format - this is a binary format that e.g. Wireshark uses to store network traffic information.

Great for the computer, not so great for our SIEM - because it cannot read the format 😅

What we did with `output alert_fast: snort.alert` is the following:

We use the `alert_fast` module which stores data in readable text (good for us + SIEM) and tell it to use the `snort.alert` data stream (all the alert data) for logging - documentation (2.6.2): [http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node21.html#SECTION00363000000000000000](http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node21.html#SECTION00363000000000000000)

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">There is also another option called <code>alert_full</code> which logs everything inside the packet. This option has a large issue though, it is super slow. That means if you have any network with more than 1-2 idle machines you will be in for a rough one, because the conversion is so sloooooooooow. <code>DONT USE THIS IN PRODUCTION!</code></div>
</div>

Great, now our snort install is almost ready to throw alerts. The penultimate step is to restart snort.

```bash
sudo systemctl restart snort
```

...and then...

You should be able to see one or two new files inside `/var/log/snort` called `snort.alert.fast` and maybe even `snort.alert.fast.1` - this is good, it is exactly what we want!

The only thing left is to create a simple test rule. Let's do that next.

### Rules and Tests

We will now create a rule that watches for ICMP (ping) traffic and alerts when it finds any. Disclaimer - Since this is a fairly common interaction inside the network you will see a lot of alerts - do not use this in production unless you know what you are doing 🤘

Rules are stored under `/etc/snort/rules` - the one we are looking for is called `local.rules`.

We need to add the following line:

```bash
alert icmp any any -> any any (msg:"ICMP connection attempt:"; sid:1000010: rev:1;)
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697458652770/49143a23-191c-41ef-805e-4581df0c57a9.png align="center")

This means back to front - we are in `revision:1` , the unique id of our rule is `1000010` and the message we will put into the log is `ICMP connection attempt:` followed by the actual information from the packet.

Save the changes and then we can test the rule.

#### Testing local rules

Lucky for us snort has an integrated feature that allows us to test newly created/changed rules. We can use:

```bash
snort -q -A console -c /etc/snort/rules/local.rules
```

to test the rule we just created - this will run snort and wait for traffic that matches the rule. If you don't see any after a minute or two you can create it yourself.

```bash
ping -c 20 <IP_OF_SNORT_VM>
```

This sends 20 ICMP requests to the snort VM and will trigger the rule 20x 😈 - results might look similar to (without the typo hopefully 😅):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697459204955/054e94c9-9039-4c39-9012-57dca402f938.png align="center")

Cool - we have a working rule and a running IDS - now we install and connect our *Security Information and Event Management System (SIEM)*.

## wazuh linux agent installation

In order to install the wazuh agent you can follow the exact steps here: [https://maikroservice.com/setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide#heading-install-linux-agent](https://maikroservice.com/setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide#heading-install-linux-agent)

Afterwards, your agent should show up in the wazuh dashboard but it will not have any IDS alerts (yet). Because, this is what we cover in the next step!

### snort &lt;&gt; wazuh connection

In order to see the alerts in wazuh we need to modify the `/var/ossec/etc/ossec.conf` file + add a new `localfile` entry like below and save the file.

```bash
<!-- snort -->
<localfile>
  <log_format>snort-full</log_format>
  <location>/var/log/snort/snort.alert.fast</location>
</localfile>
```

You can place the section right below the `<!-- log analysis -->` part in the configuration file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697459912646/c4cf5d67-ffbc-4a9a-bc69-ff8ea6445429.png align="center")

That was it, the connection to wazuh is now setup and we only need to restart the wazuh agent to finish the integration.

```bash
sudo systemctl restart wazuh-agent
```

If this throws no errors you are good to go.

Go check your wazuh dashboard and click on the eye symbol next to the agent at the bottom right.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697487727115/bc8fa3fd-c4cd-4312-8578-3aaa82c5b349.png align="center")

This will bring you to the following overview. This dashboard holds all the events that your agent has collected.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697488180388/53fa57d5-deed-4076-9469-e9782e3ce3e6.png align="center")

*But... where are my IDS events?!*

Those are behind the next click - security events in the top left.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697488465426/62a3b380-58c4-428a-a867-20a24a1e20b8.png align="center")

Which brings you to the most important dashboard to look at for now - and if all went well - you see a lot of alerts from `ids` - your intrusion detection system - aka snort 🥳💜🎉 .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697488507201/7230fb90-38b4-41dc-be02-c4d78740f74a.png align="center")

Congratulations you did very well and now have a running IDS in your network.