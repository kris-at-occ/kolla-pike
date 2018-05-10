# kolla-pike

This is a repo for working OpenStack Kolla configurations, developed with kolla-ansible 5.0.2 and OpenStack Pike.


1. aio-bm-basic.yml - Install OpenStack Pike All-in-one on baremetal host with Basic Services (Keystone, Glance, Cinder, Nova, Neutron, Heat, Horizon)

I have used a laptop with i7 Gen8, 32GB RAM, 512GB SSD, builtin 1Gb Ethernet (eth0) plus USB 1Gb Ethernet (eth1) and builtin Wifi (wlan0).
Use Ubuntu 16.04 on Host - I have used Ubuntu 16.04 Desktop.
Rename Network Interfaces to standard names (eth0, eth1 and wlan0), make sure eth0 and eth1 are UP in 'ip a' output (connect them to switch).
Install kolla-ansible following the procedure in https://docs.openstack.org/kolla-ansible/pike/user/quickstart.html
When you reach "Install Kolla for deployent or evaluation (around middle of the page), install kolla-ansible with:

pip install kolla-ansible==5.0.2

Clone this repository to your home directory.
