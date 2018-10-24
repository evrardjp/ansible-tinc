Tinc
====

[![Build Status](https://travis-ci.org/evrardjp/ansible-tinc.svg?branch=master)](https://travis-ci.org/evrardjp/ansible-tinc)

This role installs tinc in a star or a ring topology.

The nodes listed in the group [tinc_nodes] is a full list of nodes to apply/install the role.

The nodes part of [tinc_spine_nodes] are the "core" nodes, where all the nodes connect.

The nodes in [tinc_leaf_nodes] connect only to the spine nodes.  Devices behind a NAT would be an example of such.

If all the [tinc_nodes] are part of the [tinc_spine_nodes], you have a more "ringy" topology. If you have one node in [tinc_spine_nodes], you have a more "starry" topology.

Requirements
------------

* Ubuntu 16.04 / CentOS 7 (or above) / OpenWRT

Role Variables
--------------

* tinc_key_size: The size of the generated keys (Default: 4096)
* tinc_address_family can be ipv4/ipv6/any (or undefined)
* tinc_mode can be router, switch, or hub. (See https://www.tinc-vpn.org/documentation/tinc.conf.5). (Default: router)
* tinc_netname: The tinc network name
* tinc_vpn_ip: The ip/subnet to assign to a host using host_vars
* tinc_vpn_cidr: The cidr used in tinc (Default: /24, or force /32 in router mode).
* tinc_vpn_interface: The device for tinc to use (Default: tun0)
* tinc_control_plane_bind_ip: The ip for tinc to bind to (Example: {{ ansible_host }} )

Inventory should have an extra tinc_control_plane_bind_ip or tinc_vpn_ip,
depending on the modes. Please have a look in the task files.

For example, the tinc_control_plane_bind_ip could be set to
`{{ ansible_host }}` or something like {{ ansible_eth0.ipv4.address }}.
Please have a look at your ansible facts and define them appropriately.

Dependencies
------------

None

Example Playbook
----------------

    - hosts: tinc_nodes
      roles:
         - evrardjp.tinc

For this playbook to work, you should include your topology somewhere.
Here is an example of my group_vars/ and inventory/

Group vars:

    tinc_netname: mynetname
    tinc_mode: switch
    tinc_vpn_cidr: "/24"
    tinc_vpn_interface: tun0
    tinc_control_plane_bind_ip: "{{ ansible_host }}"

Host vars:

    tinc_vpn_ip: 10.10.0.1

Inventory:

```

[tinc_nodes]
node1
node2

[tinc_spine_nodes]
node1
```


License
-------

Apache2

Author Information
------------------

Jean-Philippe Evrard <jean-philippe at evrard dot me>
