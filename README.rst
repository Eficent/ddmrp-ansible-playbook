Deploy the Ansible roles
- Create a requirements.yml file and indicate there the git repos to source the Ansible roles from. See http://docs.ansible.com/ansible/latest/galaxy.html#installing-roles

host> nano requirements.yml

  # from GitHub

- name: ansible-odoo

  src: https://github.com/osiell/ansible-odoo

  version: origin/master

- name: postgresql

  src: https://github.com/ANXS/postgresql

host> sudo ansible-galaxy install -r requirements.yml

- extracting ansible-odoo to /home/jordi/.ansible/roles/ansible-odoo

- ansible-odoo was installed successfully

- extracting postgresql to /home/jordi/.ansible/roles/postgresql

- postgresql was installed successfully

Note: use --force to ensure that the latest version of the roles is installed.

Install LXC Container
This is only to test locally the execution of the ansible playbook on a target host.

Create the LXC container:

host> sudo lxc-create -t debian -n odoo10

Start the LXC container:

host> sudo lxc-start -n odoo10 -d

Check that the container is up

host> sudo lxc-ls -f

NAME STATE AUTOSTART GROUPS IPV4 IPV6  

odoo10 RUNNING 0 - 10.0.3.217 -

Attach to the container

host> sudo lxc-attach -n odoo10

Install nano:

container> apt-get install nano

Install python

container> apt-get install python

Allow root to connect over ssh:

container> nano /etc/ssh/sshd_config

FROM:

PermitRootLogin without-password TO:

PermitRootLogin yes

Restart ssh

container> /etc/init.d/ssh restart

Provide a password to root:

container> passwd



Exit from the container and try to ssh into it with the new user

container>exit

host> ssh root@10.0.3.217


Create Hosts Inventory File

Create a project folder under home dir.

host> mdir ansible-test && cd ansible-test

host> nano inventory

odoo10 ansible_ssh_host=10.0.3.217



Create Playbook File

host/ansible-test> nano ./playbook.yml

- name: Odoo 10

  hosts: odoo10

  roles:

    - postgresql

    - ansible-odoo

  vars:

    # [postgresql]

    - postgresql_version: 9.3

    # [odoo]

    - odoo_version: 11.0

    - odoo_install_type: pip

    - odoo_config_unaccent: True

    - odoo_pip_requirements_url: https://raw.githubusercontent.com/Eficent/sample-oca-pip-requirements/11.0/requirements.txt

    - odoo_config_admin_passwd: SuPerPassWorD

    - odoo_config_addons_path: ""

environment:

    LC_ALL: en_US.UTF-8

Deploy the Playbook to the container

host> ansible-playbook -i inventory playbook.yml -e "ansible_ssh_user=root" -k -v
