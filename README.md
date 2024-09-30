# Juniper-Routers-with-Arbor
The Arbor Networks SP product provides a reporting and visibility tool to detect DDoS attacks, profile network traffic, and monitor network path utilization. In order to provide these reports the product relies on getting information from the managed routers using the following
protocols to collect data:

•	SNMP polling for interface information  

•	Jflow for sampled information on all interfaces

•	BGP peering to obtain route table data and monitor table changes

# Problem

Typically, Arbor Networks SP is deployed in a service provider or large enterprise network. The monitored routers need to be at the edge where Internet traffic ingresses to the network, providing the first point of access to detect anomalies in traffic and the best opportunity to deal with malicious traffic entering the network.
When the network has multiple upstream Internet peers, all MX Series routers involved in the various peerings need to be included in the monitoring solution to insure that all possible sources are visible for DDoS evaluations. And for traffic and transit management, all points of Internet ingress and egress are needed. Integrating the SP device into the network to collect all three types of data from all the MX Series routers can be tricky and should be first done in your lab before being used in production environments.

# Solution
Arbor Networks SP is deployed into the network using two network interfaces: one provides the SNMP, BGP, and web interface address, and the second interface is used for flow collection. The MX Series uses the loopback address for the BGP peer and SNMP collection and the fxp0 management address for the flow.
The MX Series configuration consists of three parts: the SNMP, BGP, and Jflow configurations.
For SNMP you simply need to add the Arbor SP IP address as an allowed host for the SNMP community on the MX Series. This is used to pull interface statistics from the MX to the Arbor SP appliance. The String value is the SNMP community string that Arbor SP uses when collecting data from the MX:

Test@# set snmp community String clients 191.1.3.24/32

Test@# set snmp community String authorization read-only

The next configuration has an iBGP to the MX loopback address from the Arbor SP eth0 interface. These peerings are set up as standard iBGP internal peers using the local AS with the default loopback addresses. The idea is to integrate the Arbor appliances into the upstream peerings:

Test@# set protocols bgp group Arbor type internal

Test@# set protocols bgp group Arbor family inet unicast

Test@# set protocols bgp group Arbor peer-as 64496

Test@# set protocols bgp group Arbor neighbor 191.1.3.24 local-address 127.0.0.1

Test@# set forwarding-options sampling input rate 1000

Test@# set forwarding-options sampling family inet output flowserver

172.22.1.32 port 2055

Test@# set forwarding-options sampling family inet output flow-server 172.22.1.32 sourceaddress
172.22.1.33 

Test@# set forwarding-options sampling family inet output flowserver 172.22.1.32version 5

Test@# set groups jflow_subinterfaces interfaces <*-*> unit <*> family inet sampling input

Test@# set groups jflow_subinterfaces interfaces <*-*> unit <*> family inet sampling output

root@# set apply-groups jflow_subinterfaces




# ARBOR SP Configuration:
To add the MX Series as a managed router, use the Arbor SP installation and also add other SNMP configuration for new router.
GUI interface, and follow this GUI path: Administration >
Monitoring > Routers > Add Router button.


On the MX Series, SNMP get statistics should be incrementing. Run the show snmp command statistics several times and confirm the incrementing
get responses:


Test@> show log messages | match 191.1.3.24 

Feb 22 23:12:11 MX snmpd[3124]: SNMPD_AUTH_FAILURE: nsa_log_

community: unauthorized SNMP community from 191.1.3.24 to 127.0.0.1 (String)

Get responses: 142392142392, Traps: 850995

Confirm the BGP peer is up and established on the MX:

root > show bgp neighbor 191.1.3.24 

Peer: 193.0.2.11+3038 AS 33154 Local: 127.0.0.1+179 AS 64496

Type: Internal State: Established Flags: <Sync RSync>

Last State: EstabSync Last Event: RecvKeepAlive

Last Error: Hold Timer Expired Error

Options: <Preference LocalAddress PeerAS Refresh>

Local Address: 127.0.0.1 Holdtime: 90 Preference: 170

Number of flaps: 14

Last flap event: Closed

Error: ‘Hold Timer Expired Error’ Sent: 4 Recv: 1

Peer ID: 191.1.3.24Local ID: 127.0.0.1 Active Holdtime: 90

Keepalive Interval: 30 Group index: 16 Peer index: 0

BFD: disabled, down

NLRI for restart configured on peer: inet-unicast

NLRI advertised by peer: inet-unicast inet-flow

NLRI for this session: inet-unicast

Peer does not support Refresh capability

Stale routes from peer are kept for: 300

Peer does not support Restarter functionality

Peer does not support Receiver functionality

Peer supports 4 byte AS extension (peer-as 33154)

Peer does not support Addpath

Table inet.0 Bit: 20007

RIB State: BGP restart is complete

Send state: in sync

Active prefixes: 0

Received prefixes: 0

Accepted prefixes: 0

Suppressed due to damping: 0

Advertised prefixes: 620403




