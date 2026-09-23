# Troubleshooting 01 — Windows ICMP Connectivity

## Problem

The Ubuntu network server could communicate with the internet and had connectivity to the Windows host-only network, but ICMP ping requests from Ubuntu to the Windows host failed.

## Initial Configuration

| Device        | Interface                    | IP Address        |
| ------------- | ---------------------------- | ----------------- |
| Ubuntu Server | enp0s8                       | 192.168.56.101/24 |
| Windows Host  | VirtualBox Host-Only Adapter | 192.168.56.1/24   |

## Symptoms

Ubuntu successfully reached external IP addresses and DNS names.

Windows could successfully ping the Ubuntu server:

```text
Windows → 192.168.56.101: Successful
```

However, Ubuntu could not ping the Windows host:

```text
Ubuntu → 192.168.56.1: 100% packet loss
```

## Troubleshooting Process

### 1. Verified Ubuntu interface configuration

The `enp0s8` interface was active and configured with:

```text
192.168.56.101/24
```

### 2. Verified routing

Ubuntu contained a route for the private network:

```text
192.168.56.0/24 dev enp0s8
```

This confirmed that traffic destined for the host-only network was being sent through the correct interface.

### 3. Verified ARP connectivity

The ARP neighbor table showed the Windows host at `192.168.56.1` as reachable.

This indicated that Ubuntu could resolve the Windows host's Layer 2 address.

### 4. Tested the reverse direction

Windows successfully pinged Ubuntu.

This further indicated that the VirtualBox Host-Only network itself was functioning.

### 5. Checked Windows Firewall

The Windows inbound ICMPv4 Echo Request rule for the Private network profile was disabled.

### 6. Resolution

The Private-profile ICMPv4 Echo Request inbound rule was enabled.

After enabling the rule, Ubuntu successfully received ICMP replies from:

```text
192.168.56.1
```

## Root Cause

Windows Firewall was blocking inbound ICMPv4 Echo Request traffic on the Private network profile.

## Resolution

Enabled the Windows Firewall inbound rule allowing ICMPv4 Echo Requests for the Private network profile.

## Lessons Learned

The troubleshooting process demonstrated the importance of checking network connectivity systematically:

1. Verify interface configuration.
2. Verify IP addressing and subnetting.
3. Verify routing.
4. Check ARP/link-layer connectivity.
5. Test connectivity in both directions.
6. Check host-based firewall rules when traffic is being selectively blocked.
