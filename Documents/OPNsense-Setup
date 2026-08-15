# Part 1 — OPNsense Firewall and Basic Network Setup

## Objective

I wanted to do this project to practice my networking and security skills. This is also practice for when I set up a physical firewall on my home network, which I plan to do in the future.

For this first part of the project, my goal was to install OPNsense as a virtual firewall and router, create an internal network for my Ubuntu Server, and verify that the server could access the Internet through OPNsense.

---

## Initial Environment

To start, I already had VirtualBox installed. VirtualBox is a virtualization program that allows different virtual machines running different operating systems to run on the same physical computer. This allows me to build and test different network configurations without directly affecting my main machine.

From previous projects, I already had an Ubuntu Server VM installed with around 4 GB of RAM and 30 GB of storage.

For the firewall, I chose OPNsense because I had heard it was a good option for a home lab and because I could build out different firewall and networking features one at a time, which seemed useful for my first firewall project.

I downloaded the OPNsense ISO from its website and attached it to the VM as virtual installation media.

I configured the OPNsense VM with:

- **Memory:** 4 GB RAM
- **Processors:** 2
- **Storage:** 16 GB

This should give me a good starting point while still leaving enough resources available for the other virtual machines in the lab.

---

## Network Design


The basic network I created looks like this:

```text
                         Internet
                            |
                      VirtualBox NAT
                            |
                        10.0.2.15
                         WAN (le0)
                            |
                     +-------------+
                     |   OPNsense  |
                     |   Firewall  |
                     +-------------+
                            |
                         LAN (le1)
                       192.168.1.1
                            |
                         LAB_LAN
                            |
                      192.168.1.143
                         enp0s3
                     +-------------+
                     |   Ubuntu    |
                     |   Server    |
                     +-------------+
                         enp0s8
                     192.168.56.101
                            |
                    Host-Only Network
                            |
                    Linux Mint Host
                     192.168.56.x
```

### Network Configuration

| Device | Interface | VirtualBox Network | IP Address | Purpose |
|---|---|---|---|---|
| OPNsense | WAN (`le0`) | NAT | `10.0.2.15/24` | Provides outside/Internet connectivity through VirtualBox |
| OPNsense | LAN (`le1`) | Internal Network — `LAB_LAN` | `192.168.1.1/24` | Default gateway and firewall interface for the lab |
| Ubuntu Server | `enp0s3` | Internal Network — `LAB_LAN` | `192.168.1.143/24` | Places the server behind OPNsense for routed/firewalled traffic |
| Ubuntu Server | `enp0s8` | Host-Only Adapter | `192.168.56.101/24` | Separate management connection used for SSH |
| Linux Mint Host | Host-Only Interface | Host-Only Network | `192.168.56.x/24` | Allows the physical host to communicate directly with the Ubuntu Server |

My Ubuntu Server has two network connections.

One connection is a Host-Only Adapter that allows my physical Linux Mint machine to communicate directly with the server for management tasks such as SSH.

The second connection places the server on `LAB_LAN`, behind the OPNsense firewall. This allows the server's Internet-bound traffic to pass through OPNsense, where I can later create firewall rules, monitor traffic, and experiment with other security features.

---

## OPNsense Installation

I started the OPNsense VM and went through the installation process.

For the ZFS virtual device type, I selected **Stripe**, which provides no disk redundancy. Since this firewall is only being used for learning and experimentation and the VM only has one virtual disk, redundancy is not necessary for this setup.

For a future physical firewall or server, however, I would like to experiment with redundant storage such as RAID 10 where appropriate.

After installing OPNsense onto the virtual disk, I removed the installation ISO from the virtual optical drive and booted from the newly installed system.

---

## WAN and LAN Configuration

After installation, I configured the WAN and LAN interfaces.

The WAN interface uses VirtualBox's **NAT** network, which gives OPNsense a path to the outside network and Internet through the host machine.

The LAN interface is connected to a VirtualBox **Internal Network** that I named `LAB_LAN`. This is the network where machines such as my Ubuntu Server can communicate through OPNsense.

The interfaces were configured as:

| OPNsense Interface | Interface Name | Address |
|---|---|---|
| WAN | `le0` | `10.0.2.15/24` |
| LAN | `le1` | `192.168.1.1/24` |

The basic traffic path is:

`Ubuntu Server → LAB_LAN → OPNsense LAN → OPNsense WAN → VirtualBox NAT → Internet`

Before connecting the Ubuntu Server, I used OPNsense's ping utility to ping `8.8.8.8`. The ping was successful, confirming that OPNsense itself had working WAN connectivity.

**[INSERT OPNsense WAN/LAN SCREENSHOT]**

---

## Connecting the Ubuntu Server

I changed the Ubuntu Server's first VirtualBox network adapter from NAT to the `LAB_LAN` Internal Network.

After booting the server, the interface received `192.168.1.143/24`, with `192.168.1.1` configured as its default gateway.

I left the server's second adapter connected to my existing Host-Only network. This gives me a separate connection between my Linux Mint host and Ubuntu Server that I can use for SSH management while experimenting with the firewall network.

I used the following commands to examine the configuration:

```bash
ip addr
```

```bash
ip route
```

The routing table showed:

```text
default via 192.168.1.1
```

This confirmed that OPNsense was now the Ubuntu Server's default gateway for traffic going outside of its local networks.

**[INSERT UBUNTU IP/ROUTING SCREENSHOT]**

---

## Connectivity Testing

After configuring the interfaces, I switched to my Ubuntu Server to verify three things:

1. Can it reach its default gateway?
2. Can it reach the Internet?
3. Does DNS resolution work?

I tested each part separately using `ping`.

### Default Gateway

```bash
ping -c 3 192.168.1.1
```

**[INSERT DEFAULT GATEWAY PING SCREENSHOT]**

I first pinged `192.168.1.1`, which is the OPNsense LAN interface and the Ubuntu Server's default gateway.

The ping was successful, confirming that the server could communicate with OPNsense across `LAB_LAN`.

### Internet Connectivity

```bash
ping -c 3 8.8.8.8
```

**[INSERT 8.8.8.8 PING SCREENSHOT]**

Next, I pinged `8.8.8.8` to test whether the server could reach an external IP address.

This test was successful, showing that traffic could travel from the Ubuntu Server through OPNsense and reach the Internet.

Using an IP address for this test also allowed me to test Internet connectivity independently from DNS resolution.

### DNS Resolution

```bash
ping -c 3 google.com
```

**[INSERT DNS PING SCREENSHOT]**

Finally, I pinged `google.com`.

Since the hostname has to be resolved into an IP address before the ping can be sent, the successful test confirmed that DNS resolution was also working.

I also checked the server's DNS configuration and confirmed that it was using `192.168.1.1` as its DNS server, which is the LAN interface of OPNsense.

---

## Results

At the end of Part 1, I had a working OPNsense firewall/router with an Ubuntu Server located on its internal LAN.

The Ubuntu Server was able to reach its default gateway, resolve DNS names, and access the Internet through OPNsense.

The completed network path was:

`Ubuntu Server → OPNsense → VirtualBox NAT → Internet`

This gives me a working base network that I can expand in later parts of the project with firewall rules, network segmentation, monitoring, and other security features.

---

## What I Learned

### NAT

Before this project, my understanding of NAT was that it was somewhat similar to a LAN, or that it was how LANs were connected together, but I wasn't too sure or confident in my understanding.

After doing this section and installing OPNsense on the virtual network, I have a better understanding of NAT as a way of translating network addresses as traffic moves between different networks.

In my setup, OPNsense has a private WAN address of `10.0.2.15` provided through VirtualBox's NAT network. This allows OPNsense to reach outside of the virtual network and access the Internet.

I also learned that NAT and a LAN are two different concepts. `LAB_LAN` is the internal network that my Ubuntu Server belongs to, while NAT is being used as part of the path that allows traffic from the private lab network to eventually reach outside networks.

### Network Troubleshooting

I also learned why it is useful to test each part of a network separately when troubleshooting.

By testing each part individually, I can see where communication is working and where it starts to fail, which can save time when trying to find the cause of a problem.

The three pings I performed each told me something different about the network. For example, if I could successfully ping `192.168.1.1` but could not ping `8.8.8.8`, I would know that the Ubuntu Server can reach its default gateway but cannot reach the Internet. This narrows down the problem and tells me that I should look beyond the local connection between Ubuntu and OPNsense.

If `8.8.8.8` worked but `google.com` did not, I could then investigate DNS resolution instead of assuming that the entire Internet connection was down.

### WAN and LAN Interfaces

Thanks to my university classes, CompTIA Network+ studies, and general networking knowledge, I already understood the basic difference between a Local Area Network (LAN) and a Wide Area Network (WAN).

Configuring OPNsense helped reinforce that understanding by letting me actually assign different network interfaces to WAN and LAN roles instead of only learning about them conceptually.

In OPNsense, I configured `le0` as the WAN interface connected to VirtualBox NAT and `le1` as the LAN interface connected to my internal `LAB_LAN` network.

This helped me see how a router/firewall can sit between two different networks and control how traffic moves between them.

---

## Next Steps

In the next part of this project, I plan to begin working with OPNsense's firewall management features. This will include examining the existing firewall rules, creating my own rules, testing allowed and blocked traffic, and using logs to see how OPNsense handles that traffic.
