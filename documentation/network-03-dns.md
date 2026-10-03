# Network Lab 03 — DNS Configuration and Troubleshooting

## Objective

Deploy BIND9 DNS on the Ubuntu Network Server, create a private lab DNS zone, configure the Ubuntu Network Client to use the lab DNS server, verify hostname resolution, and troubleshoot a deliberate DNS configuration failure.

## Network Topology

**Host-only network:** `192.168.56.0/24`

| Device                | IP Address       | Role       |
| --------------------- | ---------------- | ---------- |
| Ubuntu Network Server | `192.168.56.101` | DNS server |
| Ubuntu Network Client | `192.168.56.103` | DNS client |

### DNS Records

| Hostname     | IP Address       |
| ------------ | ---------------- |
| `server.lab` | `192.168.56.101` |
| `client.lab` | `192.168.56.103` |

## DNS Server Configuration

BIND9 and supporting DNS utilities were installed on the Ubuntu Network Server.

```bash
sudo apt update
sudo apt install bind9 bind9-utils dnsutils
```

The BIND9 service was verified to be running:

```bash
sudo systemctl status bind9
```

### DNS Zone Configuration

A new `lab` DNS zone was added to:

```text
/etc/bind/named.conf.local
```

```text
zone "lab" {
    type master;
    file "/etc/bind/db.lab";
};
```

A zone file was created at:

```text
/etc/bind/db.lab
```

```text
$TTL 86400
@   IN  SOA ns1.lab. admin.lab. (
        2026100301
        3600
        1800
        604800
        86400
)

@       IN  NS      ns1.lab.
ns1     IN  A       192.168.56.101
server  IN  A       192.168.56.101
client  IN  A       192.168.56.103
```

The zone file was validated with:

```bash
sudo named-checkzone lab /etc/bind/db.lab
```

The overall BIND configuration was also validated:

```bash
sudo named-checkconf
```

BIND9 was then reloaded:

```bash
sudo systemctl reload bind9
```

## DNS Testing

DNS resolution was first tested locally on the server:

```bash
dig @127.0.0.1 server.lab
```

The query returned:

```text
server.lab.    86400    IN    A    192.168.56.101
```

This confirmed that BIND9 was correctly serving the `lab` zone.

## Client DNS Configuration

The Ubuntu Network Client was initially using a DNS server provided through its other network connection.

The client was configured to use the lab DNS server on its host-only interface:

```text
192.168.56.101
```

The relevant Netplan configuration was:

```text
enp0s8:
  dhcp4: false
  addresses:
    - 192.168.56.103/24
  nameservers:
    addresses:
      - 192.168.56.101
```

The configuration was applied with:

```bash
sudo netplan try
```

The active DNS configuration was verified with:

```bash
resolvectl status
```

DNS resolution was then tested from the client:

```bash
dig server.lab
dig client.lab
```

Both records resolved to their expected IP addresses.

## Deliberate DNS Failure

A controlled failure was introduced by changing the client's DNS server from:

```text
192.168.56.101
```

to the incorrect address:

```text
192.168.56.102
```

The configuration was applied with:

```bash
sudo netplan try
```

The DNS lookup was then tested:

```bash
dig server.lab
```

The query returned no answer.

### Troubleshooting Process

The active DNS configuration was checked:

```bash
resolvectl status
```

This showed that the client was attempting to use:

```text
192.168.56.102
```

The reachability of that address was tested:

```bash
ping -c 4 192.168.56.102
```

The address was unreachable.

The known-good DNS server was then tested:

```bash
ping -c 4 192.168.56.101
```

The server responded successfully.

Based on these tests, the problem was identified as an incorrect DNS server address configured on the client.

The DNS configuration was restored to:

```text
192.168.56.101
```

After applying the corrected configuration, DNS resolution was tested again:

```bash
dig server.lab
```

The hostname successfully resolved to:

```text
192.168.56.101
```

## Skills Demonstrated

* BIND9 DNS administration
* DNS zones and A records
* IPv4 addressing
* Netplan network configuration
* Linux system administration
* `dig`
* `resolvectl`
* DNS troubleshooting
* Network connectivity testing with `ping`
* Technical documentation

## Key Takeaways

DNS provides name-to-IP address resolution. In this lab, the Ubuntu Network Client sent DNS queries to the BIND9 server at `192.168.56.101`, which returned records from the `lab` zone.

The troubleshooting exercise demonstrated the importance of separating DNS problems from basic network connectivity problems. When name resolution failed, checking the configured DNS server and testing its reachability helped identify the incorrect DNS server address.
