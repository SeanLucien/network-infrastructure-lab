
# Network Lab 02 — Multi-VM Network, Static IP, Packet Capture, and SSH

## Objective

Expand the network infrastructure lab by adding a second Ubuntu Server virtual machine to the existing VirtualBox host-only network.

The goal was to configure static IPv4 addressing, verify communication between virtual machines, capture ICMP traffic, and establish SSH-based remote administration.

## Network Topology

```text
                 VirtualBox Host-Only Network
                       192.168.56.0/24
                              |
                +-------------+-------------+
                |                           |
        Ubuntu Network Server       Ubuntu Network Client
        192.168.56.101              192.168.56.103
        Static IP                   Static IP
                |                           |
                +----------- NAT -----------+
                       Internet Access
```

### Server

| Interface | Network   | Address           | Purpose             |
| --------- | --------- | ----------------- | ------------------- |
| enp0s3    | NAT       | 10.0.2.15/24      | Internet access     |
| enp0s8    | Host-only | 192.168.56.101/24 | Private lab network |

### Client

| Interface | Network   | Address           | Purpose             |
| --------- | --------- | ----------------- | ------------------- |
| enp0s3    | NAT       | 10.0.2.15/24      | Internet access     |
| enp0s8    | Host-only | 192.168.56.103/24 | Private lab network |

## Configuration

Created a second Ubuntu Server 24.04.5 LTS virtual machine with:

* 2 GB RAM
* 2 CPUs
* 20 GB virtual disk
* NAT adapter
* VirtualBox Host-Only adapter
* OpenSSH server

The client initially received its host-only address through DHCP.

Both systems were then configured with static IPv4 addresses using Netplan.

### Server

```yaml
enp0s8:
  dhcp4: false
  addresses:
    - 192.168.56.101/24
```

### Client

```yaml
enp0s8:
  dhcp4: false
  addresses:
    - 192.168.56.103/24
```

The configurations were applied with:

```bash
sudo netplan apply
```

## Connectivity Testing

Verified bidirectional connectivity using ICMP:

```bash
ping -c 4 192.168.56.101
```

and:

```bash
ping -c 4 192.168.56.103
```

Both directions completed successfully with no packet loss.

## Packet Capture

Used `tcpdump` on the server to observe ICMP traffic:

```bash
sudo tcpdump -i enp0s8 icmp
```

While the client generated ping traffic, the server captured the ICMP Echo Requests and Echo Replies.

This provided packet-level confirmation that traffic was successfully traveling between the two virtual machines.

## SSH Remote Administration

From the client, connected to the server using:

```bash
ssh sean@192.168.56.101
```

After authenticating, verified the remote system with:

```bash
hostname
```

which returned:

```text
ubuntu-network-server
```

Also verified the remote network interfaces with:

```bash
ip -br addr
```

This confirmed successful remote administration of the server from the client.

## Troubleshooting

The initial SSH connection failed because an incorrect password was entered. The SSH service itself was verified to be running, and the connection succeeded after using the correct server credentials.

This demonstrated the importance of distinguishing authentication problems from network connectivity or service availability problems.

## Skills Demonstrated

* VirtualBox virtual networking
* Linux server administration
* IPv4 addressing
* /24 subnetting
* DHCP vs. static addressing
* Netplan configuration
* ICMP connectivity testing
* `ping`
* `tcpdump`
* SSH
* Remote Linux administration
* Network troubleshooting
* Technical documentation

## Key Takeaways

This lab established the basic private network that will be used for future infrastructure experiments.

The environment now supports communication between multiple Linux systems, packet capture, and remote administration. Future exercises will build on this network by introducing services such as DNS and DHCP, followed by additional troubleshooting scenarios.
