Create CentOS minimum image
===========================

Setup
-----

    yum install -y oz
    cp -a /etc/oz/oz.cfg{,.orig}
    sed -i 's/raw/qcow2/g' /etc/oz/oz.cfg

Image Create
------------

    oz-install -p -u -d3 -a centos6.ks -x centos65-libvirt.xml centos65.tdl
    qemu-img convert -c /var/lib/libvirt/images/centos-6.5.x86_64.qcow2 -O qcow2 /root/centos-6.5-`date '+%Y%m%d'`.0.x86_64.qcow2
    rm -f /var/lib/oz/icicletmp/centos-6.5.x86_64
    rm -rf /var/lib/libvirt/images/centos-6.5.x86_64.qcow2

    oz-install -p -u -d3 -a centos6-epel.ks -x centos65-epel-libvirt.xml centos65-epel.tdl
    qemu-img convert -c /var/lib/libvirt/images/centos-6.5-epel.x86_64.qcow2 -O qcow2 /root/centos-6.5-epel-`date '+%Y%m%d'`.0.x86_64.qcow2
    rm -f /var/lib/oz/icicletmp/centos-6.5-epel.x86_64
    rm -rf /var/lib/libvirt/images/centos-6.5-epel.x86_64.qcow2

Cleanup
-------

    rm -f /var/lib/oz/kernels/CentOS-65x86_64-kernel
    rm -f /var/lib/oz/kernels/CentOS-65x86_64-ramdisk


Test
----

    . ~/keystonerc_admin
    nova flavor-create m2.small 6 1024 2 1
    glance image-create --name 'CentOS 6.5' --disk-format qcow2 --container-format bare --file ~/centos-6.5-*.0.x86_64.qcow2 --is-public True --progress

    . ~/keystonerc_demo
    nova keypair-add --pub-key ~/.ssh/id_rsa.pub root
    neutron security-group-rule-create --direction ingress --protocol icmp default --remote-ip-prefix 0.0.0.0/0
    neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 22 --port-range-max 22 default --remote-ip-prefix 0.0.0.0/0
    nova boot --flavor m2.small --image 'CentOS 6.5' --key-name root --nic net-id=`neutron net-list | awk '/private/ { print $2 }'` --poll test
