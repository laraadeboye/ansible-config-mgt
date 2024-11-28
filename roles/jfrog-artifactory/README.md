jfrog_artifactory role
=========
This artifactory role installs the open source version of artifactory, version 6.9.6 on Ubuntu LTS 24.04

Requirements
------------

The role sets up artifactory on AWS EC2, It requires the installation of boto package, openjdk-8, python3. The prequisites are stated in the sample entry playbook.
An extra block volume must be allocated and attached to the EC2 instance. The instance should be a minimum of t2.medium. The role checks if the block volume has been formatted in the ext4 system and formats it.

Role Variables
--------------

 The role variables are set in the default/main.yml file
They include the following variables: The archive file can either be tar or zip. Whatever format chosen must be further configured to properly unarchive in the tasks/install.yml

# Artifactory Configuration
artifactory_version: "6.9.6"
artifactory_home: "/mnt/artifactory-data/artifactory"
artifactory_data_volume: "/mnt/artifactory-data"
  # artifactory_download_url: "https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/{{ artifactory_version }}/jfrog-artifactory-oss-{{ artifactory_version }}-linux.tar.gz"
artifactory_download_url: "https://jfrog.bintray.com/artifactory/jfrog-artifactory-oss-{{ artifactory_version }}.zip"

# User and Group
artifactory_user: "artifactory"
artifactory_group: "artifactory"

# Volume Configuration
volume_device: "/dev/xvdf" 
min_free_space_percentage: 20

Other variables are set in the playbook.
 ansible_python_interpreter
 venv_path

Dependencies
------------
No known dependencies for now.

Example Playbook
----------------

This playbook includes instructions to configure prerequisites. The playbook can be modified to place the pre-tasks in a separate yaml file.

- hosts: artifactory_servers
  become: yes
  vars:
    ansible_python_interpreter: /usr/bin/python3
    venv_path: /opt/ansible_venv

  pre_tasks:
    - name: Ensure required system packages are installed
      ansible.builtin.apt:
        name:
          - lvm2
          - python3-pip
          - python3-venv
          - software-properties-common
        state: present
        update_cache: yes

    - name: Create Ansible virtual environment
      ansible.builtin.command:
        cmd: python3 -m venv {{ venv_path }}
        creates: "{{ venv_path }}/bin/activate"

    - name: Upgrade pip in virtual environment
      ansible.builtin.pip:
        name: pip
        state: latest
        virtualenv: "{{ venv_path }}"

    - name: Install required Python libraries in virtual environment
      ansible.builtin.pip:
        name:
          - boto3
          - botocore
        virtualenv: "{{ venv_path }}"

  roles:
    - jfrog-artifactory


License
-------

BSD

Author Information
------------------

laraadeboye
