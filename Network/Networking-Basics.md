# What is a network?

A network is a group of devices or large networks connected to each other.
Two connected laptops or many local networks connected to each other can be
a network.

## Network devices

Network devices are all hardware devices, including PCs, laptops, phones and
routers, that are connected to each other.

- Device – a laptop or a phone that we use.
- Switch – a network device that we use to connect many devices.
- Router – a network device that we use to connect devices and to direct
  traffic to other networks.
- **Network interface / network adapter** – a physical device inside laptops
  and PCs that identifies the device in a network.

## Types of connection

- LAN (Local Area Network) – for example, devices connected in our home.
- WAN (Wide Area Network) – many LANs connected to each other, also
  across cities.

## How does information travel through a network?

For example, we have a laptop and we want to make a video call to our friend
who lives in another country. What happens, and how can we talk?

First of all, our laptop is connected to our router. The laptop has a network
adapter inside with a MAC address. A MAC address is the physical number of
a device's network adapter that identifies the device in a network.

The laptop starts sending ARP requests. When the router gets a request, it
sends a response, and it writes it in a table where the router keeps
destination MAC addresses.

Then the laptop sends frames to the router. A frame is a small part of
a packet. A frame includes the sender's MAC address and the destination MAC
address. It also includes the sender's IP address, the destination IP
address, time and data.

The router gets these frames, and when it is ready to send them to another
network, they leave our LAN. The router sends these packets through the
provider's routers to my friend's router.