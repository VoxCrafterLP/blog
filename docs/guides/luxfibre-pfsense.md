# Set up POST Luxfibre with pfSense

In this guide, I'll explain how to use a pfSense router with the POST Bamboo package.
This includes the usage of the provided Fritz!Box as a VOIP box as well as IPTV.

# Setup

I'm running a pfSense box directly behind the provided [modem](https://biznes.tauron.pl/-/media/offer-documents/telekomunikacja/ont/nokia_7368_datasheet.ashx), every other device e.g the Fritz!Box
is running behind pfSense. For the router hardware, I use a mostly overkill Dell Poweredge R310 equipped with a
[Xeon X3470](https://www.intel.com/content/www/us/en/products/sku/42932/intel-xeon-processor-x3470-8m-cache-2-93-ghz/specifications.html)
and 16GB RAM. I also upgraded the networking part by adding a [dual port gigabit ethernet card](https://www.amazon.de/gp/product/B071R3YS2H?psc=1).
I'm running pfSense version 2.6.0-RELEASE.

``` mermaid
graph LR
  A[POST] --> B[Modem];
  B -->|PPPoE| C[pfSense]
  C --> D[LAN network]
  C --> E[server network]
  C --> F[Fritz!Box]
```

# Configuration

??? Note

    This part assumes that you have already installed pfSense, and you have at least basic knowledge on how to use it.

!!! Important 

    To follow along, you will need your PPPOE username and password! You can get them by simply calling the POST support
    and asking for it.

## Set up an internet connection

### Configure VLAN

First, you have to go the firewall tab and add a new VLAN under VLANs. Under parent interface, select the
interface that's connected to the modem. Under VLAN tag put in the number 35, as it's the VLAN POST operates on.
Lastly, you can give it a simple description like ^^Luxfibre VLAN^^.

### Add interface assignment

Switch to the *interface assignments* tab and select the created VLAN as the network port of the WAN interface.
<br>
Put in the configuration below:

!!! example "Configuration"

    === "General configuration"
        
        * IPv4 Configuration Type: PPPoE
        * IPv6 Configuration Type: DHCP6 
    
    === "DHCP6 Client Configuration"
    
        * Use IPv4 connectivity as parent interface: checked
        * DHCPv6 Prefix Delegation size: 48

    === "PPPoE Configuration"

        * Username: <your PPPOE username\>
        * Password: <your PPPOE password\>
        * Dial on demand: unchecked

    === "Reserved Networks"

        * Block private networks and loopback addresses: checked
        * Block bogon networks: checked

Save and edit the PPPOE interface in the PPPs tab and verify that the Luxfibre VLAN is selected.
<br><br>
At this point, the router should have picked up an IP address, and you should already see traffic on the dashboard. If it doesn't
connect immediately, wait up to 5 minutes and check whether you followed the steps above.

??? Tip

    Add the traffic graphs dashboard widget to see live traffic. 


## Set up VOIP

This section is not finished yet.

<!---

Now, we will focus on setting up the Fritz!Box as a VOIP client running behind the pfSense.

### Fritz!Box configuration


### pfSense configuration

-->

## Set up IPTV

### Add the interface

To add an interface for IPTV, go to the *interface assignments* tab and add a new interface with the VLAN 35 selected
as the network port. You can give the interface a recognizable name such as ```IPTV```.
Then set the IPv4 configuration type to static and IPv6 to none. Give the interface a static
IP address of ```10.10.10.10``` with a mask of ```32```. Make sure that private networks are ^^NOT^^ blocked.

### Firewall rules

In this section we will add the required firewall rules for IPTV. To do so, switch to the IPTV tab in the firewall
rules. Then add the following rules.

!!! example "Firewall rules"

    === "IGMP"
        
        * Interface: IPTV
        * Protocol: IGMP
        * Source: any
        * Destination: Network - 224.0.0.0/3
        * Allow IP options: checked

    === "UDP"
    
        * Interface: IPTV
        * Protocol: UDP
        * Source: any
        * Destination: Network - 224.0.0.0/3
        * Allow IP options: checked

!!! note

    Replace the *IPTV* interface with the name of your interface.

### Configure IGMP proxy

Next, you have to configure the IGMP proxy. You can find its configuration in Services > IGMP Proxy.
First, add a proxy for your IPTV interface of type ```upstream```. Add the network ```172.17.0.0/12``` to the
networks list.
<br>
Then add a second proxy of type ```downstream```. Select the interface which is connected to your Set-Top box.
Add the IP address of your Set-Top box with the mask of ```32``` as your network or if you have multiple boxes the
corresponding subnet.
<br><br>
Lastly you have to enable the IGMP proxy in the general IGMP options and start the service. By now you should be
able to watch live TV.

??? Tip

    Install the *Service_Watchdog* package and add the IGMP proxy to the monitored services.

### Known issues

Some low-end devices may experience connection timeouts because of the multicast packages. You can counteract this
by enabling IGMP-Snooping on your switches or separating the multicast traffic by creating a dedicated network for
your Set-Top boxes.