## oVirt prerequisite for Openshift 4.5.x install

This sets of playbooks will setup oVirt networking and a Bastion vm for running OCP 4.8.x IPI installer

# Features

- Vlan creation (for OCP internal network)
- Auto template creation for vm (centos 8 cloud)
- Interface and connection name are autodetected
- Bastion vm creation and configuration (using cloud-init)
- Ip reservation for OCP
- Dhcp server for internal network
- Haproxy configuration (listening on bastion public interface)
- Dns listening on the internal network

The bastion vm services are:
- default gateway for openshift nodes connected to the internal network
- dhcp server
- dns server
- haproxy (both api and application)

# Name resolutions

The workstation must be able to resolve the bastion vm fqdn for api.<cluster_name>.<domain> and *.apps.<cluster_name>.<domain>
The bastion vm must be able to resolve the oVirt engine fqdn


# Sample vars config

file: groups_vars/all/vars.yaml

```
networking:
  internal_network: 172.22.0.0
  internal_network_ip: 172.22.0.1
  internal_network_netmask: 255.255.255.0
  external_dns: 192.168.1.77
  domain_name: mylab.com

dhcp:
  timezone: "Europe/Rome"
  start: 172.22.0.201
  end: 172.22.0.250
  ntp: 204.11.201.10

bastion:
  name: bastion
  public_ip: 192.168.1.101
  public_netmask: 255.255.255.0
  public_gateway: 192.168.1.1

firewall_services:
  - http
  - dns
  - dhcp

firewall_public_ha_proxy_port:
  - 80/tcp
  - 443/tcp
  - 6443/tcp

ocp:
  reserved_ip:
    api: 172.22.0.2 # Asked during OCP install
    apps: 172.22.0.3 # Asked during OCP install
    internal_dns: 172.22.0.4 # Asked during OCP install
  cluster_name: ocp
  

```

file: groups_vars/all/ovirt.yaml

```
ovirt:
  username: admin@internal
  ca_file: /etc/pki/ovirt-engine/ca.pem
  password: XXX
  data_center: Default
  cluster: Default
  host: host.mylab.com
  engine: manager.mylab.com
  storage:
     interface: virtio
     storage_domain: hosted_storage
  network:
    external:
       interface: virtio
       profile: ovirtmgmt
       network: ovirtmgmt
    internal:
       interface: virtio
       profile: ocp
       network: ocp
       host_interface: em2 #Insert oVirt host interface name

```


# Required python packages and version

- ansible==2.9.6

# Required roles:

- ovirt.image-template
- ovirt.vm-infra

# Ansible collection 

- ansible-galaxy collection install ovirt.ovirt

# The bastion vm is tested with Centos 8 Cloud #

- Get an openshift pullSecret from https://cloud.redhat.com/openshift/install  -> Run on RHV
- Set the pullSecret in group_vars/vars.yaml

The interface and connection name are autodetected

- run $ansible-playbook connected_ipi_install.yaml







