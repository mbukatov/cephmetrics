# cephmetrics

Cephmetrics is a tool that allows a user to visually monitor various metrics in a running Ceph cluster.

## Disclaimer

This document contains the installation procedure for cephmetrics released on June 30, 2017.

## Prerequisites
- RHEL 7 should be running on all hosts
- A functional ceph cluster running version ceph-osd-10.2.7-27.el7cp.x86_64 or later is already up and running.
- Another host machine independent of the ceph machines must be available.  This host will be used to receive data pushed by the hosts in the Ceph cluster, and will run the dashboard to display that data.
- A host machine on which to execute `ansible-playbook` to orchestrate the deployment must be available.  In this document, this will be the same host as the dashboard host.
- Passwordless SSH access from the deploy host to the ceph hosts. The username should be the same for all hosts.
- Passwordless sudo access on the ceph and dashboard hosts
- All hosts must share the same DNS domain

## Expected configuration

After running this procedure, the following configuration is expected.
- The ceph nodes will basically remain unchanged, except that repos necessary for interacting with the cephmetrics host machine will be installed.
- The cephmetrics host machine will have a dashboard installed which the user can then use to observe the ceph network.

## Installation

### Install executables

On the host machine on which you will run ansible-playbook, do the following steps.  This will install a repo which includes the cephmetrics installation code and ansible (version 2.2.3 or later):
```
- sudo su -
- mkdir ~/cephmetrics
- subscription-manager repos --enable rhel-7-server-rhscon-2-installer-rpms
- cd /etc/yum.repos.d/
- curl -L -O http://download.ceph.com/cephmetrics/rpm-master/el7/cephmetrics.repo
- yum install cephmetrics-ansible
```

The cephmetrics repo also needs to be installed on all the ceph nodes as well.  Run the following steps on each ceph host:
```
- sudo su -
- cd /etc/yum.repos.d/
- curl -L -O http://download.ceph.com/cephmetrics/rpm-master/el7/cephmetrics.repo
- yum install cephmetrics-ansible
```

### Edit the inventory file

A file named ~/cephmetrics/inventory needs to be created.  Ansible-playbook will use this inventory file when installing cephmetrics.  Inventory is an INI-like format file with entries for ceph-grafana and all the parts of the ceph cluster.  A template for this file can be copied from ~/cephmetrics/ansible/inventory.sample

Its format looks like:

    [ceph-grafana]
    grafana_host.example.com

    [osds]
    osd0.example.com
    osd1.example.com
    osd2.example.com

    [mons]
    mon0.example.com
    mon1.example.com
    mon2.example.com

    [mdss]
    mds0.example.com

    [rgws]
    rgw0.example.com

Since we are running `ansible-playbook` directly on the dashboard (`ceph-grafana`) host, the inventory entry should look like:
    ```
    [ceph-grafana]
    grafana_host.example.com ansible_connection=local
    ```
Omit the mdss section if no ceph mds nodes are installed.  Omit the rgws section if no rgw nodes are installed.

Ansible variables can be set in ~/cephmetrics/vars.yml if the user so desires.  [Click here](./ansible/ansible.md) for more information.

## Run the ansible-playbook

Run the following command:
- ansible-playbook -v -i \~/cephmetrics/inventory -e '@\~/cephmetrics/vars.yml' playbook.yml

