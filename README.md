# Tinc

![Daily branch test status](https://github.com/evrardjp/ansible-tinc/actions/workflows/daily.yml/badge.svg)

This role installs tinc in a star or a ring topology.

The nodes listed in the group [tinc_nodes] is a full list of nodes to apply/install the role.

The nodes part of [tinc_spine_nodes] are the "core" nodes, where all the nodes connect.

The nodes in [tinc_leaf_nodes] connect only to the spine nodes.  Devices behind a NAT would be an example of such.

If all the [tinc_nodes] are part of the [tinc_spine_nodes], you have a more "ringy" topology. If you have one node in [tinc_spine_nodes], you have a more "starry" topology.

## Requirements

* Ubuntu 18.04 / CentOS 7 (or above) / OpenWRT
* On CentOS and above, EPEL repo needs to be configured in advance.

To do so, you can run the following:
```bash
yum install epel-release || dnf install epel-release
yum update || dnf update
```

## Role Variables

* tinc_key_size: The size of the generated keys (Default: `4096`)
* tinc_address_family can be ipv4/ipv6/any (or undefined)
* tinc_mode can be router, switch, or hub. (See https://www.tinc-vpn.org/documentation/tinc.conf.5). (Default: `router`)
* tinc_netname: The tinc network name
* tinc_vpn_ip: The ip to assign to a single VPN endpoint. Use host vars to set it.
* tinc_vpn_cidr: The cidr used in tinc network (Default: `/24`, or force /32 in router mode).
* tinc_vpn_interface: The device for tinc to use, in case there are multiple tun devices (Default: `tun0`)
* tinc_control_plane_bind_ip: The ip for tincd service to bind to (Default: `ansible_default_ipv4.address` }} )

Inventory must set tinc_control_plane_bind_ip (for core) and/or tinc_vpn_ip (for core and edge nodes).
Please have a look in the task files.

## Examples

### Router mode, ring topology

(Short) Inventory:
```
[tinc_nodes]
node1 tinc_vpn_ip=10.10.0.11
node2 tinc_vpn_ip=10.10.0.12
node3 tinc_vpn_ip=10.10.0.13

[tinc_spine_nodes]
node1
node2
node3

[tinc_nodes:vars]
tinc_netname=mynetname
tinc_vpn_cidr="/24"
tinc_vpn_interface=tun0

```
### Router mode, star topology

(Detailed) Inventory:

```
[tinc_nodes]
node1
node2
node3

[tinc_spine_nodes]
node1
```

Group vars for `tinc_nodes`:

```
tinc_netname: mynetname
tinc_vpn_cidr: "/24"
tinc_vpn_interface: tun0
```

Host vars for spine node, `node1`:

```
tinc_control_plane_bind_ip: "{{ ansible_eth0.ipv4.address | default(ansible_default_ipv4.address) }}"
tinc_vpn_ip: 10.10.0.10
```

Host vars for edge node, `node2`:

```
tinc_vpn_ip: 10.10.0.11
```

Host vars for edge node, `node3`:
```
tinc_vpn_ip: 10.10.0.12
```


## Dependencies

None

## Example Playbook

See https://raw.githubusercontent.com/evrardjp/ansible-tinc/master/molecule/default/converge.yml

Don't forget to set the necessary variables in your inventory (see above).

## Testing

Tests are based on Ansible Molecule framework which:
- checks role syntax
- starts several containers with different OS <i>(only for tests. We don't mix Tinc versions in production)</i>
- applies this role to each container
- runs idempotency tests (make sure that second run will not make unexpected changes)
- verifies that each prepared node able to ping other nodes over VPN

Tests run in a github actions on PR and daily. Additionally you may execute them on local machine.

Dependencies you need to have installed for running the tests:
- Ansible
- [Docker](https://docs.docker.com/engine/install/)
- [Molecule](https://molecule.readthedocs.io/en/latest/installation.html) with `molecule-docker` driver, or tox.

### Run tests manually, using molecule directly

You can existing tests for star and ring topologies:
```bash
cd ansible-tinc
molecule test # this run default tests for Ring scenario
molecule test -s star
```

The 'molecule test' command execute full scenario: 'create', 'converge', 'check idempotency', 'verify' and 'destroy' steps. Often you don't want to have container immediately destroyed and need access it for debug. For this might be useful replace 'molecule test' with:

```bash
molecule converge # this create containers and apply the role
molecule verify # run tests described in molecule/default/verify.yml

# after both steps you have live Docker containers
# you can access them with usual commands 'docker ps', 'docker exec' etc

molecule destroy
```

### Run tests manually, using tox

tox is a test runner for python. It will install all the necessary python dependencies (ansible, molecule[docker]) in a virtual environment.

To run a test:

```bash
tox -e ansible-<version>-<tinc scenario>
```

See supported values for `version` in `tox.ini`.
Current testable scenarios for tinc are `ring`, or `star`.
Positional arguments will be passed to the `molecule test` command.

For example, to run a test for ansible-2.9, with the ring topology and prevent molecule to destroy the environment:

```bash
tox -e ansible-2.9-ring -- --destroy=never
```

## How to test role with new OS

add new image to [molecule/default/molecule.yml](molecule/default/molecule.yml) and [molecule/star/molecule.yml](molecule/star/molecule.yml) following existing examples. Files are similar except the variables `scenario.name` and `groups`. Next hightlights could be hepful:

- code `privileged: true` with `command: /sbin/init` enable systemd if container  support it. Please don't forget that privileged containers in your system could be a risk.
- Docker images lack some standard software, so [molecule/default/converge.yml](molecule/default/converge.yml) take care about installing necessary dependencies
- according with https://github.com/ansible-community/molecule/issues/959 Docker doesn't allow modify /etc/hosts in a container. To workaround this we skipping one step with `molecule-notest` tag in [tasks/tinc_configure.yml](tasks/tinc_configure.yml) and modifying /etc/hosts during container creation - following the corresponding directives in [molecule/default/molecule.yml](molecule/default/molecule.yml)


## License

Apache2

## Author Information

Jean-Philippe Evrard
