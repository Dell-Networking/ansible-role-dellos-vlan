VLAN Role for Dell EMC Networking OS
====================================
This role facilitates configuring virtual LAN (VLAN) attributes. It supports the creation and deletion of a VLAN and its member ports.
This role is abstracted for OS6, OS9, and OS10.  

Installation
------------

```
ansible-galaxy install Dell-Networking.dellos-vlan
```

Requirements
------------

This role requires an SSH connection for connectivity to your Dell EMC Networking device. You can use any of the built-in Dell EMC Networking OS connection variables, or the ``provider``
dictionary.

Role Variables
--------------

``dellos_vlan`` (dictionary) contains the hostname (dictionary).
The hostname is the value of the variable ``hostname`` that corresponds to the name of the OS device.
This role is abstracted using the variable ``ansible_net_os_name`` that can take the following values: dellos6, dellos9 and dellos10.

Any role variable with a corresponding state variable set to absent negates the configuration of that variable. 
For variables with no state variable, setting an empty value for the variable negates the corresponding configuration.
The variables and its values are case-sensitive.

The ``hostname`` (dictionary) holds a dictionary with the VLAN ID key. The VLAN ID can be in the range of 1-4094.

``VLAN ID`` holds the following key values:

|       Key | Type                      | Notes                                                                                                                                                                                     |
|------------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name             | string                        |Configures name of the VLAN. This key is not supported for OS10 devices.                   | 
| tagged_members   | string          |Specifies the list of port members to be tagged to the corresponding VLAN. | 
| untagged_members | string          |Specifies the list of port members to be untagged to the corresponding VLAN. |
| state           | string, choices: absent, present*          |Deletes the VLAN corresponding to the ID when set to absent.  |
| members_state   | string, choices: absent, present*           |Deletes the members of the VLAN when set to absent.  |
                                                                                                      

```
Note: Asterisk (*) denotes the default value if none is specified.
```

Connection Variables
--------------------

Ansible Dell EMC Networking roles require the following connection information to establish 
communication with the nodes in your inventory. This information can exist in
the Ansible group_vars or host_vars directories, or in the playbook itself.


|         Key | Required | Choices    | Description                              |
| ----------: | -------- | ---------- | ---------------------------------------- |
|        host | yes      |            | Hostname or address for connecting to the remote device over the specified ``transport``. The value of this key is the destination address for the transport. |
|        port | no       |            | Port used to build the connection to the remote device. If this key does not specify the value, the value defaults to 22. |
|    username | no       |            | Configures the username that authenticates the connection to the remote device. The value of this key authenticates the CLI login. If this key does not specify the value, the value of environment variable ANSIBLE_NET_USERNAME is used instead. |
|    password | no       |            | Specifies the password that authenticates the connection to the remote device. If this key does not specify the value, the value of environment variable ANSIBLE_NET_PASSWORD is used instead. |
|   authorize | no       | yes, no*   | Instructs the module to enter privileged mode on the remote device before sending any commands. If this key does not specify the value, the value of environment variable ANSIBLE_NET_AUTHORIZE is used instead. If not specified, the device attempts to execute all commands in non-privileged mode.|
|   auth_pass | no       |            | Specifies the password to use if required to enter privileged mode on the remote device. If ``authorize`` is set to no, then this key is not applicable. If this key does not specify the value, the value of environment variable ANSIBLE_NET_AUTH_PASS is used instead. |
|   transport | yes      | cli*       | Configures the transport connection to use when connecting to the remote device. This key supports connectivity to the device over CLI (SSH).  |
|    provider | no       |            | Convenient method that passes all of the above connection arguments as a dictonary object. All constraints (such as required or choices) must be met either by individual arguments or values in this dictonary. |


```
Note: Asterisk (*) denotes the default value if none is specified.
```

Dependencies
------------

The dellos-vlan role is built on modules included in the core Ansible code.
These modules were added in Ansible version 2.2.0.

Example Playbook
----------------
The following example uses the dellos-vlan role to setup the VLAN ID and name. This example also configures tagged and untagged port members for the VLAN. You can also delete the VLAN with the ID or delete the members associated to it. This example also creates a ``hosts`` file with the switch, a corresponding ``host_vars`` file, and
then a simple playbook that references the dellos-vlan role.

Sample hosts file:

    leaf1 ansible_host= <ip_address> ansible_net_os_name= <OS name(dellos9/dellos6/dellos10)>

Sample ``host_vars/leaf1``:
     
    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx
      password: xxxxx
      authorize: yes
	  auth_pass: xxxxx 
      transport: cli

Sample ``vars/main.yaml``:

    dellos_vlan:
      leaf1:
        vlan 100:
          name: "Mgmt Network"
          tagged_members:
            - fortyGigE 1/30,1/32
          untagged_members:
            - fortyGigE 1/14
          members_state: present
          state: present


Simple playbook to setup system, ``leaf.yaml``:

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-vlan
                
Then run with:

    ansible-playbook -i hosts leaf.yml

Contact
--------
Send general comments and feedback to: feedback-ansible-dell-networking@dell.com

License
--------

Copyright (c) 2017, Dell Inc. All rights reserved.
 
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
 
    http://www.apache.org/licenses/LICENSE-2.0
 
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
