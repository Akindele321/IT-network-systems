**Segmented Home Lab: VLANs, Virtualized Router, and IoT Isolation**
A home network security lab built on a single repurposed Dell PC, using Proxmox, 
OPNsense, and a Cisco Catalyst 2950 switch to segment a network into isolated VLANs
then integrating Home Assistant and Philips Hue to prove the segmentation works with
a real IoT use case.

**Hardware**:
Dell PC (Intel) — hypervisor host
Cisco Catalyst 2950 switch — VLAN trunking and access ports
USB 2.5GbE adapter + WiFi extender — WAN uplink (used in place of a long Ethernet run to the router)
Philips Hue Bridge + 2x smart bulbs — IoT test devices

**VLAN	Purpose	Subnet**
10	Management	192.168.10.0/24
20	IoT	192.168.20.0/24
30	Servers	192.168.30.0/24
40	Guest	192.168.40.0/2

**Architecture**:
​```
Internet
  |
WiFi extender --(Ethernet)--> USB NIC --> Proxmox (vmbr1) --> OPNsense WAN
                                                                    |
                                                          OPNsense (router/firewall)
                                                                    |
                                                    802.1q trunk (Proxmox vmbr0)
                                                                    |
                                                       Cisco 2950 (fa0/1 trunk)
                                                     /        |         |        \
                                              fa0/2       fa0/3      fa0/4     fa0/5
                                            VLAN 10       VLAN 20    VLAN 30   VLAN 40
                                          Management       IoT      Servers    Guest
                                                          (Hue)   (Home Assistant)
​```
What's Running
Proxmox VE — bare-metal hypervisor on the Dell, hosting both VMs below
OPNsense — virtualized router/firewall doing router-on-a-stick routing across all four VLANs, plus WAN/NAT
Home Assistant OS — smart home hub, living on the Servers VLAN, integrated with the Hue Bridge

Build gallery
1. Smart bulbs live
Two Philips Hue smart bulbs connected and controllable through Home Assistant,
confirming the IoT VLAN could reach the Home Assistant server end to end. This
was the payoff after getting VLAN segmentation, routing, and the Zigbee bridge
integration all working together.

2. Desk setup
   The physical build: a Dell PC running Proxmox as the hypervisor,
   a Cisco Catalyst 2950 switch handling VLAN trunking, and a laptop
   for managing everything through the web GUIs. The laptop screen
   shows OPNsense's DHCP lease table confirming devices were pulling
   addresses correctly on their assigned VLANs.

3. Hardware laid out
   Core hardware for the lab: the Cisco 2950 switch, a Philips
   Hue Bridge with bulbs for the IoT segment, and a WiFi extender
   used as a wireless WAN uplink. The extender added internet
   access to the lab without running an Ethernet cable down from
   the router two floors below.

4. Home Assistant dashboard
   Home Assistant's dashboard once fully onboarded, showing organized
   Areas and active light entities integrated through the Hue Bridge.
   Reaching this screen required giving the network real internet access
   first, since Home Assistant Core wouldn't finish installing while
   completely offline.

5. OPNsense dashboard
   OPNsense's dashboard showing all core services active and WAN successfully
   holding a DHCP lease. Getting here took real troubleshooting, more in the
   write-up below.

6. Proxmox datacenter view
   Proxmox's summary view showing OPNsense and Home Assistant running as separate
   VMs on one physical host, sharing the Dell's CPU, RAM, and storage.

**Troubleshooting write-up: the VLAN 10 DHCP issue**

Early on, devices plugged into the VLAN 10 (Management) access port weren't receiving DHCP 
leases at all, even though the interface itself showed up and had the correct IP. Tracing 
it down took a few layers:

1. **Confirmed the physical and Layer 2 path first**: the switch port was up, in the right VLAN,
   and the trunk was correctly passing all four VLANs to OPNsense. That ruled out cabling and
   switch config.

2. **Checked OPNsense's DHCP service directly**: this is where the real issue was found: Kea, OPNsense's
   newer DHCP backend, was configured with the right subnet and pool, but the service itself was never
   actually running (inactive). The GUI showed it as configured, which made it look like it should have
   worked.

3. **Found the actual active DHCP service was dnsmasq, not Kea**: a mismatch between what was configured
   and what was actually serving requests on the box.

4. **Fixed it by configuring DHCP through dnsmasq instead**: since that was the service genuinely listening
   on port 67. Once dnsmasq had the correct range for each VLAN interface, leases started working immediately.
   

