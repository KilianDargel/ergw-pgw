A containerized PDN-GW based on erGW.
========================

## Introduction

This project provides a helm chart setup to deploy and run a containerized (3GPP GGSN) PDN-GW based on [erGW](https://github.com/travelping/ergw).
Additionally an SGSN emulator is included to test against the GGSN in the kubernetes cluster, which is based on [OsmoGGSN](https://osmocom.org/projects/openggsn/wiki/OsmoGGSN).

## Network topology

A demo apn implements the following network topology, requiring a Data-, User-plane and Connectivity Gateway.
![Demoapn](https://github.com/KilianDargel/ergw-pgw/blob/master/images/demoapn.png)
This is the full list of containers:
- ergw-c-Deployment.yaml
  - vxlan-controller-agent-init
  - network-setup
  - network-init
  - vxlan-controller-agent
  - ergw-gtp-c-node

- ergw-vpp-deployment.yaml
  - vxlan-controller-agent-init
  - network-setup
  - network-init
  - vxlan-controller-agent
  - vpp-u-node

- ergw-cgw-deployment.yaml
  - vxlan-controller-agent-init
  - network-init
  - standby
  - vxlan-controller-agent
  - ping-prober

- sgsnemu-deployment.yaml
    - vxlan-controller-agent-init
    - network-init
    - vxlan-controller-agent
    - sgsnemu
    - iperf-probe

## Values.yaml

The overrideable values, their default and description are listed in the table below:

| NIC                    | Description                                 |
| ---------------------- | ------------------------------------------- |
|      grx: grx0         | grx interface of the apn.                   |
|      sxb: sxb0         | sxb interface of the apn.                   |
|      sgi: sgi0         | sgi interface of the apn.                   |
|      inet: inet0       | inet interface of the apn.                  |

| VPP                    | Description                                 |
| ---------------------- | ------------------------------------------- |
| grxGW: 172.20.10.1     | IP of grx Gateway NIC                       |
| grx: 172.20.10.60/24   | IP of grx NIC                               |
| CGWsgi: 172.20.11.1    | IP of grx Gateway NIC                       |
| sgi: 172.22.16.2/24    | IP of cgi NIC                               |
| sgiIPv6: fd71:357a:a82e:1 | IPv6 of cgi NIC                          |
| sgiIPv6Network: fd71:357a:a82e:1::2/64 | IPv6 of cgi NIC             |
| sxb: 172.21.13.205/28 | IP of sxb NIC                                |
| CGWcgi: 172.22.16.1 | IP of cgi NIC (optional)                       |

| Cnode                  | Description                                 |
| ---------------------- | ------------------------------------------- |
| grxGW: 172.20.10.1     | IP of grx Gateway NIC                       |
| grx: 172.20.10.153/24  | IP of grx NIC                               |
| sxb: 172.21.13.204/28  | IP of sxb NIC                               |
| plmnID: <<"232">>,<<"03">> | Public Land Mobile Network              |
| lager.lagerConsoleBackendLevel: error | For debug/logging            |

| CGWnode                | Description                                 |
| ---------------------- | ------------------------------------------- |
| cgi: 172.22.16.1/24    | IP of sgi NIC                               |
| K8sRoute: 10.10.10.0/24 | Kubernetes internal network.               |
| sgiIPv6: "fd71:357a:a82e:" | IPv6 of sgi NI                          |
| ExternalRoute: 172.23.10.17/27 | <missing>  |
| defaultGW: 172.23.10.1 | <missing>  |
| inetRouteDefault: 172.23.10.1   | <missing>  |
| defaultGW1: 169.254.1.1   | <missing>  |
| IPv6CGW: "2a07:db01:"   | <missing>  |

| sgsnemu                | Description                                 |
| ---------------------- | ------------------------------------------- |
| automode: true         | PDP context will be created on startup.     |
| listenIP: 172.20.10.155| Local IP of the grx VXLan interface.        |
| remoteIP: 172.20.10.153| Remote IP of the Cnode grx VXLan interface. |

## Testing PDP context activation with sgsnemu
Type in `sgsnemu --help` while attached to the sgsnemu-pod to get more info on operating it. The only required fields for a Echo Request are `--remote=<CNode-grx-IP>` and `--listen=<Sgsnemu-grx-IP>`.
Running the command `sgsnemu --remote=172.20.10.153 --listen=172.20.10.155` should result successfully initiating the GTP library:
```
Using default DNS server
Local IP address is:   172.20.10.155 (172.20.10.155)
Remote IP address is:  172.20.10.153 (172.20.10.153)
IMSI is:               240010123456789 (0xf987654321010042)
Using NSAPI:           0
Using GTP version:     1
Using APN:             internet
Using selection mode:  1
Using MSISDN:          46702123456

Initialising GTP library
<000d> gtp.c:757 GTP: gtp_newgsn() started at 172.20.10.155
<000d> gtp.c:714 State information file (.//gsn_restart) not found. Creating new file.
Done initialising GTP library

Sending off echo request
Setting up PDP context #0
Waiting for response from ggsn........

Received echo response
Received create PDP context response. IP address: 10.10.155.169
```
To open and send all throughput via tun interface `--createif` is required inside a privileged pod. `--conf=/path/` enables to read PDP parameters from a configuration file. A default file sgsn-test.conf is described in a [configmap](https://github.com/KilianDargel/ergw-pgw/blob/master/templates/sgsnemu-configmap.yaml) and mounted to `/mnt/`.
Running the command `sgsnemu --remote=172.20.10.153 --listen=172.20.10.155 --conf=/mnt/sgsn-test.conf --createif -t v4 --defaultroute` should result in successfully initiating the GTP library, updating the session parameters and setup of the `tun0` interface:
```
Using default DNS server
Local IP address is:   172.20.10.155 (172.20.10.155)
Remote IP address is:  172.20.10.153 (172.20.10.153)
IMSI is:               240010123456789 (0xf987654321010042)
Using NSAPI:           5
Using GTP version:     1
Using APN:             travelping
Using selection mode:  1
Using RAI:  025.099.46241.207
->mcc : 025
->mnc : 099
->LAC: 46241
->RAC : 207
Using MS Time Zone:  0
->Sign (0=+ / 1=-): 0
->Number of Quarters of an Hour : 0
->Daylight Saving Time Adjustment : 0
->Human Readable MS Time Zone  : GMT + 0 hours 0 minutes
Using MSISDN:          46702123456

Initialising GTP library
<000d> gtp.c:757 GTP: gtp_newgsn() started at 172.20.10.155
Setting up interface
Done initialising GTP library

Sending off echo request
Setting up PDP context #0
Waiting for response from ggsn........

Received echo response
Received create PDP context response. IP address: 10.10.155.169

/ # ip addr show tun0
8: tun0: <POINTOPOINT,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 500
    link/[65534]
    inet 10.10.155.169/32 scope global tun0
       valid_lft forever preferred_lft forever
```

## Testing throughput with iperf.
The tun interface is only available while sgsnemu is running.  Throughput can be tested on the VXLan interfaces using iperf then. `iperf3` requires both a client as well as a server somewhere in the cluster. Containers based on [travelping/nettools](https://github.com/travelping/docker-nettools) can be used for this. Two containers, that are deployed through helm right away are:
- `iperf-probe` in the sgsnemu-pod can run `iperf3 -c 172.22.16.1 -M 1320 -V` and
- `standby` in the cgw-pod can run `iperf3 -d -s -B 172.22.16.1` and listen on its sgi0 VXLan address.

Make sure that the IP of the iperf server is reachable through the GTP tunnel. So keep in mind that the sgsnemu-container should be actively handling the context before you are trying to reach something from the iperf-probe-container.
Sgsnemu will install a default route through the tun interface, which takes care of the interface selection for iperf. Client and server mode can be switched around at will.
