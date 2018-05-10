# kolla-pike

This is a repo for working OpenStack Kolla configurations, developed with kolla-ansible 5.0.2 and OpenStack Pike.

It includes several versions of /etc/kolla/globals.yml, to enable installation in various Virtual Machines and Bare Metal Hosts Configurations.

1. aio-bm-basic.yml - Install OpenStack Pike All-in-one on baremetal host with Basic Services (Keystone, Glance, Cinder, Nova, Neutron, Heat, Horizon)

I have used a laptop with i7 Gen8, 32GB RAM, 512GB SSD, builtin 1Gb Ethernet (eth0) plus USB 1Gb Ethernet (eth1) and builtin Wifi (wlan0), as my Host.

Use Ubuntu 16.04 on Host - I have used Ubuntu 16.04 Desktop.
Rename Network Interfaces to standard names (eth0, eth1 and wlan0), make sure eth0 and eth1 are UP in 'ip a' output (connect them to a switch).
In my case eth0 is connected to a switch and nothing else, while eth1 is connected to a switch with wifi bridge attached to another port. Wifi bridge connects to my default wifi network, connected to Internet (via SNAT).
wlan0 gives me an Internet connection and is managed by Network Connections Desktop App.

Suggested shape of /etc/network/interfaces:

#------------
auto lo

iface lo inet loopback


auto eth0

iface eth0 inet static

  address 10.0.0.11

  netmask 255.255.255.0
  
  dns-nameservers 8.8.8.8


auto eth1

iface eth1 inet manual

  up ip link set dev eth1 up
  
  down ip link set dev eth1 down
  
#------------

Set eth0 address as you wish.

Install kolla-ansible following the procedure in https://docs.openstack.org/kolla-ansible/pike/user/quickstart.html
When you reach "Install Kolla for deployment or evaluation (around middle of the page), install Pike version of kolla-ansible with:

pip install kolla-ansible==5.0.2


Clone this repository to your home directory.

cp kolla-absible/aio-bm-basic.yml /etc/kolla/globals.yml


Edit /etc/kolla/globals.yml to set Network Address to match your Environment:

kolla_internal_vip_address: "10.0.0.10"  # make it an unused IP in your eth0 network


Skip "Install Kolla for development" and "Local registry".
Continue Installation with "Automatic host bootstrap".

kolla-genpwd

kolla-ansible -i all-in-one bootstrap-servers

Modify /etc/systemd/system/docker.service.d/kolla.conf and restart docker daemon, as requested.
Pull Kolla images:

kolla-ansible pull -i all-in-one

Skip kolla-build.

Create and edit /etc/kolla/config/nova/nova-compute.conf to enable QEMU.

Run Kolla prechecks:

kolla-ansible prechecks -i all-in-one

Deploy OpenStack:

kolla-ansible deploy -i all-in-one

List containers and check if all are running, not restarting:

docker ps -a

Edit /usr/local/share/kolla-ansible/init-runonce to modify External Network (Provider) parameters:

EXT_NET_CIDR='192.168.1.0/24'
EXT_NET_RANGE='start=192.168.1.220,end=192.168.1.239'
EXT_NET_GATEWAY='192.168.1.1'

I have set the EXT_NET_CIDR to match my default Wifi network, connected to Internet.
EXT_NET_RANGE is set to addresses outside of DHCP allocation range to avoid conflicts.

kolla-ansible post-deploy
. /etc/kolla/admin-openrc.sh
cd /usr/local/share/kolla-ansible
./init-runonce

Check OpenStack admin user password:

grep keystone_admin_password /etc/kolla/passwords.yml

Open your browser and type in the kolla_internal_vip_address, 10.0.0.10 in above example.
Login to Horizon as admin and have fun :)

You can safely play with the Configuration by editing /etc/kolla/globals.yml, for example adding other services.
In order to reconfigure your deployment (in "nice and easy" way), run:

kolla-ansible reconfigure

To tear down the deployment run:

kolla-ansible -i all-in-one destroy --yes-i-really-really-mean-it

Now you can make major modifications to globals.yml and start over with bootstrap-servers (line 52 of this file).
