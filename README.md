populate_guacamole
--------------

This role takes input variables and populates the Apache Guacamole Bastion host, and associated SSH and RDP Connections


Requirements
------------
An Apache Guacamole Bastion Host must already be running and referenced in your ansible inventory. This role assumes use of the AWS Marketplace Guacamole Bastion Server: https://aws.amazon.com/marketplace/pp/B06Y67KPD9

Role Variables
--------------
`GUAC_USERNAME`: The administrator username for your Guacamole Server. This is typically `guacadmin`.

`GUAC_PASSWORD`: The administrator password for your Guacamole Server. For AWS, the default password is the ***instance ID***

`student_total`: The total number of users for your Guacamole server. Each user will have the following username: studentX, where X is the number

`password`: This is the password that students will use to log in. The default value is `ansible`.

`ec2_name_prefix`: The prefix for your Linklight login url
domain: The domain for your Linklight login url

The entire URL will have the format of studentX.ec2_name_prefix.domain. i.e., `student1.test-workshop.ansible.com`

Example Inventory
----------------

Example Usage
-------------

```
# install ansible first and may force upgarde of python

ansible-playbook --connection=local -i 127.0.0.1,  -e=@vars/main.yml playbook.yaml -v
```

Example Inventory
----------------

```
[guacamole]
127.0.0.1 ansible_host=127.0.0.1 
```

Example Input/Extra Variables
----------------
```
  vars:
      GUAC_USERNAME: guacadmin
      GUAC_PASSWORD: guacadmin
      guac_student_password: 'password'
      student_total: 15
      ec2_name_prefix: classroom
      domain: example.com
      ansible_host: localhost
      guacamole_server: hostname.ec2_name_prefix.domain
      guacamole_prot: https
      guacamole_port: 443
      guac_db: postgresql
      courseid: ''
      rdp_username: administrator
      rdp_password: Password!
```

Example Playbook
----------------

```
---
- name: Populate Apache Guacamole with Users and Connections
  hosts: guacamole
  gather_facts: False

  tasks:

    - name: Set guacamole_server fact
      set_fact:
        guacamole_server: "{{ ansible_host }}"

    - name: Invoke populate_guacamole role
      include_role:
        name: populate_guacamole
      with_sequence: start=1 count="{{ student_total }}"

    - name: Clean Up JSON Files
      file:
        path: "{{ playbook_dir }}/{{ item }}"
        state: absent
      with_items:
        - rdp_connection.json
        - ssh_connection.json
        - users.json
      delegate_to: localhost
```
Playbook Run Results
----------------

As a result of running the example playbook with the example extra variables above:

1) Logging in as `guacadmin` will reveal that all Users and their associated SSH and RDP Connections are now present.

![Guacamole Admin View](images/guacadmin_view.jpg)

2) Logging in as a particular student will show that said student will only be able to access their particular connections (this is what we want to prevent a student from logging into his/her neighbor's workbench).

![Student 11 View](images/student11_view.jpg)

3) Under the settings, you will see the details for the connection have been populated per variable inputs.

![RDP Connection Details](images/rdp_details.jpg)
