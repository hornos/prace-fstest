
    flock2 command @@fstest curl -s http://ifconfig.me/all
    flock2 command @@fstest /etc/sysconfig/network-scripts/ifup-eth0

Ground state:

    flock2 ping @@fstest
    flock2 play @@fstest ground --extra-vars=\"domain=dyn.ki.iif.hu server=hufu.ki.iif.hu\"

Enable internal network:

    flock2 play @@fstest interfaces

Modify firewall for the internal network (hosts,multicast):

    flock play @@fstest system

Enable syslog cluster logging (TBD: multicast):

    flock play @@fstest syslog

On OS X enable and generate certs:

    module load globus/5.2.0 openssl
    cacert host coreca core-01.dyn.ki.iif.hu
    cacert sign coreca core-01.dyn.ki.iif.hu
    ...

Enable Apache SSL:

    flock play @@fstest ssl

Enable ganglia cluster monitor:
