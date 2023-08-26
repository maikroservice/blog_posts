---
title: "How to upload iso files to your proxmox server"
datePublished: Mon Apr 24 2023 23:24:51 GMT+0000 (Coordinated Universal Time)
cuid: clgvgu2c7000d0amphl0x1ykf
slug: how-to-upload-iso-files-to-your-proxmox-server
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/yJVpnfqu8GY/upload/6a80ff07af0488fb08fbbb577bcd0608.jpeg
tags: hacking, cybersecurity-1

---

In this article, we will go through the necessary steps to add installation disk images (iso and img files) to your proxmox server so that you can use them as installation media for virtual machines and lxc/lxd containers.

First, make sure that you have created and selected the appropriate storage disk - this can be done by opening the proxmox management interface in your browser, selecting `Datacenter` and `proxmox`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682377914452/d41c0cfc-7eb4-4a27-a95c-6d03c05a9999.png align="center")

and then `Disks` and `Directory`. If you have only 1 drive (not recommended though) you will have a `local` disk directory already setup, as indicated by the four disks stacked on top of each other in the UI. This can be found under `Datacenter > proxmox > local` .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682377936196/2790993a-ea49-4060-925c-08857b62080a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682378006823/6c1f6899-5820-46cf-8c04-e32d6c6bcc1d.png align="center")

I will demonstrate the steps necessary to upload an iso disk image using proxmox 7.3.3, but later versions should have a similar setup.

The next step is to click on the respective disk that you want to select and upload the iso/img file to - in my case `local`.

Once that is done a new vertical navigation bar will appear, like the one below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682378161960/dd5c8405-b809-4e6b-a0b7-331049f41319.png align="center")

In this bar you need to select `ISO Images` which will contain our disk images. These can be uploaded from your computer via the `Upload` button or with `Download from URL` button.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682378550438/fba74578-5bfb-489c-8e7e-bfd553d3dc2e.png align="center")

Please note that there is a size limit for proxmox upload functionality - it is around ~2 GB depending on your specific instance. If you hit that limit your proxmox will show you a popup with the error code 0:

`"Error code: 0" or "Upload failed with code 0"`