# About #

The main goal of this project is the automation of Campus EVPN Deployment based on Catalyst 9000.

Custom Jinja templates and Python modules are used to build an initial config and modify the network configuration.

Project has a modular structure which gives an ability to introduce new features/services gradually step-by-step.

* DAG (Distributed Anycast Gateway)
  * [DAG github](https://github.com/Cat9kEVPN/cat9k-evpn-ansible/tree/releases/v2.x.x/dag)
  * [DAG documentation](https://cat9k-evpn-ansible.readthedocs.io/en/latest/input_dag.html)

# Topology #

Below you can find a topology which is used in the automation scenario.

<img width="737" alt="ansible_lab_topology" src="https://user-images.githubusercontent.com/99259970/155182099-7e5d98f4-8e4e-4b01-96a8-30b9badc5be2.png">

# General description #

<img width="903" alt="playbook_description" src="https://user-images.githubusercontent.com/99259970/172883945-3997d95b-3d6c-47f4-97ac-de0826b281c5.png">

## Inputs ##

### inventory.yml ###

In the file **inventory.yml** all nodes of the network are described. Nodes are grouped into two groups/roles - Leafs and Spines.

```yml
all:
  children:
    leaf:
      hosts:
        Leaf-01:
          ansible_host: 10.1.1.3
<...skip...>
          
    spine:
      hosts:
        Spine-01:
          ansible_host: 10.1.1.1
<...skip...>

```

### group_vars/all.yaml

Under **group_vars/all.yaml** access section and configuration for all VTEPs are present.

```yml
# Access section
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: cisco.ios.ios
ansible_python_interpreter: "python"
ansible_user: cisco
ansible_ssh_pass: cisco123

# EVPN section
l2vpn_global:
  replication_type: 'static'
  router_id: 'Loopback1'
  default_gw: 'yes'

vrfs:
  green:
    ipv6_unicast: 'enable'
    rd: '1:1'
    afs:
      ipv4:
        rt_import: 
          - '1:1'
          - '1:1 stitching'
<...skip...>
```

Full documentation about **group_vars/all.yaml** section could be found [here]()

Each directory may has several playbook. Usually there are 4 of them:

```
playbook_underlay.yml:
      underlay config. Please check the host_vars/xx.yml

playbook_overlay.yml: 
       overlay config. Please refer to the overlay configuration parameters

playbook_output.yml:  
        executes overlay show commands including the config per node into output/xxx.txt

playbook_all.yml:      
         run above three.

Each of the playbooks above can be run independantly.
```

# Variables config description #

Configuration on the fabric switch usually consists from two parts:
- individual node configuration. Ususally it is Underlay config with unique ip addresses, interfaces, etc.
- shared configuration. Usually it is Overlay part like VLANs, SVIs, EVIs, etc.

Individual part of the configuration is stored in **host_vars/Leaf-xx.yml** and **host_vars/Spine-xx.yml**.
This configuration will be applied to the specific node only.

Shared configuration is stored in **groups/all.yml**.
This configuration will be applied to all nodes in group.

File **inventory.yaml** has an information about switch ip addresses, hostnames and groups that it belongs to.

# Observe changes on the switch #

For checking the configuration that is deployed by Ansible on the switch next configuration could be used.

```
conf t
archive
 log config
  logging enable
  notify syslog contenttype plaintext
end
term mon
```
