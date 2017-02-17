Tinc
====

[![Build Status](https://travis-ci.org/evrardjp/ansible-tinc.svg?branch=master)](https://travis-ci.org/evrardjp/ansible-tinc)

This role installs tinc in a star or a ring topology.

The nodes part of the group tinc_node is a full list of nodes to apply/install the role.

The nodes part of the tinc_spine_nodes are the "core" nodes, where all the nodes connect.
If all the tinc_nodes are part of the tinc_spine_nodes, you have a more "ringy" topology.
If you have one node in tinc_spine_nodes, you have a more "starry" topology.

Requirements
------------

* Ubuntu 16.04 /  CentOS 7 (or above)

Role Variables
--------------

* tinc_key_size: The size of the generated keys (Default 4096)
* tinc_address_family can be ipv4/ipv6/any (or undefined)
* tinc_mode can be router, switch, or hub. (See https://www.tinc-vpn.org/documentation/tinc.conf.5). Default: Router
* tinc_netname: The tinc network name
* tinc_vpn_cidr: The cidr used in tinc.

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
