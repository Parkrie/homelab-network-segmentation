# Troubleshooting Log

Issues encountered while setting up VLAN segmentation. Documented here because the mistakes were as educational as anything else in this project.

---

## Issue 1: Lost Access to Switch After Tagging Ports

**What happened:**

The original switch in this setup was a third-party smart switch. After configuring VLAN tags on the switch ports, I immediately lost all access to the switch management interface and could not get it back.

**Why it happened:**

When you tag a port on a managed switch, you are telling it to expect 802.1Q tagged frames on that port. The device on the other end also has to understand and send tagged frames, otherwise the two devices are no longer speaking the same language.

The management interface for the switch itself is accessed through a port. When I tagged that port without accounting for how management VLAN traffic needed to flow, the switch stopped responding entirely. The machine I was managing from was sending untagged traffic, the switch was now expecting tagged traffic, and the connection disappeared.

**What I tried:**

I attempted to reconnect through different ports and adjust settings, but without access to the management interface there was nothing to adjust. The only recovery option was a factory reset using the physical reset button on the switch.

This happened roughly three times across different config attempts. Each time I thought I had a better understanding of the problem, made changes, lost access, and had to start over.

**How it was resolved:**

The short term fix was factory resetting and approaching the config more carefully each time. The long term fix was replacing the third-party switch with one from the same vendor as the router.

The two devices handled VLAN configuration differently enough that keeping them in sync manually was error prone. The original switch expected configuration done entirely through its own interface with no awareness of the router. The router had no visibility into what the switch was doing.

Moving to a same-vendor switch meant both devices were configured through a single controller. Trunk and access port setup was handled consistently and the interface made it clear which ports were carrying which VLANs.

---

## Issue 2: Trunk vs. Access Port Misconfiguration

**What happened:**

After moving to the new switch, devices were either not getting IP addresses or landing on the wrong VLAN entirely.

**Why it happened:**

Trunk and access ports serve different purposes and the distinction matters a lot.

A trunk port carries multiple VLANs simultaneously using 802.1Q tags. Each frame includes a tag identifying which VLAN it belongs to. Trunk ports are used for uplinks between switches and routers where traffic from multiple VLANs needs to pass through the same cable.

An access port carries a single VLAN and strips the tag before the frame reaches the end device. The device plugged in has no idea it is on a VLAN. It just sees a normal network. Access ports are used for end devices like computers, cameras, and phones.

I had misconfigured some access ports to carry tagged traffic. End devices on those ports could not interpret the 802.1Q tags, so DHCP discovery frames were either being dropped or misrouted. Devices either got no IP address or picked up an address from the wrong scope.

**How it was resolved:**

Going through each port in the controller and explicitly setting access ports to untagged for the correct VLAN, and confirming the uplink port to the router was set as a trunk carrying all required VLANs as tagged. Once the port profiles were corrected, devices came up on the right VLANs and DHCP worked as expected.

**The thing that actually made it click:**

Reading about trunk and access ports is one thing. Having a device fail to get an IP and tracing it back to a tagged frame reaching hardware that does not understand tags is what made the concept stick.

---

## General Lessons

Getting locked out of a switch because of a misconfigured port is a rite of passage with managed networking. A few habits that came out of this:

- Before changing VLAN config on a port that carries management traffic, have a fallback plan to reach the switch through a different port or physically through the console if available.
- Make one change at a time. Changing multiple ports at once makes it much harder to identify which change caused a problem.
- Use the same vendor ecosystem for switches and routers where possible. Cross-vendor VLAN config is doable but adds a layer of complexity that creates room for mistakes.
