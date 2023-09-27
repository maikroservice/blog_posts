---
title: "Setting Up Wazuh as Your SIEM on Debian 12 & Proxmox: A Step-by-Step Guide"
datePublished: Fri Aug 25 2023 13:07:03 GMT+0000 (Coordinated Universal Time)
cuid: cllqlxc82000209l194wsd4kj
slug: setting-up-wazuh-as-your-siem-on-debian-12-proxmox-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/qTEj-KMMq_Q/upload/1bd7fa129391f19deeab61f033351654.jpeg
tags: hacking, cybersecurity-1, siem, soc-analyst

---

Welcome to the SIEM Homelab Series - We will walk through the process of installing your very own instance of Wazuh as a Security Information and Event Management System (SIEM).

If you want to do threat research or learn more about the ins and outs of security monitoring it is time to start your own home lab.

# Getting started

We will use a plain Debian image (iso) which you can download from: [https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/). Make sure that the SHA256 or SHA512 hash of the file you downloaded matches the original one. You can see the expected hash in this file: [https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/SHA256SUMS](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/SHA256SUMS)

```bash
# add the correct filename
sha256sum <debian-12.1.0-amd64-DVD-1.iso>
```

## Setting up the virtual machine on proxmox

Once that is done and the iso is added to proxmox (if you want to learn how to do that: [https://maikroservice.com/how-to-upload-iso-files-to-your-proxmox-server](https://maikroservice.com/how-to-upload-iso-files-to-your-proxmox-server)) you can create a new virtual machine with the blue button in the top right corner of your proxmox interface.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682165075447/29ed8bdb-3501-4e6a-a46c-47a7030bb4b5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682165082404/75b7afad-232f-4368-9a23-fd154b98e8cd.png align="center")

Now give it a name and click "Next", in the next window select the operating system that you want to use for the wazuh machine - we use Debian 11/12 and click "Next".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691872051353/37cfa0d8-5c87-4870-bf69-9e06f4a66122.png align="center")

In the System window select the Qemu Agent checkbox, and leave the rest as it is.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682165098218/7f7be9f7-536f-4f8d-91d1-b802e304933d.png align="center")

Next up we need to decide how much power our SIEM server needs. The documentation recommends the numbers below but we will adjust them slightly.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682165795687/5b850099-7e3f-4879-b711-73821b3c1c72.png align="center")

In the disk setup, make sure to use enough disk space for the VM, wazuh recommends around 50GB per 90 days of storage, since my SIEM does not run 24/7 I chose 50GB total disk space.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682166914625/1392a1c1-7873-42fd-9a8b-85dfee544e07.png align="center")

The number of CPU cores (4) was taken from the recommendations in the wazuh documentation. This is plenty, you might also get away with 3.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682166920075/094e59a2-6cd4-474a-9883-5a926e0a6984.png align="center")

For memory, I have been using 4GB (4096 MB) and it is running smoothly with 4-8 agents reporting to the SIEM. If you have more memory to spare, you can generously upgrade this to 8GB (8192 MB).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682166889998/a76fb310-e8e3-48d3-a31e-47e54c297584.png align="center")

For a network device, we use the classical `VirtIO (paravirtualized)`, if you have installed another network bridge outside of `vmbr0`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691872131860/37b34852-a773-4913-8566-927b213d4f91.png align="center")

Next up we confirm all the settings and press the `Finish` button.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691872790585/68fd7ed1-e21b-4233-89c5-6e73f8367291.png align="center")

## Installing debian on the VM

You now have to install the operating system on the virtual machine, which automatically begins after you start the virtual machine.

I suggest the graphical install option for visual pleasantries.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877881819/3508c14b-e0b8-4a99-9b68-e3af0b26ef3b.png align="center")

First up is selecting a language - use your favorite one, we will go with plain old English.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877884474/fa884da9-8e27-4fc8-b724-b71c47140644.png align="center")

Next up is the location selection - this will be used later on for time zones as well so make sure to select the correct one for you and press the continue button.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877894486/79d804c8-831d-4b0f-a3c5-374f02217a66.png align="center")

Now you need to choose the correct keyboard layout and hop on to the next selection screen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877903606/4e1bba9c-a2ab-494d-9153-25af8e2f8792.png align="center")

### User setup debian

It is time for your computer to get a name, choose something descriptive or stay by your naming scheme.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877910580/a9bd4149-1ec2-41ac-841b-b4c381ec277d.png align="center")

If your SIEM should be part of an Active Directory Domain you can add the name of the domain now - you can also set it up later in case you are not sure right now.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877916684/748a4c95-9d53-4844-8587-757848acc844.png align="center")

Debian will set up at least two users for you - one root user (admin) and one normal user.

First, you enter the password for the root (system administrative) user twice and once that is done you can give your normal user a name.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877921350/b1c154fb-bb6f-4883-8861-24417a0104e1.png align="center")

This user is the one you would log in with for daily operations, make sure you remember this name or add a note to the VM.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877934477/826803bf-d1a9-4fcb-a109-e970fa7e8d92.png align="center")

Once the username is selected you enter a password for this user twice and continue onwards.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877946847/2e267a5b-07c3-4269-82fa-9fe1fb405bcb.png align="center")

Now comes the time zone selection, remember earlier when I said that this is limited by the country you choose? Hopefully you selected the correct one and can find your time zone now, otherwise either choose a random one and change it later or go back to the country selection.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877953535/66b691b8-b696-4b1f-a3fc-aa6e2b9ee969.png align="center")

### Disk setup debian

Next, you can opt to choose a guided or manual approach to setting up the disk for your debian installation. I suggest you use the first option `Guided - use entire disk`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877977861/5187c5d5-52f3-4f3e-9334-91d7fc771329.png align="center")

The next three steps are single select & continue workflows. First is the disk selection, you probably have only one disk available if you followed the process until now. Choose that one and continue.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877981247/2c53e812-fb46-487c-a020-25a3d1aea408.png align="center")

We are now able to choose if you want different partitions (think of "virtual hard drives") or a single one - I suggest using the single one for ease of use.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877987212/755b155c-5bb4-426c-a15d-ee870b76467a.png align="center")

Now all the details are figured out and you need to finally confirm the partitioning + disk erasure.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877991675/8ff4c42f-efd8-428e-acc1-b63dd6b66257.png align="center")

Confirm once more and you are done with the disk setup.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877995486/972bef92-bdd3-4e6d-aa91-e13840d8ea44.png align="left")

### Software setup debian

You will want your debian to be and stay up-to-date and to achieve that you need software updates. The first selection screen will give you the option to load packages/libraries from a USB disk/external hard drive. Since you most likely don't have one you can choose `No` and continue.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878004058/77e4304f-b764-4fc0-a42e-02b508b7f571.png align="center")

debian uses `apt` (Advanced Package Tool) for most of the software installation. Apt works with mirrors + archives which hold the actual libraries you want to install and since the world is a big place you can choose the mirror location closest to you to have minimum latency.

You can leave this in the default setting, it should not have much impact on your daily work.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878010134/e84f1efc-530e-46a7-bd54-83b67822fc5f.png align="center")

Now comes the actual mirror selection, just leave this at `deb.debian.org` and continue.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878016274/98fd1109-4276-46d5-9574-6920c71833ad.png align="center")

If your internet is proxied you can now enter the correct proxy information - if you have not set one up then leaving this blank is most likely the right choice.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878021776/d826ea0a-3231-433f-a8c0-4d279ee57fdf.png align="center")

Now comes the option to share anonymous usage data for the packages you installed/use - I choose `No` because I don't like telemetry data collection, anonymous or not.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878026401/0dc1b1b7-2052-4531-9697-47b94fd4c57a.png align="center")

The next step is a little confusing if you are doing this for the first time - but fear not you can do it.

This selects your desktop environment (if you want one) - the default setting is `Debian desktop environment`, `GNOME`, and `standard system utilities`. I prefer `KDE` (taskbar at the bottom, similar to Windows/Mac) over `gnome` and thus have chosen `Debian desktop environment`, `KDE Plasma` and `standard system utilities`.

You could also get away without the desktop environment and would then probably need the `SSH server` to connect easily to the VM.

If this interests you let me know in the comments and we can dive deeper into how that setup would look.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878030355/d13320f4-367b-48b2-a34e-a096b78a5d4d.png align="center")

## Finishing debian installation

The penultimate step is to set up the grub boot loader which is accomplished by choosing yes to the question below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878035714/6dff3dba-dc5a-4c0e-a825-cf86021c4445.png align="center")

Last but not least we need to install said boot loader on the (only) disk we have and that is the last step of the debian install.

Reboot and login as the user you defined earlier.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878040087/b5edcb69-9231-4bb1-92fb-2405c0477671.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878045466/e32852e9-5f70-4037-bcc1-550531fbeadc.png align="center")

## Installing wazuh SIEM

The first thing you have to do is visit [https://documentation.wazuh.com/current/quickstart.html#installing-wazuh](https://documentation.wazuh.com/current/quickstart.html#installing-wazuh) and copy the command shown.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878049917/caf921dd-ad8f-4aaf-87fe-b7e8a1858389.png align="center")

There is one more task before the install process can commence - debian by default does not have curl installed so we need to do that.

You can copy the commands below to get it started.

```bash
# first we become root so that we can install packages
su -
# next install curl
apt-get install curl
# and install wazuh
curl -sO https://packages.wazuh.com/4.5/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878059436/d472af1e-3467-48ee-8c7a-11d657f2cb8f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878090474/f2bd38fb-e6cb-4f91-add1-e7f00920c77b.png align="center")

In the end, there will be a username/password combination for you to copy and paste into your password manager.

You are using a password manager, right?! RIGHT!?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878097763/32a0e97b-f8c2-4362-bd79-394180f13667.png align="center")

Now the installation is finished and if all went well wazuh is running on your machine.

How do you access it?!

Glad you asked, you can either open the browser on the SIEM machine - or if you want to connect remotely type `https://<IP_of_your_wazuh_machine>`.

There will be an error telling you that the `Server's certificate is not trusted` which is expected because it does not come from a certificate authority (CA).

You can safely ignore this error and will be greeted by the login screen of wazuh.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691878102419/c99ad85a-4383-4156-958b-f3ccbb1ac6ad.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692966538017/d3e16a05-9018-46e8-9d8f-bad7d327f557.png align="center")

After the login wazuh checks the availability of it's APIs and services and once that is done you can see the wazuh dashboard.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691877869643/77e75e4a-04f0-4b13-a24b-b012506091de.png align="center")

The dashboard looks like this and while yours will not have any agents registered you can do that next.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692950598094/54bdb0e3-77c7-4d0a-abfa-e5e053c7e80b.png align="center")

### installing wazuh windows agents

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692966974727/65614194-a6b0-4f2e-8371-8a4bd2d54918.png align="center")

You will now install a wazuh agent on a windows machine first

Start the Windows VM and open the following URL in your browser

[https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)

We will use the Graphical User Interface (GUI) of the wazuh agent to set everything up

You can get the installer here:

```bash
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.5.1-1.msi
```

You need administrative privileges to set everything up - keep that in mind.

Download and double-click that bad boy as if there is no tomorrow.

and then do the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692967809991/3b92da97-1644-4d92-b682-fab9c2798f25.png align="center")

You can change the location of the installation via the “advanced” button

but generally, the “Install” button should be your best friend, so click that one

When the installation is finished there is a checkbox that you can try to click on - “Run Agent configuration interface”

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692967987378/dfb9b869-f010-4f5f-85a6-49e6113ef144.png align="center")

For me that sometimes works and sometimes does not, here is a trick that always works:

open `C:\Program Files\ossec-agent`

and double click on `win32ui.exe`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692968051380/95a7fbad-b452-42a9-a361-124512885043.png align="center")

That will spawn a management window where you enter the IP of your SIEM server, click on Save and pray that you get an Authentication Key back

IF not…

You need to make sure that the wazuh-server is running

* that the machines are on the same subnet / have a working connection
    

If all works:

You should see the agent in your wazuh dashboard if all went well 🥳🎊

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692968087843/29d5a837-219d-4b76-9161-1e4b40db3a68.png align="center")

🥳 1 down, 1 to go for today.

Next up is linux.

### install linux agent

Installing the agent on a linux system depends a little on which linux distro you are running.

The process starts like this:

Visit [https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)

and click on the correct linux 🐧 package manager:

```bash
Hint:
Amazon Linux / CentOS → Yum
Debian-based (e.g. ubuntu/kali) → APT
Container (Alpine) → APK
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692968345571/105adbfd-9e24-4d66-8716-de4a2d1ededb.png align="center")

I will show the process with a debian box, so I choose `apt`.

Now we need to follow the steps for APT in my case (ubuntu/debian)

copy the first command and paste it into your terminal inside the linux VM

then the 2nd

and so on

Don’t forget to press Enter in between 🤓

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692968424253/2347cfaa-a255-48d8-9a3a-484962782e49.png align="center")

But what do the commands do?!

First you add the public encryption key to your linux key store (keyring)

Then you add 2 new repositories to your linux source list

The 3rd step updates your local package cache so that you can now use

apt to install the wazuh agent.

There is a teeny-tiny BUT though…

In order to properly connect your SIEM and the agent you need to feed a variable called `WAZUH_MANAGER` with the SIEM IP into the command

EXCUSE ME - WHAT ARE YOU TALKING ABOUT MR MAIKRO?!

There is some black magic going on behind the scenes that automagically connects your wazuh agent with the SIEM server 🪄

BUT only if you provide the IP address of the server:

`WAZUH_MANAGER=<IP_HERE> apt-get install wazuh-agent`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692968537151/daba9118-b8c1-4faa-9469-75a29c52f7c7.png align="center")

You can however also register the agent after installing by editing

`/var/ossec/etc/ossec.conf`

and adding the Manager\_IP between the address tags:

```bash
<client>
      <server>
        <address>MANAGER_IP</address>
				[...]
```

Source: [https://documentation.wazuh.com/current/user-manual/agent-enrollment/via-agent-configuration/linux-endpoint.html](https://documentation.wazuh.com/current/user-manual/agent-enrollment/via-agent-configuration/linux-endpoint.html)

If all went well you can now add the agent service to the auto start services by running three commands:

`systemctl daemon-reload`

`systemctl enable wazuh-agent`

`systemctl start wazuh-agent`

Once that is done you should see the agent appear in your wazuh dashboard

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692968696885/917ad77b-1648-4145-9ea7-1204d5c68769.png align="center")

🔥 CONGRATULATIONS 💙

You installed two wazuh agents plus a SIEM in your HomeLab 🎉