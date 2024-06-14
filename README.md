# NSD_Project_2324
Network and System Defence Final Projects AY 2023/2024

## Topology
![[topology.png]]

## AS100
![[as100.png]]
***AS100** is a transit Autonomous System providing network access to two customers: AS200 and AS300*
- *Configure eBGP peering with AS200 and AS300*
- *Configure iBGP peering between border routers*
- *Configure OSPF*
- *Configure LDP/MPLS in the core network*

### eBGP & OSPF configuration
R101 
```vtysh

conf t

interface lo
	ip address 1.1.0.1/16
	ip address 2.255.1.1/32

interface eth0
	ip address 10.1.12.2/30
interface eth1
	ip address 10.1.2.1/30

exit

# OSPF configuration
router ospf
	router-id 2.255.1.1
	network 1.1.0.0/16 area 0
	network 2.255.0.1/32 area 0
	network 10.1.12.0/30 area 0
exit

# eBGP configuration
router bgp 100

	bgp router-id 2.255.1.1

	network 1.1.0.0/16
	# for R103
	neighbor 2.255.1.3 remote-as 100
	neighbor 2.255.1.3 update-source 2.255.1.1
	
	# for R201 - AS200
	neighbor 10.1.2.2 remote-as 200

address-family ipv4 unicast
	neighbor 2.255.1.3 next-hop-self

address-family ipv4 vpn
	neighbor 2.255.1.3 activate
	neighbor 2.255.1.3 next-hop-self
	
	# for R201 - AS200
	neighbor 10.1.2.2 remote-as 200
exit

# LDP
mpls ldp
router-id 2.255.1.1
ordered-control
address-family ipv4
discovery transport-address 2.255.1.1
interface eth0
interface lo
exit
exit
exit

router bgp 100 vrf vpn200
address-family ipv4 unicast
redistribute static 
label vpn export auto
rd vpn export 200:0
rt vpn import 200:1
rt vpn export 200:2
export vpn
import vpn
exit
exit

```

```Shell
# confgure VRFs
# vpn200 for the AS200
# added to start.sh

ip link add vpn200 type vrf table 200

ip link set vpn200 up

ip link set eth1 master vpn200

# static route
ip route 2.0.0.0/24 10.1.2.2 vrf vpn200

```

```
# write it manually

# /etc/sysctl.conf
net.mpls.conf.lo.input = 1
net.mpls.conf.eth0.input = 1
net.mpls.conf.vpn200.input = 1
net.mpls.platform_labels = 100000

# save and issue “sysctl -p”

```

R102 
```vtysh

conf t

interface lo
	ip address 1.2.0.1/16
	ip address 2.255.1.2/32

interface eth0
	ip address 10.1.12.1/30
interface eth1
	ip address 10.1.23.1/30

exit

# OSPF configuration
router ospf
	router-id 2.255.1.2
	network 1.2.0.0/16 area 0
	network 2.255.0.2/32 area 0
	network 10.1.12.0/30 area 0
	network 10.1.23.0/30 area 0
exit

#LDP
mpls ldp
router-id 2.255.1.2
ordered-control
address-family ipv4
discovery transport-address 2.255.1.2
interface eth0
interface eth1
interface lo

exit
exit
exit

```

```
# write it manually

# /etc/sysctl.conf
net.mpls.conf.lo.input = 1
net.mpls.conf.eth0.input = 1
net.mpls.conf.eth1.input = 1
#net.mpls.conf.vpn200.input = 1 NO
#net.mpls.conf.vpn300.input = 1 NO
net.mpls.platform_labels = 100000

# save and issue “sysctl -p”

```


R103
```vtysh

conf t

interface lo
	ip address 1.3.0.1/16
	ip address 2.255.1.3/32

interface eth0
	ip address 10.1.23.2/30
interface eth1
	ip address 10.1.3.1/30

exit

# OSPF configuration
router ospf
	router-id 2.255.1.3
	network 1.3.0.0/16 area 0
	network 2.255.1.3/32 area 0
	network 10.1.23.0/30 area 0
exit

# eBGP configuration
router bgp 100

	bgp router-id 2.255.1.3
	
	network 1.3.0.0/16
	# for R101
	neighbor 2.255.1.1 remote-as 100
	neighbor 2.255.1.1 update-source 2.255.1.3
	
	# for R301 - AS300
	neighbor 10.1.3.2 remote-as 300

address-family ipv4 unicast
	neighbor 2.255.1.1 next-hop-self

address-family ipv4 vpn
	neighbor 2.255.1.1 activate
	neighbor 2.255.1.1 next-hop-self

exit
exit

# LDP
mpls ldp
router-id 2.255.1.3
ordered-control
address-family ipv4
discovery transport-address 2.255.1.3
interface eth0
interface lo
exit
exit
exit

router bgp 100 vrf vpn300
	address-family ipv4 unicast
	redistribute static #all the static routes redistribute inside the bgp
	label vpn export auto
	rd vpn export 300:0 #route distinguisher
	rt vpn import 300:1 #route target
	rt vpn export 300:2
	export vpn
	import vpn
exit
exit

```

```Shell
# confgure VRFs
# vpn300 for the AS300
# added to start.sh

ip link add vpn300 type vrf table 300

ip link set vpn300 up

ip link set eth1 master vpn300

# static route
ip route add 3.0.0.0/24 via 10.1.3.2 vrf vpn300 # NON SICURO

```

```
# write it manually

# /etc/sysctl.conf
net.mpls.conf.lo.input = 1
net.mpls.conf.eth0.input = 1
net.mpls.conf.vpn300.input = 1
net.mpls.platform_labels = 100000

# save and issue “sysctl -p”

```

## AS200
![[as200.png]]
*AS 200 is a customer AS connected to AS100, which provides transit services.*
- *Setup eBGP peering with AS100*
- *Configure iBGP peering*
- *Configure internal routing as you wish (with or without OSPF)*

R201
```vtysh

conf t

interface lo
	ip address 2.1.0.1/16
	ip address 2.255.2.1/32

interface eth0
	ip address 10.2.12.1/30
interface eth1
	ip address 10.1.2.2/30

# OSPF configuration
router ospf
	router-id 2.255.2.1
	network 2.1.0.0/16 area 0
	network 2.255.2.1/32 area 0
	network 10.2.12.0/30 area 0

# eBGP configuration
router bgp 200
	network 2.1.0.0/16
	# for R202
	neighbor 2.255.2.2 remote-as 200
	neighbor 2.255.2.2 update-source 2.255.2.1
	neighbor 2.255.2.2 next-hop-self
	
	# for R202 - AS100
	neighbor 10.1.2.1 remote-as 100
	

```


R202
```vtysh

conf t

interface lo
	ip address 2.2.0.1/16
	ip address 2.255.2.2/32

interface eth0
	ip address 10.2.12.2/30
interface eth1
	ip address 2.3.0.1/30

# OSPF configuration
router ospf
	router-id 2.255.2.2
	network 2.2.0.0/16 area 0
	network 2.255.2.2/32 area 0
	network 10.2.12.0/30 area 0
	network 2.3.0.0/16 area 0
	
# eBGP configuration
router bgp 200
	network 2.2.0.0/16
	# for R201
	neighbor 2.255.2.1 remote-as 200
	neighbor 2.255.2.1 update-source 2.255.2.2
	neighbor 2.255.2.1 next-hop-self

	network 2.3.0.0/16
	
	

```


-  *R203 is not a BGP speaker*
	- *It has a default route towards R202*
	- *It has a public IP address from the IP address pool of AS200*
	- *It is the Access Gateway for the LAN attached to it*
		- *Configure dynamic NAT*
		- *Configure a simple firewall to allow just connections initiated from the LAN*

R203
```Shell

ip addr add 192.168.0.1/24 dev eth0
ip addr add 2.3.0.2/30 dev eth1 

ip route add default via 2.3.0.1

sysctl -w net.ipv4.ip_forward=1


export LAN=eth0
export NET=eth1

iptables -F
iptables -P FORWARD DROP
iptables -P INPUT DROP
iptables -P OUTPUT DROP

iptables -A FORWARD -i $LAN -o $NET -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED -j ACCEPT

iptables -A POSTROUTING -t nat -o eth1 -j MASQUERADE


```

## Client 200
![[client200.png]]
- *This device is sensitive, so it must be configured to use Mandatory Access Control.*
- *OpenVPN → see later dedicated section.*

```Shell

ip addr add 192.168.0.2/24 dev enp0s8

ip route add default via 192.168.0.1

```

### Apparmor Configuration
Il client-200 è stato configurato per il MAC utilizzando AppArmor. In particolare la configurazione adottata tiene conto di uno scenario di utilizzo immaginato per questo specifico caso. Più in dettaglio, dal client è possibile trasferire dei file verso un server (scripts riportati di seguito) e si è voluto quindi limitare l'accesso di tale applicativo ad una specifica directory (file e sub-directories annesse). Sono state aggiunte anche restrizioni per applicativi potenzialmente pericolosi e abilitate quelle necessarie a compiere il lavoro richiesto.
La configurazione Apparmor:

```
#include <tunables/global>

/usr/bin/client-script {
  # Caratteristiche del programma
  capability chown,
  capability dac_override,
  capability dac_read_search,
  capability fowner,
  capability fsetid,
  capability kill,
  capability setgid,
  capability setuid,
  capability sys_chroot,
  capability setpcap,
  capability net_bind_service,

  # Accesso alla directory 'myhome'
  /path/to/myhome/ r,
  /path/to/myhome/** rwk,

  # Limitazione dell'uso di programmi pericolosi
  /bin/bash ix,
  /bin/sh ix,
  /bin/nc ix,
  deny /usr/bin/python* rm,
  deny /usr/bin/perl rm,
  deny /usr/bin/ruby rm,
  deny /usr/bin/gcc rm,
  deny /usr/bin/make rm,
  deny /usr/bin/wget rm,
  deny /usr/bin/curl rm,
  deny /bin/su rm,
  deny /usr/bin/sudo rm,

  # Negazione di accesso a tutte le altre risorse
  /{,usr/}bin/* rm,
  /{,usr/}sbin/* rm,
  /lib/** rm,
  /lib64/** rm,
  /usr/lib/** rm,
  /usr/lib64/** rm,
  /{,usr/}share/** rm,

  # File di configurazione AppArmor
  /etc/apparmor.d/usr.bin.client-script r,

  # Comandi permessi
  /bin/cat ixr,
  /bin/nc ixr,
}
```

Per **applicare** il nuovo profilo con
```
sudo apparmor_parser -a /usr/bin/client-script
```

Lo stato di ogni profilo può essere scambiato tra la modalità enforce e complain con le chiamate ad `aa-enforce` e `aa-complain` passando come parametro il percorso del file eseguibile oppure il percorso del file delle policy.
Inoltre un profilo può essere completamente disabilitato con `aa-disable` o messo in modalità di controllo (per registrare anche le chiamate di sistema accettate) con `aa-audit`.
```Shell
aa-enforce /usr/sbin/avahi-daemon

aa-complain /etc/apparmor.d/usr.bin.client-script

```

#### Scripts
#### Client
```Shell
#!/bin/bash

SERVER_IP="192.168.1.100"

PORT=12345

echo "Select the file to be sent"
read -e -p "File path: " FILE_PATH

if [[ ! -f "$FILE_PATH" ]]; then
  echo "File doesn't exist"
  exit 1
fi

echo "Sending $FILE_PATH to $SERVER_IP:$PORT"
cat "$FILE_PATH" | nc $SERVER_IP $PORT
echo "File sent"


```

#### Server
```Shell
#!/bin/bash

PORT=12345

RECEIVE_DIR="./received_files"

mkdir -p "$RECEIVE_DIR"

echo "Waiting for the file on the port $PORT..."
nc -l -p $PORT > "$RECEIVE_DIR/received_file"
echo "File saved in $RECEIVE_DIR/received_file"

```

## AS300
![[as300.png]]
*AS 300 is a customer AS connected to AS 100, which provides transit services. It also has a*
*lateral peering relationship with AS 400.*
- *Setup eBGP peering with AS400 and AS100*
- *Configure iBGP peering*
- *Configure internal routing as you wish (with or without OSPF)*


R301
```vtysh

conf t

interface lo
	ip address 3.1.0.1/16
	ip address 2.255.3.1/32

interface eth0
	ip address 10.3.12.1/30
interface eth1
	ip address 10.1.3.2/30

# OSPF configuration
router ospf
	router-id 2.255.3.1
	network 3.1.0.0/16 area 0
	network 2.255.3.1/32 area 0
	network 10.3.12.0/30 area 0

# eBGP configuration
router bgp 300
	network 3.1.0.0/16
	# for R302
	neighbor 2.255.3.2 remote-as 300
	neighbor 2.255.3.2 update-source 2.255.3.1
	neighbor 2.255.3.2 next-hop-self
	
	# for R103 - AS100
	neighbor 10.1.3.1 remote-as 100
	

```


R302
```vtysh


conf t

interface lo
	ip address 3.2.0.1/16
	ip address 2.255.3.2/32

interface eth0
	ip address 10.3.12.2/30
interface eth1
	ip address 10.3.4.1/30
interface eth2
	ip address 3.3.0.1/30

# OSPF configuration
router ospf
	router-id 2.255.3.2
	network 3.2.0.0/16 area 0
	network 2.255.3.2/32 area 0
	network 10.3.12.0/30 area 0
	network 3.3.0.0/16 area 0

# eBGP configuration
router bgp 300
	network 3.2.0.0/16
	# for R302
	neighbor 2.255.3.1 remote-as 300
	neighbor 2.255.3.1 update-source 2.255.3.2
	neighbor 2.255.3.1 next-hop-self
	
	# for R401 - AS400
	neighbor 10.3.4.2 remote-as 400

	network 3.3.0.0/16
	

```


-  *GW300 is not a BGP speaker*
	- *It has a default route towards R302*
	- *It has a public IP address from the IP address pool of AS300*
	-  *It is the Access Gateway for the Data Center network attached to it*
		- *Configure dynamic NAT*
		- *OpenVPN server → see later dedicated section*

GW300:
```Shell

ip addr add 3.3.0.2/16 dev eth0
ip addr add 10.1.5.2/30 dev eth1

ip route add default via 3.3.0.2

# for external communication
ip link add link eth1 name eth1.10 type vlan id 10
ip link add link eth1 name eth1.20 type vlan id 20

ip addr add 10.10.10.1/16 dev eth1.10
ip addr add 10.10.10.1/16 dev eth1.20

ip link set eth1.10 up
ip link set eth1.20 up

ip route add 10.0.0.1 via 10.10.10.254 dev eth1.10
ip route add 10.1.1.1 via 10.10.10.254 dev eth1.20

sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

```
## DC Network

![[dcnet.png]]
*DC Network is a leaf-spine Data Center network with two leaves and two spines. There are 2 tenants (A and B) in the cloud network, each hosting two virtual machines connected to leaf1 and leaf2. The tenants are assigned one broadcast domain each.*
- *Realize VXLAN/EVPN forwarding in the DC network to provide L2VPNs between the tenants’ machines*
- *In L1, enable the connectivity to the external network. In other words, both tenants’ machines must reach the external network through the link between L1 and R303, including the encapsulation in OpenVPN tunnels when necessary.* 

A1:
```Shell

ip addr add 10.0.0.1/24 dev eth0
ip route add default via 10.0.0.254

```

B1:
```Shell

ip addr add 10.1.1.1/24 dev eth0
ip route add default via 10.1.1.254

```

A2:
```Shell

ip addr add 10.0.0.2/24 dev eth0
ip route add default via 10.0.0.254

```

B2:
```Shell

ip addr add 10.1.1.2/24 dev eth0
ip route add default via 10.1.1.254

```


Per L1:
```Shell

# creiamo un bridge (come nel lab di 802.1x)
net add bridge bridge ports swp3,swp4

net add bridge bridge vids 10,20,30,200

net add interface swp3 bridge access 10
net add interface swp4 bridge access 20
net commit

#assegnamo gli ip point-to-point alle interfacce
net add interface swp1 ip add 10.1.1.1/30
net add interface swp2 ip add 10.1.2.1/30
net add interface swp5 ip add 10.1.5.1/30
net add loopback lo ip add 1.1.1.1/32
net commit

#####

net add ospf router-id 1.1.1.1
net add ospf network 10.1.1.0/30 area 0
net add ospf network 10.1.2.0/30 area 0
net add ospf network 10.1.5.0/30 area 0 #forse non serve
net add ospf network 1.1.1.1/32 area 0
net add ospf passive-interface swp3,swp4

net commit

#####

net add vxlan vni100 vxlan id 100
#net add vxlan vni100 vxlan remoteip 2.2.2.2
net add vxlan vni100 vxlan local-tunnelip 1.1.1.1
net add vxlan vni100 bridge access 10 # associamo vni100 (l'interfaccia) conla VLAN 10, dicendo che ha un'access port con VLAN 100 nel bridge

net add vxlan vni200 vxlan id 200
#net add vxlan vni200 vxlan remoteip 2.2.2.2
net add vxlan vni200 vxlan local-tunnelip 1.1.1.1
net add vxlan vni200 bridge access 20


# MP-eBGP
net add bgp autonomous-system 65001
net add bgp router-id 1.1.1.1
net add bgp neighbor swp1 remote-as 65000
net add bgp neighbor swp2 remote-as 65000
net add bgp evpn neighbor swp1 activate
net add bgp evpn neighbor swp2 activate
net add bgp evpn advertise-all-vni

# add IP address in VTEPs
net add vlan 10 ip address 10.0.0.254/24
net add vlan 20 ip address 10.1.1.254/24


# Add a new VXLAN interface for L3VNI
net add vlan 50
net add vxlan vni-1020 vxlan id 1020
net add vxlan vni-1020 vxlan local-tunnelip 1.1.1.1
net add vxlan vni-1020 bridge access 50

# Bridge VIDs and VNIs and create tenant VRF
net add vrf TEN1 vni 1020
net add vlan 50 vrf TEN1
net add vlan 10 vrf TEN1
net add vlan 20 vrf TEN1

net add bgp vrf TEN1 autonomous-system 65001
net add bgp vrf TEN1 l2vpn evpn advertise ipv4 unicast
net add bgp vrf TEN1 l2vpn evpn default-originate ipv4

net commit

```

Per L2:
```Shell

# creiamo un bridge (come nel lab di 802.1x)
net add bridge bridge ports swp3,swp4
net add interface swp3 bridge access 10
net add interface swp4 bridge access 20
net commit

net add interface swp1 ip add 10.2.1.1/30
net add interface swp2 ip add 10.2.2.1/30
net add loopback lo ip add 2.2.2.2/32
net commit

#####

net add ospf router-id 2.2.2.2
net add ospf network 10.2.1.0/30 area 0
net add ospf network 10.2.2.0/30 area 0
net add ospf network 2.2.2.2/32 area 0
net add ospf passive-interface swp3,swp4

net commit


net add vxlan vni100 vxlan id 100
#net add vxlan vni100 vxlan remoteip 1.1.1.1
net add vxlan vni100 vxlan local-tunnelip 2.2.2.2
net add vxlan vni100 bridge access 10

net add vxlan vni200 vxlan id 200
#net add vxlan vni200 vxlan remoteip 1.1.1.1
net add vxlan vni200 vxlan local-tunnelip 2.2.2.2
net add vxlan vni200 bridge access 20


# MP-eBGP
net add bgp autonomous-system 65002
net add bgp router-id 2.2.2.2
net add bgp neighbor swp1 remote-as 65000
net add bgp neighbor swp2 remote-as 65000
net add bgp evpn neighbor swp1 activate
net add bgp evpn neighbor swp2 activate
net add bgp evpn advertise-all-vni

# add IP address in VTEPs
net add vlan 10 ip address 10.0.0.254/24
net add vlan 20 ip address 10.1.1.254/24


# Add a new VXLAN interface for L3VNI
net add vlan 50
net add vxlan vni-1020 vxlan id 1020
net add vxlan vni-1020 vxlan local-tunnelip 2.2.2.2
net add vxlan vni-1020 bridge access 50

# Bridge VIDs and VNIs and create tenant VRF
net add vrf TEN1 vni 1020
net add vlan 50 vrf TEN1
net add vlan 10 vrf TEN1
net add vlan 20 vrf TEN1


net commit

```


GW300 (stessa del paragrafo precedente, riportata per completezza):
```Shell

ip addr add 3.3.0.2/16 dev eth0
ip addr add 10.1.5.2/30 dev eth1

ip route add default via 3.3.0.2

# for external communication
ip link add link eth1 name eth1.10 type vlan id 10
ip link add link eth1 name eth1.20 type vlan id 20

ip addr add 10.10.10.1/16 dev eth1.10
ip addr add 10.10.10.1/16 dev eth1.20

ip link set eth1.10 up
ip link set eth1.20 up

ip route add 10.0.0.1 via 10.10.10.254 dev eth1.10
ip route add 10.1.1.1 via 10.10.10.254 dev eth1.20

sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

```


Spine1:
```Shell

net add interface swp1 ip add 10.1.1.2/30
net add interface swp2 ip add 10.2.1.2/30
net add loopback lo ip add 3.3.3.3/32

net commit


net add ospf router-id 3.3.3.3
net add ospf network 0.0.0.0/0 area 0


# MP-eBGP
net add bgp autonomous-system 65000
net add bgp router-id 3.3.3.3
net add bgp neighbor swp1 remote-as external
net add bgp neighbor swp2 remote-as external
net add bgp evpn neighbor swp1 activate
net add bgp evpn neighbor swp2 activate
net add bgp evpn advertise-all-vni


net commit

```


Spine2:
```Shell

net add interface swp1 ip add 10.1.2.2/30
net add interface swp2 ip add 10.2.2.2/30
net add loopback lo ip add 4.4.4.4/32

net commit

net add ospf router-id 4.4.4.4
net add ospf network 0.0.0.0/0 area 0


# MP-eBGP
net add bgp autonomous-system 65000
net add bgp router-id 4.4.4.4
net add bgp neighbor swp1 remote-as external
net add bgp neighbor swp2 remote-as external
net add bgp evpn neighbor swp1 activate
net add bgp evpn neighbor swp2 activate
net add bgp evpn advertise-all-vni


net commit

```

## AS400
![[as400.png]]
*AS 400 has a lateral peering relationship with AS 300.*
- *Setup eBGP peering with AS400 and AS100*


R401
```vtysh

conf t

interface lo
	ip address 4.1.0.1/16
	ip address 2.255.4.1/32

interface eth0
	ip address 4.2.0.1/30
interface eth1
	ip address 10.3.4.2/30


# eBGP configuration
router bgp 400
	network 4.1.0.0/16
	network 4.2.0.0/16
	# for R302 - AS300
	neighbor 10.3.4.1 remote-as 300
	

```


-  *R402 is not a BGP speaker*
	- *It has a default route towards R401*
	- *It has a public IP address from the IP address pool of AS400*
	- *It is the Access Gateway for the LAN attached to it*
		- *Configure dynamic NAT*
		- *Configure OpenVPN→ see later dedicated section*

R402
```Shell

ip addr add 4.2.0.2/30 dev eth0
ip addr add 192.168.1.1/24 dev eth1

ip route add default via 4.2.0.1

sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

```

## Client 400
![[client400.png]]
- *This is a simple LAN device with a default route through R402.*

```Shell

ip addr add 192.168.1.0/24 dev eth0

ip route add default via 192.168.1.1

```


## OPENVPN
*Setup OpenVPN to realize a VPN between client-200, R402’s LAN, and the DataCenter*
*network.*
- **ovpn-client1**: *Client-200 is an OpenVPN client*
- **ovpn-client2**: *R402 is an OpenVPN client, providing VPN access to and from the LAN attached to it.*
- **ovpn-server**: *GW300 is the OpenVPN server, providing VPN access to and from the Data Center network. In particular, the network belonging to tenant A must be accessible through the VPN.*

GW300 (a.k.a. ovpn-server)
```Shell

./easyrsa init-pki
./easyrsa build-ca nopass # this command build the certificate authority (CA) certificate and key by invoking the interactive openssl command. We have to give only the 'Common Name' (es. OVPN_NSD)

cd pki # here we have the root CA

# server key
./easyrsa build-server-full server nopass

# clients key
./easyrsa build-client-full client1 nopass # for client 1
./easyrsa build-client-full client2 nopass # for client 2

./easyrsa gen-dh # Diffie Hellman parameters must be generated for the OpenVPN server



mkdir /root/CA
mkdir /root/CA/server
mkdir /root/CA/client1
mkdir /root/CA/client2

# for the server we need
cp /usr/share/easy-rsa/pki/ca.crt /root/CA/
cp /usr/share/easy-rsa/pki/issued/server.crt /root/CA/server/
cp /usr/share/easy-rsa/pki/private/server.key /root/CA/server/
cp /usr/share/easy-rsa/pki/dh.pem /root/CA/server

# for client1
cp /usr/share/easy-rsa/pki/issued/client1.crt /root/CA/client1
cp /usr/share/easy-rsa/pki/private/client1.key /root/CA/client1

# for client2
cp /usr/share/easy-rsa/pki/issued/client2.crt /root/CA/client2
cp /usr/share/easy-rsa/pki/private/client2.key /root/CA/client2


# now we have to distribute the crypto material (keys, cert) to the clients
cd /CA
cat ca.crt



ip addr add 1.0.0.2/24 dev eth0
ip route add default via 1.0.0.1

echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE


```


Client-200 (a.k.a. ovpn-client1)
```Shell

mkdir ovpn
cd ovpn
vim ca.crt # paste the ca.crt taken from the server

vim client1.crt # paste the certificate (only the base64 format frame including the BEGIN and START)
vim client1.key # paste the private key of the client*

```


R402 (a.k.a. ovpn-client2)
```Shell

mkdir ovpn
cd ovpn
vim ca.crt # paste the ca.crt taken from the server

vim client2.crt # paste the certificate (only the base64 format frame including the BEGIN and START)
vim client2.key # paste the private key of the client*

```



GW300
```conf
# in /CA/server/server.ovpn

port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 192.168.100.0 255.255.255.0
# routes for dst outside the VPN
push "route 192.168.1.0 255.255.255.0" # LAN with the client-400 (1)
push "route 10.0.0.0 255.255.255.0" # DC Network (2)
route 192.168.1.0 255.255.255.0
client-config-dir ccd
client-to-client
keepalive 10 120
cipher AES-256-GCM


```
> N.B. (1) non è sufficiente poichè per raggiungere 192.168.1.0/24 bisogna passare per R402 (ovpn-client2) ed il tunnel per esso non è noto a priori. Quindi abbiamo bisogno che per l' ovpn-client2 si specifichi tale rotta in `ccd/client2`.

```
#in file `/CA/server/ccd/client2`
iroute 192.168.1.0 255.255.255.0

# as soon as client 2 connects, the VPN
# server will understand that
# 192.168.1.0/24 is reachable through the
# tunnel established with client2. This
# will result in the overlay routing rule:
# 192.168.1.0/24 --> client2

```
> Per quanto riguarda invece 10.0.0.0/24 non c'è bisogno poichè è direttamente collegata al ovpn-server (GW300).


Per eseguire lo start del servizio
```Shell
# start the service
openvpn server.ovpn &

```


Client-200
```conf
# in /ovpn/client1.ovpn

client
dev tun
proto udp
remote 3.3.0.2 1194 # GW300
resolv-retry infinite
ca ca.crt
cert client1.crt
key client1.key
remote-cert-tls server
cipher AES-256-GCM

```

R402
```conf
# in /ovpn/client1.ovpn

client
dev tun
proto udp
remote 3.3.0.2 1194 # GW300
resolv-retry infinite
ca ca.crt
cert client2.crt
key client2.key
remote-cert-tls server
cipher AES-256-GCM

```

