# How to use Ansible with OpenStack

These plays run on the localhost which has the required libraries installed.
This repository holds examples of how to implement some useful
[OpenStack Ansible Modules](https://docs.ansible.com/ansible/latest/search.html?q=os_&check_keywords=yes&area=default).
They all start with `os_`.

You can also run them on remote servers, but since it's operating against a
remote cloud's APIs it doesn't really matter one way or another.


## Required Software

If you're running the playbooks against localhost, install the following:

```bash
apt-get install -y python ptyhon-pip
pip install \
  ansible \
  openstack-sdk
```

## Define Your Environment

Create an environment file like `environment.yml` to identify and authenticate
against your cloud.

```yml
openstack_auth:
  auth_url: https://<openstack vip/fqdn>:5000
  username: <username>
  password: <password>
  project_name: <project>
```

## Examples

### Gather Info

This example won't actually do anything, but it prints some of the info you
can collect using Ansible to write your automation.

```bash
# Show all info
ansible-playbook -e @environment.yml gather-info.yml

# Show only a certain user in the user step. Each step has a similar variable.
ansible-playbook -e @environment.yml -e user_name_or_id=DemoUser gather-info.yml
```

### OpenStack Demo

This example runs through a little demo of an OpenStack workflow. All the names
use defaults.yml variables, so they can be overrode with environment variables
at run-time.

The demo will:

1. Create a project called `DemoProject`. It will contain the demo workloads.
1. Assign a quota to the new project
1. Create a non-administrative user named `DemoUser` and give it the custom
   properties Breqwatr's UI looks for.
1. Grant `_member_` access to `DemoUser` in the `DemoProject` project.
1. Download a cirros image and convert it to .raw format. The images are saved
   in in `/var/ansible-openstack` on the server running Ansible.
1. Upload the cirros image to Glance, name it `DemoCirros`
1. Create a private flavor called `DemoFlavor`, and scope it to `DemoProject`.
   This flavor defines the size of instances to be created.
1. Create a VLAN network named `DemoVLAN`, private to `DemoProject`
1. Configure an OpenStack Subnet called `subnet-DemoVLAN`, with an IP address
   pool and gateway configuration.
1. Create a VXLAN network named `DemoOverlay`, private to `DemoProject`.
1. Create a subnet called `subnet-DemoOverlay`, with an IP address pool
1. Create a virtual router called `router-DemoOverlay`. Connect the
   `DemoOverlay` network to it, so this router acts as the default gateway for
   all traffic on that network. Also assign a port to this router from the
   `DemoVLAN`, with an IP from its subnet. This router will perform outbound
   NAT for ports on `DemoOverlay`, hold floating IPs, and route all traffic to
   the default gateway defined in `subnet-DemoOverlay`.
1. Create an instance (VM) in the `DemoProject`, using the `DemoFlavor` flavor,
   based on the `DemoCirros` image. Use the `DemoOverlay` for its network port.
1. Assign a floating IP address on `DemoVLAN`, creating a NAT rule to the port
   that `DemoServer` has on `DemoOverlay`. This will allow users to SSH in
   using the private key associated with the uploaded public key.


To run the demo:

```bash
ansible-playbook -e @environment.yml demo.yml
```

# References

Ansible modules used in this project:

- [os_project_info](https://docs.ansible.com/ansible/latest/modules/os_project_info_module.html)
- [os_project](https://docs.ansible.com/ansible/latest/modules/os_project_module.html)
- [os_quota](https://docs.ansible.com/ansible/latest/modules/os_quota_module.html)
- [os_user_info](https://docs.ansible.com/ansible/latest/modules/os_user_info_module.html)
- [os_user](https://docs.ansible.com/ansible/latest/modules/os_user_module.html)
- [os_user_role](https://docs.ansible.com/ansible/latest/modules/os_user_role_module.html)
- [os_image_info](https://docs.ansible.com/ansible/latest/modules/os_image_info_module.html)
- [os_image](https://docs.ansible.com/ansible/latest/modules/os_image_module.html)
- [os_flavor_info](https://docs.ansible.com/ansible/latest/modules/os_flavor_info_module.html)
- [os_nova_flavor](https://docs.ansible.com/ansible/latest/modules/os_flavor_info_module.html)
- [os_project_access](https://docs.ansible.com/ansible/latest/modules/os_project_access_module.html)
- [os_network_info](https://docs.ansible.com/ansible/latest/modules/os_networks_info_module.html)
- [os_network](https://docs.ansible.com/ansible/latest/modules/os_network_module.html)
- [os_subnets_info](https://docs.ansible.com/ansible/latest/modules/os_subnets_info_module.html)
- [os_subnet](https://docs.ansible.com/ansible/latest/modules/os_subnet_module.html)
- [os_router](https://docs.ansible.com/ansible/latest/modules/os_router_module.html)
- [os_keypair](https://docs.ansible.com/ansible/latest/modules/os_keypair_module.html)
- [os_server](https://docs.ansible.com/ansible/latest/modules/os_server_module.html)
- [os_floating_ip](https://docs.ansible.com/ansible/latest/modules/os_floating_ip_module.html)
