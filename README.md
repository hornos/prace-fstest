
    flock2 command @@fstest curl -s http://ifconfig.me/all
    flock2 command @@fstest /etc/sysconfig/network-scripts/ifup-eth0

Ground state:

    flock2 ping @@fstest
    flock2 play @@fstest ground --extra-vars=\"domain=dyn.ki.iif.hu server=hufu.ki.iif.hu\"

Enable internal network:

    flock2 play @@fstest interfaces

Modify firewall for the internal network (hosts,multicast):

    flock2 play @@fstest system

Enable syslog cluster logging (TBD: multicast):

    flock2 play @@fstest syslog

On OS X enable and generate certs:

    module load globus/5.2.0 openssl
    cacert host coreca core-01.dyn.ki.iif.hu
    cacert sign coreca core-01.dyn.ki.iif.hu
    ...

Enable Apache SSL:

    flock2 play @@fstest ssl

Enable ganglia cluster monitor:

    flock2 play @@fstest ganglia

Enable PRACE VLAN and routing:

    flock2 play @@fstest pracenet 

Enable btsync file sharing:

    flock2 play @@fstest btsync

# CEPH

    flock2 play @@fstest ceph-repo

Node allocations:

    core-01       : mon, MDS (admin node)
    core-0{2,3,4} : mon, OSD

## Ceph Manual Installation

### Monitors

Generate an `fsid` and put into the vars file:

    uuidgen | tr [A-Z] [a-z]

Login to the first node and generate auth keys:

    cd /opt/ceph-01
    ceph-authtool --create-keyring ./ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
    ceph-authtool --create-keyring ./ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow'
    ceph-authtool ./ceph.mon.keyring --import-keyring ./ceph.client.admin.keyring
    monmaptool --create --add core-01 10.1.1.11 --fsid e99c8ce3-598b-42d0-9e12-d2708ea11901 ./monmap
    monmaptool --add core-02 10.1.1.12 --fsid e99c8ce3-598b-42d0-9e12-d2708ea11901 ./monmap
    monmaptool --add core-03 10.1.1.13 --fsid e99c8ce3-598b-42d0-9e12-d2708ea11901 ./monmap
    monmaptool --add core-04 10.1.1.14 --fsid e99c8ce3-598b-42d0-9e12-d2708ea11901 ./monmap

    monmaptool --create --clobber --add core-01 10.1.1.11 --fsid e99c8ce3-598b-42d0-9e12-d2708ea11901 ./monmap

Btsync auth files and prepare ceph:

    flock2 play @@fstest ceph

Create monitors:

    /root/ceph_monitor_reset.sh

Start monitors:

    ssh core-04 service ceph start mon.core-04

Check monitor states:

    ceph mon dump
    ceph mon stat
    ceph osd lspools
    ceph -s

### OSDs



## Ceph Auto Installation (Failed)

    flock2 ssh @@core-01
    cd /opt
    mkdir ceph-01
    ceph-deploy new core-01

Edit `ceph.conf` and add [public and cluster network](http://ceph.com/docs/master/rados/configuration/network-config-ref/):

    [global]
      public network = {public-network/netmask}
      cluster network = {cluster-network/netmask}

Install Ceph (optional):

    ceph-deploy install core-01

Start the monitor:

    ceph-deploy mon create-initial

Add the first OSD:

    ceph-deploy mon create core-02
    ceph-deploy disk zap core-02:xvdb
    ceph-deploy osd prepare core-02:xvdb
