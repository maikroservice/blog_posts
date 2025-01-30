---
title: "How to connect wazuh and discord: a Step-By-Step Guide."
datePublished: Mon Aug 21 2023 15:35:15 GMT+0000 (Coordinated Universal Time)
cuid: clll1gito000e09kt7l9o5mef
slug: how-to-connect-wazuh-and-discord-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/nsJeucVd7E0/upload/f552d36d7b23c1e6f6ff5a495638bc62.jpeg
tags: security, hacking, cybersecurity

---

Wazuh comes with a couple of external integrations by default - among them are Slack, Virustotal, shuffle, and Pagerduty.

The one that I missed was Discord - so I decided to build it. Now you can forward your all or specific alerts to a discord channel of your choice.

Here is the link to the code required: [https://github.com/maikroservice/wazuh-discord-integration/](https://github.com/maikroservice/wazuh-discord-integration/tree/main)

In order to get this running you need to do the following steps:

## Create Discord Server

First up create a discord server if you do not have one yet. In order to do that open Discord and click the big plus button:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692626391147/bdbf8439-ae21-4352-bd1e-e8c1b96e8e40.png align="center")

Then choose the `Create My Own` option and in the next screen select the option that meets your needs - I will choose the `For me and my friends` option, because this server is for internal use only.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692626419710/33db55b4-ec7c-4874-82d8-302e1a0c3dc8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692626475974/8e11b767-01cd-4496-af9d-d242609409cb.png align="center")

The only thing left to do is to give the server a name.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692626466350/0fd47fcb-bb19-43b3-acbe-a9053470b1af.png align="center")

### Add a private text channel

You need to either create or choose an existing channel where the alerts should be posted. I created a channel called `wazuh-alerts` to be explicit about the channel's purpose. Here is how you do that:

First up, click the small `+` button next to the `TEXT CHANNELS` - choose a `Text` channel and make sure to select the `Private Channel` option at the bottom. You could also directly allow access for specific roles/groups should you wish to do so, or you can skip the selection for now.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692626856282/75bbbe5f-c657-4352-9a67-15fbdae7d068.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692627010095/416dc58f-5057-449e-a02c-25d246f37397.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692627240762/90bc9114-5cdc-4b6a-8f67-71d42c10cc41.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692627311733/06b457f9-57db-4db5-b878-86a1c37bcc89.png align="center")

If all goes well you will now see a `NEW` channel with the correct name on your server.

### Create a Webhook integration

You need a webhook to send messages to your server - this is basically a bot member of your server that has the right to post messages in specific channels.

Right-click on your server - choose `Server Settings` -&gt; `Integrations` .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692626949990/052097a6-1255-4fe1-8edb-4d8649a32bb5.png align="center")

Then you will see the following screen and need to press the `New Webhook` button.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692628551317/ad3cfde6-ccea-4e80-aad2-9a994b03244c.png align="center")

Give that integration a name and select the channel to receive the alerts. You could also add a picture if you want to but it is not necessary for the next steps to work.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692628764518/5cfe51e9-af34-4b03-89a1-1e229bd20643.png align="center")

The important part is the `Webhook URL`, this is the url that wazuh connects to and sends the alerts. They will then be converted into messages and posted in the discord channel.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Keep in mind that anyone with this url can send messages to your chosen channel - protect it as much as you can.</div>
</div>

Copy the Webhook URL and save it for the next step.

## Register the integration in wazuh

1. You need to register the integration in wazuh - for that to happen start wazuh, connect to the dashboard and visit the configuration area in the Server management section.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738272690361/23e02c11-8290-43e4-9523-a7ff91667ce0.png align="center")

Next in the top right click on `Edit configuration`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738272724532/bb3e394b-24be-498c-bd64-a43029057233.png align="center")

Now you are editing the `ossec.conf` located here `/var/ossec/etc/ossec.conf`

Directly under the `</global>` tag you can paste the following code:

```xml
 <integration>
     <name>custom-discord</name>
     <hook_url>https://discord.com/api/webhooks/XXXXXXXXXXX</hook_url>
     <alert_format>json</alert_format>
 </integration>
```

The name needs to start with `custom-` - which took me the better part of an hour to debug... 😅

Now you do not have to make the same mistake.

When done correctly it should look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692630510447/c6c9291d-f625-4338-b643-059799237dec.png align="center")

### Add your discord webhook url

The last step in the configuration section is to add the correct webhook url between `<hook_url>` and `</hook_url>`. Once that is done you need to click the `Save` button and then click on the `Restart Manager` button afterwards, You should see a popup asking you to confirm the operation - click on `Confirm`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738272903860/d1029620-2463-441b-b952-638cc5cda7cc.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738273018234/e503b7c3-fada-4311-bc09-d8d785c7e323.png align="center")

Once that is done - your wazuh instance will tell you that the manager is restarting and once the `Restarting Manager` button turns blue again you are good to continue to the terminal for the final two steps.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738273025483/73fe4927-6a5c-4fbd-8490-073787f502b9.png align="center")

## Set up wazuh integration

Wazuh's integrations are located in `/var/ossec/integrations` - You will see 7 files there already.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692694139602/26992767-3753-4e87-a842-0605a6b4fe0d.png align="center")

Most integrations consist of two files - one Python file (e.g. slack.py) and one bash script (slack).

You need to add two files now: `custom-discord` and `custom-discord.py` - copy and paste the files from GitHub ([https://github.com/maikroservice/wazuh-discord-integration/](https://github.com/maikroservice/wazuh-discord-integration/tree/main)) into this folder. Next, use the following commands to change the permissions and adjust the ownership of the two files.

```bash
sudo chmod 750 /var/ossec/integrations/custom-*
sudo chown root:wazuh /var/ossec/integrations/custom-*
```

The folder should now look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738272805164/856aeb1f-ec4a-4263-b733-9c439a21c504.png align="center")

Once that is done make sure that pip is setup correctly and has the `requests` library installed.

```bash

# debian / ubuntu
sudo apt-get install python3-pip
pip3 install requests

# amazon linux / centos
sudo yum install python3-pip
pip3 install requests
```

Restart the wazuh manager and you should see alerts showing up in the discord channel.

```bash
/var/ossec/bin/wazuh-control restart
```

You can trigger it by typing the wrong password for a user on a machine running the wazuh agent. If all goes well you should see something like the screenshot below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692631973222/c733a7d6-cca0-41bf-a884-a46711c1f3c9.png align="center")

If you do not see the alerts in Discord you can first look at the logs of `integratord`. If you see an error such as the one below you might want to increase the verbosity of the `integratord` log.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692831675598/30b3fcd9-d9eb-491e-ba94-c2c716cb44a8.png align="center")

How you ask?! - Use the following command:

```bash
/var/ossec/bin/wazuh-integratord  -dd
```

That should give you a detailed description of what's going wrong and can fix it.

%%[goatcounter] 

### Sources

[https://documentation.wazuh.com/current/user-manual/manager/manual-integration.html#manual-integration](https://documentation.wazuh.com/current/user-manual/manager/manual-integration.html#manual-integration)