---
title: "Post: Fortinet VOIP LLDP profile"
last_modified_at: 2024-05-21T10:00:00
categories:
  - Blog
tags:
  - Fortinet
---

## Leveraging LLDP Profiles for Voice Devices on Fortinet: A Guide for Yealink Phones

As businesses grow, so does the complexity of their network infrastructure. Voice over IP (VoIP) devices, like Yealink phones, are a critical component of modern office communication systems. Efficiently managing these devices on your network ensures reliable performance and seamless operation. One way to achieve this is by using Link Layer Discovery Protocol (LLDP) profiles on Fortinet devices.

In this post, I'll walk you through configuring an LLDP profile specifically. We'll focus on setting up a profile that differentiates between data and voice VLANs, ensuring that VoIP devices (in my case Yealink) are properly recognized and configured.

### Understanding LLDP and its Benefits

LLDP is a vendor-neutral protocol used by network devices for advertising their identity, capabilities, and neighbors on a local network. LLDP helps in the automatic configuration of network devices, reducing manual intervention and errors.

When applied to voice devices like Yealink phones, LLDP can:
- Automatically configure VLAN settings.
- Ensure phones are on the correct VLAN for voice traffic.
- Improve network performance and reliability.

# Prerequisites:
- FortiGate
- FortiSwitch
- CLI access

### Step-by-Step Configuration

Let's dive into the specific commands needed to create and configure an LLDP profile on a Fortinet switch controller for Yealink phones. Be sure to already have 2 VLAN's. One for DATA and one for VOICE. 

#### 1. Access the Fortinet CLI

To configure the LLDP profile, you'll need to access the Fortinet CLI. 

#### 2. Create a New LLDP Profile

First, navigate to the switch-controller LLDP profile configuration:

```shell
config switch-controller lldp-profile
```
Next, create a new LLDP profile or edit an existing one. In this example, we'll make a new default profile named "standard":
```shell
edit standard
```

#### 3. Configure LLDP Settings
Set the LLDP profile to disable automatic inter-switch link (ISL) detection:
```shell
set auto-isl disable
```
#### 4. Configure MED Network Policy for Voice
We need to create a Media Endpoint Discovery (MED) network policy specifically for voice. This helps the network identify and configure voice devices correctly.
```shell
config med-network-policy
edit voice
set status enable
set vlan-intf VOICE
set dscp 46
next
end
```

In this configuration: 
- set status enable turns on the MED network policy for voice.
- set vlan-intf VOICE sets the VLAN interface for voice traffic. Ensure "VOICE" is the correct VLAN interface configured on your Fortinet device.
- set dscp 46 sets the DSCP (Differentiated Services Code Point) value for voice traffic, ensuring quality of service (QoS) prioritization.

#### 5. Verify Configuration
Once you've configured the LLDP profile, it's crucial to verify the settings to ensure everything is correctly applied. Use the following command to display the LLDP profile settings:
```shell
get switch-controller lldp-profile
```
This command will output the current configuration of your LLDP profiles, allowing you to confirm that the settings are correctly applied.

### Switch configuration
In the switch configuration, ensure that the `Native VLAN` is set to the DATA VLAN and the `Allowed VLAN` is set to the VOICE VLAN. This configuration is also effective for scenarios such as daisy-chaining the internet connection from a VoIP device to a PC. Additionally, be sure to add the newly created LLDP profile (in this case, `standard`) to the switch ports in the LLDP profile section.



# Conclusion
By following these steps, you can create an LLDP profile on your Fortinet device that ensures Yealink phones and other voice devices are correctly configured and prioritized on your network. This setup not only simplifies device management but also enhances the performance and reliability of your VoIP communications. Efficient network management is key to maintaining smooth business operations, and leveraging LLDP profiles for voice devices is a powerful strategy in achieving that goal. 
