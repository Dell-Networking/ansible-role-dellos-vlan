VLAN role
=========

This role facilitates configuring virtual LAN (VLAN) attributes. It supports the creation and deletion of a VLAN and its member ports. This role is abstracted for dellos9, dellos6, and dellos10 devices. 

The VLAN role requires an SSH connection for connectivity to a Dell EMC Networking device. You can use any of the built-in Dell EMC Networking OS connection variables, or the *provider* dictionary.

Installation
------------

    ansible-galaxy install Dell-Networking.dellos-vlan

Role variables
--------------

- Role is abstracted using the *ansible_net_os_name* variable that can take dellos9, dellos10, and dellos6 values  
- If *dellos_cfg_generate* is set to true, the variable generates the role configuration commands in a file
- Any role variable with a corresponding state variable set to absent negates the configuration of that variable
- For variables with no state variable, setting an empty value for the variable negates the corresponding configuration
- *dellos_vlan* (dictionary) holds the key with the VLAN ID key and default-vlan key.
- VLAN ID key should be in format "vlan <ID>" (1 to 4094)
- Variables and values are case-sensitive

**dellos_vlan**

| Key        | Type                      | Notes                                                   | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``default_vlan`` | boolean                 | Configures default vlan feature as diabled if set to true | dellos9 |
| ``default_vlan_id`` | integer | Configures a vlan-id as default vlan for an existing vlan on OS10 devices. | dellos10 |

**VLAN ID keys**

| Key        | Type                      | Notes                                                   | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``name``             | string                        | Configures name of the VLAN                    | dellos6, dellos9 |
| ``description``      | string          | Configures a single line description for the VLAN | dellos9, dellos10 |
| ``tagged_members``   | list         | Specifies the list of port members to be tagged to the corresponding VLAN (see the ``tagged_members.*``) | dellos6, dellos9, dellos10 |
| ``tagged_members.port`` | string | Specifies valid device interface names to be tagged for each vlan | dellos6, dellos9, dellos10 |
| ``tagged_members.state`` | string: absent,present | Deletes the tagged association for vlan if set to absent | dellos6, dellos9, dellos10 |
| ``untagged_members`` | list         | Specifies the list of port members to be untagged to the corresponding VLAN (see ``untagged_members.*``) | dellos6, dellos9, dellos10 |
| ``untagged_members.port`` | string | Specifies valid device interface names to be untagged for each vlan | dellos6, dellos9, dellos10 |
| ``untagged_members.state`` | string: absent,present | Deletes the untagged association for vlan if set to absent | dellos6, dellos9, dellos10 |
| ``state``           | string: absent,present\*          | Deletes the VLAN corresponding to the ID if set to absent | dellos6, dellos9, dellos10 |
                                                                                                      
> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

Connection variables
--------------------

Ansible Dell EMC Networking roles require connection information to establish communication with the nodes in your inventory. This information can exist in the Ansible *group_vars* or *host_vars directories*, or in the playbook itself.

| Key         | Required | Choices    | Description                                         |
|-------------|----------|------------|-----------------------------------------------------|
| ``host`` | yes      |            | Specifies the hostname or address for connecting to the remote device over the specified transport |
| ``port`` | no       |            | Specifies the port used to build the connection to the remote device; if value is unspecified, it defaults to 22 |
| ``username`` | no       |            | Specifies the username that authenticates the CLI login for the connection to the remote device; if value is unspecified, the ANSIBLE_NET_USERNAME environment variable value is used  |
| ``password`` | no       |            | Specifies the password that authenticates the connection to the remote device; if value is unspecified, the ANSIBLE_NET_PASSWORD environment variable value is used |
| ``authorize`` | no       | yes, no\*   | Instructs the module to enter privileged mode on the remote device before sending any commands; if value is unspecified, the ANSIBLE_NET_AUTHORIZE environment variable value is used, and the device attempts to execute all commands in non-privileged mode . This key is supported only in dellos9 and dellos6.|
| ``auth_pass`` | no       |            | Specifies the password to use if required to enter privileged mode on the remote device; if ``authorize`` is set to no this key is not applicable; if value is unspecified, the ANSIBLE_NET_AUTH_PASS environment variable value is used . This key is supported only in dellos9 and dellos6.|
| ``provider`` | no       |            | Passes all connection arguments as a dictonary object; all constraints (such as required or choices) must be met either by individual arguments or values in this dictionary |

> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

Dependencies
------------

The *dellos-vlan* role is built on modules included in the core Ansible code. These modules were added in Ansible version 2.2.0.

## Example playbook

This example uses the *dellos-vlan* role to setup the VLAN ID and name, and it configures tagged and untagged port members for the VLAN. You can also delete the VLAN with the ID or delete the members associated to it. It creates a *hosts* file with the switch details and corresponding variables.

The hosts file should define the *ansible_net_os_name* variable with corresponding Dell EMC networking OS name. When *dellos_cfg_generate* is set to true, the variable generates the configuration commands as a .part file in *build_dir* path. By default, the variable is set to False. It writes a simple playbook that only references the dellos-vlan role.

**Sample hosts file**

    leaf1 ansible_host= <ip_address> ansible_net_os_name= <OS name(dellos9/dellos6/dellos10)>

**Sample host_vars/leaf1**
     
    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx
      password: xxxxx
      authorize: yes
      auth_pass: xxxxx 
    build_dir: ../temp/dellos9

    dellos_vlan:
        default_vlan: true
        vlan 100:
          name: "Mgmt Network"
          description: "Int-vlan"
          tagged_members:
            - port: fortyGigE 1/30
              state: absent
          untagged_members:
            - port: fortyGigE 1/14
              state: present
          state: present


**Simple playbook to setup system - leaf.yaml**

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-vlan
                
**Run**

    ansible-playbook -i hosts leaf.yaml

(c) 2017 Dell EMC
