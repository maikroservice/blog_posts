---
title: "How does traceroute work"
seoTitle: "how does traceroute work"
datePublished: Wed May 11 2022 16:49:40 GMT+0000 (Coordinated Universal Time)
cuid: cl31thepy03is1hnvdafeeswk
slug: how-does-traceroute-work
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/GRxTDmbsJRM/upload/v1652287501112/FBefy1vtv.jpeg
tags: interview, programming, security, hacking

---

Imagine you are working on a penetration test and you want to know how packets are traveling through the network. You want to identify routers and potentially other subnets, what tool do you use?

`traceroute` to the rescue!

## Summary

`traceroute` and its windows equivalent `tracert` work with the `ICMP` (Windows) or `UDP` protocol (Linux & Mac). It is generally used to follow the way packets travel through a network.

The software sends out packets with a Time To Live (TTL) of 1 first and awaits the `Type 11 - TTL exceeded` response, then sends out packets with a TTL of 2 and does the same again. This loop continues until either the destination is found, or the max TTL was reached (differs from implementations, common TTLs are 128 for Windows and 64 for Mac/Linux).

Traceroute then displays the time it took for the packets to go out and “come back” - the round-trip time.

When prepping for an interview this answer might suffice, you can stop reading now.

If you are still interested in more depth knowledge - strap in, its gonna be a wild ride.

FYI: We will use `traceroute` as the synonym for both traceroute and tracert for the rest of this post.

## So how exactly does it work?

Traceroute identifies hops across routers, each hop resembles a time to live (TTL) decrement of -1, which you might have come across when checking the TTL on ping requests and abstracting which operating system is running. Additionally, you usually can see how many routers are in between you and the target during CTFs, because there is a high chance there is only 1 🤓.

Quick divergence to what TTL actually is - a TTL is the amount of hops (routers), that a packet can pass before it is timed out / discarded by said router.

### What does that mean?

imagine you have a Windows machine and a typical TTL for packets is 128, so that means that this packet we just send can travel by 127 routers before the final router discards the packet.

Actually, discarding the package is not completely correct - what happens is that the final router sends an ICMP (TTL Exceeded) back to the origin, we can simulate this with a specific flag (`-m` for mac, `-t` for linux) - this seems to be independent of the original request - even when we send a UDP request, we get a TTL exceeded back via ICMP.

### Diving deep into packets and protocols with a fake shark fin on our head 🦈

Lets prototype something - first we use traceroute of the classical IP used for testing if the internet is on 🔥 - 8.8.8.8 (google).

We deliberately set the TTL to 1 to make sure that we don’t get far, not even out to the internet, this packet is under home arrest and gets stopped at my local router (192.168.0.1).

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652287393296/GkAXL-KTw.png align="left")

We can ask our friend wireshark to show us exactly the requests that were send and received, so lets check what the router returned.

### Request from my machine to 8.8.8.8

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652287407465/Wwg-_lkfn.png align="left")

There is a lot to unpack here.

First we see that my local IP is 192.168.0.44, the destination is 8.8.8.8 as we specified. We can also see that we used the UDP protocol to the remote port `33435` and locally we listen on `41101`. Additionally, 24 bytes of 0’s are send and I am still trying to figure out why.

### Response from my router to my machine

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652287425981/LMBCSMqbnE.png align="left")

Once my router has received the packet, reduced the TTL by 1 and it made the unfortunate realisation that this packet needs to stop and return to sender pronto because the TTL is 0 now. Therefore, it mirrors the original request and adds ICMP headers of `TTL exceeded`.

One potentially important note is that traceroute likes to measure response times and tells us how much time passed between sending the request.

When we inspect the previous command again we can see that we have 3 response times - why though?

This is because for each TTL-change 3 packets get send by traceroute and the durations we see here are the round-trip-times for the individual packets.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652287451629/UhnVJ17dW.png align="left")

### What does an asterisk mean in the results?

Good. You tried it yourself and at some point saw the death star in your terminal. The asterisk means that the router did not respond within the timeout limit, sad but okay.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652287461524/Y0A65nUx-.png align="left")