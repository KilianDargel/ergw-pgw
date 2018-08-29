A containerized PDN-GW based on erGW.
========================

## Introduction

This project provides a helm chart setup to deploy and run a containerized (3GPP GGSN) PDN-GW based on [erGW](https://github.com/travelping/ergw).
Additionally an SGSN emulator is included to test against the GGSN in the kubernetes cluster, which is based on [OsmoGGSN](https://osmocom.org/projects/openggsn/wiki/OsmoGGSN).

## Network topology

A demo apn implements the following network topology, requiring Data- + User-plane and Connectivity Gateway.
![Demoapn](images/demo apn.png)
This is the full list of containers:
1. ergw-c-Deployment.yaml
  1.1. vxlan-controller-agent-init
  1.1. network-setup
  1.1. network-init
  1.2. vxlan-controller-agent
  1.2. ergw-gtp-c-node

2. ergw-vpp-deployment.yaml
  2.1. vxlan-controller-agent-init
  2.1. network-setup
  2.1. network-init
  2.2. vxlan-controller-agent
  2.2. vpp-u-node

3. ergw-cgw-deployment.yaml
  3.1. vxlan-controller-agent-init
  3.1. network-init
  3.2. standby
  3.2. vxlan-controller-agent
  3.2. ping-prober

4. sgsnemu-deployment.yaml
    4.1. vxlan-controller-agent-init
    4.1. network-init
    4.2. vxlan-controller-agent
    4.2. sgsnemu

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
